---
title : "Cài đặt CloudFormation"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

Phần này sẽ đi vào các bước để triển khai một ứng dụng áp dụng các kiến thức đã giới thiệu ở phần trên.

#### Điều kiện tiên quyết

Trên máy bạn đã cài đặt AWS CLI và đã cấu hình kết nối với một IAM user có đủ quyền.

Sử dụng các template trong Github Repo sau: [Github link](https://github.com/vanminh1701/fcj-ws02-cfn-ecs)

#### Thiết kế ứng dụng

![CloudFormation Architecture](/images/3.1-cloudformation-architecture.svg)

Ứng dụng được thiết kế  sử dụng StackSet để triển khai trên 2 Region và Nested Stack để gom các tài nguyên liên có liên quan lại với nhau, giúp cho việc quản lý stack dễ dàng và có hệ thống hơn.
StackSet bao gồm 3 stack:

- VPC - triển khai một mạng lưới VPC để quản lý tách biệt về mạng. VPC được tách thành 1 stack riêng để nhằm mục đích mở rộng cho các service khác, và chia sẻ các tài nguyên bằng cách Export giá trị cho các stack khác có thể sử dụng được
- ECS - triển khai 1 cụm ECS và các máy chủ tính toán, một ECR để lưu trữ Docker image và một Application Load Balancer nhận truy cập từ người dùng  
- Database - triển khai một cơ sở dữ liệu PostgreSQL và một bản sao chỉ đọc ở Region khác phục vụ cho chiến lược khắc phục thảm họa  
  
Để đáp ứng cho chiến lược khắc phục thảm họa được nói đến ở phần sau, chúng ta sẽ tận dụng tính năng StackSet của CloudFormation giúp dễ dàng triển khai hệ thống trên nhiều Region khác nhau mà không cần phải thực hiện tạo stack ở từng Region. Tùy theo loại khắc phụ thảm họa mà sẽ có những điều kiện để triển khai stack tương ứng:

- Dịch vụ ECR hỗ trợ tính năng tự động sao chép sang Region khác nên chỉ cần triển khai trên Region chính
- Tương tụ với dịch vụ RDS, chỉ cần khởi chạy một database ở Region chính và một bản sao chỉ đọc ở Region thứ yếu
- Đối với cụm ECS ở Region thứ yếu, vì nó chỉ được dùng khi có sự cố thảm họa xảy ra và tiêu chí đặt ra về RTO không quá khắc khe và không cần phải xử lý yêu cầu ngay lập tức. Do đó, ta sẽ thêm điều kiện để không phải khởi chạy bất cứ máy chủ nào ở Region thứ yếu.

#### Khởi chạy stack

Đầu tiên cần tải các tệp mẫu lên một S3 bucket để CloudFormation có thể đọc các mẫu Nested Stack và StackSet. Lưu ý, thay `tvm-cf-templates` bằng S3 bucket của bạn và thay thế toàn bộ trong các tệp mẫu.
Sau đó có thể tiến hành tạo stack bằng AWS CLI

```sh
aws s3 sync . s3://tvm-cf-templates --exclude ".git/*"
aws cloudformation create-stack --stack-name stack-set \
            --template-body file://stackset.yml \
            --capabilities CAPABILITY_NAMED_IAM
```

Trong bài lab này, Region chính được chọn ở Singapore (ap-southeast-1) và Region thứ yếu ở Tokyo (ap-northeast-1). Ngoài ra mặc định khi triển khai các stack, CloudFormation có khả năng nhận biết thứ tự tạo các stack nhờ vào các tham chiếu cấu hình trong tệp. Nhưng đối với trường hợp nhập các giá trị từ một stack khác mà không có mẫu tham chiếu, bạn cần xác định điều kiện này một cách thủ công bằng thuộc tính `DependsOn`. Ví dụ cần phải tạo VPC trước khi tạo các resource khác nên các stack ECS và Database sẽ `DependsOn` VPC stack.

```yml
Resources:
  VPCStackSet:
    Type: 'AWS::CloudFormation::StackSet'
  ECSStackSet:
    Type: 'AWS::CloudFormation::StackSet'
    DependsOn: VPCStackSet
  DataBaseStackSet:
    Type: 'AWS::CloudFormation::StackSet'
    DependsOn: VPCStackSet
```

StackSet trên nhiều Region, ta có thể  khởi tạo các stack chạy song song hoặc tuần tự bằng thuộc tính `RegionConcurrencyType` là `PARALLEL` hoặc `SEQUENTIAL`

```yml
Resources:
  VPCStackSet:
    Type: 'AWS::CloudFormation::StackSet'
    Properties:
      OperationPreferences:
        RegionConcurrencyType: PARALLEL
  DataBaseStackSet:
    Type: 'AWS::CloudFormation::StackSet'
    DependsOn: VPCStackSet
    Properties:
      OperationPreferences:
        RegionConcurrencyType: SEQUENTIAL
        RegionOrder:
          - ap-southeast-1 # This Region run first
          - ap-northeast-1
```

Đối với `VPCStackSet` vì VPC tách biệt hoàn toàn ở 2 Region nên lựa chọn `PARALLEL` để giúp giảm thời gian khởi tạo stack `DataBaseStackSet` cần chạy tuần tự bởi vì cần phải hoàn thành khởi tạo database trước khi tạo bản sao cho nó. Thứ tự chạy được quy định trong mục `RegionOrder`, như trong cấu hình `ap-southeast-1` sẽ được chạy trước cho đến khi hoàn thành và `ap-northeast-1` sẽ bắt đầu chạy.
