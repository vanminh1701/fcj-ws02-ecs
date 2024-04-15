---
title : "Disaster Recovery Strategy"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

Phục hồi thảm họa là quá trình chuẩn bị và khắc phục các sự cố bất ngờ khiến trang web không thể hoạt động bình thường. Những sự kiện đó có thể là mất điện tử, lỗi của con người đối với tài nguyên, lỗi phần cứng hoặc phần mềm độc hại từ tin tặc. Điều này đòi hỏi phải chạy cơ sở hạ tầng thứ cấp và xử lý lưu lượng truy cập trong khi chúng tôi đang sửa chữa cơ sở hạ tầng chính.

Thông thường, hầu hết các tài nguyên trong AWS đều hỗ trợ tính sẵn sàng cao ở nhiều Vùng sẵn sàng trong cùng một Khu vực. Ví dụ: khi triển khai Cân bằng tải ứng dụng, bạn có thể chọn triển khai trên mọi AZ trong Khu vực và khi một AZ ngừng hoạt động, ALB vẫn có sẵn trên các AZ khác. Khi bạn làm việc với Auto Scaling Group, nếu trên AZ không thành công, ASG sẽ tạo ra các phiên bản mới trên các AZ khác để đạt được công suất mong muốn.
![Disaster Recovery strategies](/images/4.4-disaster-recovery-strategies.png)

AWS đã phân loại các chiến lược khắc phục thảm họa thành 4 phương pháp:

- Sao lưu và khôi phục: bạn có lịch sao lưu dữ liệu dự án của mình như cơ sở dữ liệu, kho lưu trữ tệp. Và sau đó bạn khôi phục những dữ liệu đó vào cơ sở hạ tầng mới.
- Đèn hoa tiêu: bạn sẽ có sẵn cơ sở dữ liệu nhưng không có tài nguyên tính toán để phục vụ lưu lượng
- Chế độ chờ ấm: bạn chạy công suất thấp để đảm bảo hệ thống có thể xử lý lưu lượng nhỏ và mở rộng quy mô dần dần
- Multi site Active/Active: Bạn sao chép cơ sở hạ tầng chính và chỉ cần chuyển đổi lưu lượng

Phần này sẽ giới thiệu chiến lược Sao lưu và Khôi phục bằng cách tận dụng tính năng sao chép được quản lý của dịch vụ AWS và CloudFormation StackSet để triển khai các dịch vụ ở 2 Khu vực.

- Bật sao chép trên S3 và ECR
- Triển khai một cụm ECS khác với nhiệm vụ dịch vụ mong muốn là 0
- Triển khai ALB nhàn rỗi

Nếu thảm họa xảy ra, chúng ta sẽ tăng số lượng nhiệm vụ mong muốn của ECS và thay đổi Route53 thành cơ sở hạ tầng thứ cấp theo cách thủ công.

#### Setup Elastic Container Registry replication

Chúng tôi sẽ tạo một CloudFormation nested stack cho ECR chỉ tạo trên Region chính với tùy chọn `Condition IsPrimary` trong mẫu

```yml
Conditions:
  IsPrimary: !Equals [!Ref AWS::Region, ap-southeast-1]
Resources:
...
    ecrStack:
    Type: AWS::CloudFormation::Stack
    Condition: IsPrimary
    Properties:
        TemplateURL: !Sub https://${BucketName}.s3.${AWS::Region}.amazonaws.com/stacks/ecr.yml
        Parameters:
        RepositoryName: ws02
        SecondaryRegion: ap-northeast-1
...
```

Mẫu CloudFormation này tạo một ECR repository để lưu trữ docker images và một `ReplicationConfiguration` với quy tắc lọc tên repository và đích đến là Region khác trong cùng một tài khoản.

```yml
Resources:
  Ws02Repository:
    Type: AWS::ECR::Repository
    Properties:
      ImageTagMutability: IMMUTABLE
      RepositoryName: !Ref RepositoryName
      ImageScanningConfiguration:
        ScanOnPush: true
  ECRReplication:
    Type: AWS::ECR::ReplicationConfiguration
    DependsOn: Ws02Repository
    Properties:
      ReplicationConfiguration: 
        Rules:
          - Destinations:
              - Region: !Ref SecondaryRegion
                RegistryId: !Ref AWS::AccountId
            RepositoryFilters:
              - Filter: !Ref Ws02Repository
                FilterType: PREFIX_MATCH

```

#### Setup CloudFormation StackSet

CloudFormation giúp triển khai hệ thống trên nhiều Region một cách nhanh chóng và quản lý tập trung các stack. CloudFormation StackSet hỗ trợ gắn các thông số input cho stack, nhưng bạn cũng có thể ghi đè các thông số cho một Region cụ thể trong cấu hình `StackInstancesGroup`.

```yml

  ECSStackSet:
    Type: 'AWS::CloudFormation::StackSet'
    DependsOn: VPCStackSet
    Properties:
      StackSetName: ws02-ecs
      Description: ECS stack set on Singapore and Tokyo
      PermissionModel: SELF_MANAGED
      ExecutionRoleName: AWSCloudFormationStackSetExecutionRole
      Capabilities:
        - CAPABILITY_IAM
        - CAPABILITY_AUTO_EXPAND
      ManagedExecution:
        Active: true
      Tags:
        - Key: test
          Value: true
      TemplateURL: !Ref ECSTemplate
      Parameters:
        - ParameterKey: VpcStackExportPrefixName
          ParameterValue: ws02
      OperationPreferences:
        FailureToleranceCount: 0
        MaxConcurrentCount: 1
        RegionConcurrencyType: PARALLEL
      StackInstancesGroup:
      # Primary region
        - DeploymentTargets:
            Accounts:
              - !Ref AWS::AccountId
          Regions:
            - ap-southeast-1 # Singapore
      # Disaster region
        - DeploymentTargets:
            Accounts:
              - !Ref AWS::AccountId
          Regions:
            - ap-northeast-1 # Tokyo
          ParameterOverrides:
            - ParameterKey: BucketName
              ParameterValue: tvm-cf-templates-tokyo

```
