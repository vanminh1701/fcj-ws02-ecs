---
title : "Introduction CloudFormation"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

### Giới thiệu dịch vụ CloudFormation

CloudFormation là một dịch vụ cho phép bạn thiết kế và cài đặt các tài nguyên AWS bằng các bản thảo code. Một số ưu điểm khi sử dụng dịch này là:

- Đơn giản hóa việc quản lý hạ tầng

  Đối với phương pháp truyền thống, bạn sẽ tạo và quản lý từng dich vụ riêng lẻ trên AWS Console. Việc này khiến việc tạo các tài nguyên một cách riêng lẻ, độc lập và yêu cầu bạn phải hiểu rõ về thứ tự tài nguyên nào cần tạo trước và sau. Việc liên kết giữa các ứng dụng cũng trở nên phức tạp và dễ gây lỗi do bất cấn.
  Khi sử dụng CloudFormation, bạn có thể cấu hình các tài nguyên trong những bản thảo, giúp bạn có 1 cái nhìn tổng thể về hệ thống tốt hơn, và đối với các yêu cầu phụ thuộc giữa các tài nguyên, CloudFormation sẽ đảm bảo tạo theo đúng trình tự mà bạn mong muốn.

- Nhân rộng hệ thống một cách nhanh chóng

  Một vài ví dụ khi bạn cần nhân rộng hệ thống như bạn cần đảm bảo hệ thông có tính sẵn sàng cao và bạn cần triển khai hệ thống ở một Region khác, tạo môi trường UAT phục vụ cho việc kiểm tra các tính năng mới. Sau khi bạn đã có bản thảo cho một môi trường, bạn có thể tái sử dụng nó để có thể tạo một môi trường tương tự một cách nhanh chóng chỉ bằng vài thao tác lệnh với CloudFormation.

- Dễ dàng điều khiển và theo dõi các thay đổi trong hệ thống

  Với một số trường hợp như bạn cần nâng cấp hệ thống, thay đổi một vài thông số cấu hình. Bạn chỉ cần thay đổi cấu hình trong các bản thảo. CloudFormation có khả năng so sánh các thay đổi trong hạ tầng hiện tại và các thay đổi bạn vừa cật nhật. Thông thường bạn có thể dùng các công cụ quản lý mã nguồn để tạo các phiên bản cho hạ tầng của bạn.

  Trong quá trình thay đổi hạ tầng, CloudFormation cũng hỗ trợ bạn đáng kể trong việc xác định lỗi xảy ra và tiến hành rollback hạ tầng một cách tự động, đảm bảo downtime tối thiểu cho hệ thống.
  
#### Các khái niệm trong CloudFormation

***Template***

Là các bản thảo có định dạng JSON hoặc YAML để xác đinh các tài nguyên xây dựng nên hệ thống của bạn. Một ví dụ về một bản thảo giúp bạn cấu hình và tạo một EC2 instance đơn giản

```yml
AWSTemplateFormatVersion: 2010-09-09
Description: A sample template
Resources:
  MyEC2Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      ImageId: ami-0ff8a91507f77f867
      InstanceType: t2.micro
      KeyName: testkey
      BlockDeviceMappings:
        - DeviceName: /dev/sdm
          Ebs:
            VolumeType: io1
            Iops: 200
            DeleteOnTermination: false
            VolumeSize: 20
```

***Stacks***

Stack dùng để nhóm các tài nguyên liên quan giúp cho bạn dễ dàng hơn trong việc quản lý. Tất cả tài nguyên bên trong Stack được xác định bằng các *template*. Ví dụ bạn có thể tạo 1 Stack với các template bao gồm 1 ECS cluster vaf 1 Application Load Balancer.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: CloudFormation Project with ECS service 
Resources:
  ecsStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: !Sub https://${BucketName}.s3.${AWS::Region}.amazonaws.com/stacks/ecs.yml
      Parameters:
        ECSName: ECSName
  ecsALBStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: !Sub https://${BucketName}.s3.${AWS::Region}.amazonaws.com/stacks/alb.yml
      Parameters:
        ALBName: ALB
```

***Change sets***

Khi bạn cật nhật hạ tầng trong quá trình phát triển ứng dụng, bạn có thể xem các thay đổi sẽ được cật nhật trong hệ thống bằng CloudFormation *change sets*. Tính năng này giúp cho bạn có thể xem xét đánh giá mức độ ảnh hưởng tới hệ thống hiện tại và cân nhác trước khi thực thi thay đổi hạ tầng.

***Export Stack value***

Tính năng chia sẻ tài nguyên cho các Stack khác được giới hạn trong Region, có nghĩa là các Stack trong cùng 1 Region mới có thể tạo liên kết với các tài nguyên được chia sẻ. Để sử dụng tính năng này, bạn cần tạo các Export trong mục Outputs của CloudFormation template. Ví dụ sau đây để chia sẻ thông tin VPCId với tên `${AWS::StackName}-VpcId` trong đó `AWS::StackName` là một pseudo parameter được xác định sẵn bởi AWS để chỉ tên của Stack được tạo ra.

```yml
Outputs:
  VPC:
    Description: A reference to the created VPC
    Value: !Ref VPC
    Export:
      Name: !Sub ${AWS::StackName}-VpcId
```
