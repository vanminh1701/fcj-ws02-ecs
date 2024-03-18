---
title : "Setup CloudFormation"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

Phần này sẽ đi vào các bước để triển khai một ứng dụng áp dụng các kiến thức đã giới thiệu ở phần trên.

#### Điều kiện tiên quyết

Trên máy bạn đã cài đặt AWS CLI và đã cấu hình kết nối với một IAM user có đủ quyền.

Sử dụng các template trong Github Repo sau: [fcj-ws02-cfn-ecs](https://github.com/vanminh1701/fcj-ws02-cfn-ecs)

#### Thiết kế ứng dụng

![CloudFormation Architecture](/images/3.1-cloudformation-architecture.png)
Ứng dụng sẽ được chia thành 2 Stack lớn là VPC và ECS. Mục tiêu khi tách VPC độc lập khỏi ECS cluster để  có thể mở rộng thiết kế cho các dịch vụ khác dùng chung 1 VPC.CloudFormation Stack có tính năng chia sẻ thông tin các tài nguyên để các Stack khác có thể dùng chung. Ví dụ trong bài này, ECS Stack sẽ dùng các thông số `PrivateSubnets` và `VpcId` từ VPC Stack để tạo các tài nguyên.

Các bước để cài đặt ứng dụng:

- Khởi tạo VPC stack
  Để tiến hành tạo VPC stack, khởi chạy lệnh trong terminal:

```bash
aws cloudformation create-stack \
  --stack-name ws2-vpc \
  --template-body file://./vpc.yml \
  --parameters ParameterKey=EnvironmentName,ParameterValue=ws2-vpc
```

Kết quả trả về từ terminal như sau và ban có thể thấy được 1 Stack được tạo trên CloudFormation Console.

```sh
{
    "StackId": "arn:aws:cloudformation:us-east-1:471112666427:stack/ws2-vpc/fff8ced0-e4ba-11ee-aeef-0affc807a469"
}
```

![CloudFormation Architecture](/images/3.4-cloudformation-vpc-console.png)

Ngoài ra bạn có thể thay đổi thông tin mặc định của VPC stack bằng cách thêm vào tùy chọn `--parameters`. Các thông số cho phép thay đổi được liệt kê trong mục `Parameters`, gồm có: *VpcCIDR, PublicSubnet1CIDR, PublicSubnet2CIDR, PrivateSubnet1CIDR, PrivateSubnet2CIDR*

```yml
Parameters:
  EnvironmentName:
    Description: An environment name that is prefixed to resource names
    Type: String
  VpcCIDR:
    Description: Please enter the IP range (CIDR notation) for this VPC
    Type: String
    Default: 10.192.0.0/16
  PublicSubnet1CIDR:
    Description: Please enter the IP range (CIDR notation) for the public subnet in the first Availability Zone
    Type: String
    Default: 10.192.10.0/24
  PublicSubnet2CIDR:
    Description: Please enter the IP range (CIDR notation) for the public subnet in the second Availability Zone
    Type: String
    Default: 10.192.11.0/24
  PrivateSubnet1CIDR:
    Description: Please enter the IP range (CIDR notation) for the private subnet in the first Availability Zone
    Type: String
    Default: 10.192.20.0/24
  PrivateSubnet2CIDR:
    Description: Please enter the IP range (CIDR notation) for the private subnet in the second Availability Zone
    Type: String
    Default: 10.192.21.0/24
```

- Tạo 1 S3 bucket để lưu các template.

Dối với bước 1 vì VPC Stack độc lập và chỉ cần 1 file template nên có thể  link trực tiếp từ local file. Nhưng ECS bao gồm nhiều *nested stack* và CloudFormation cần biết đường dẫn tới các template đó bằng thông số `TemplateURL`. Vì vậy cần phải tạo 1 S3 bucket lưu các template trước khi tạo CloudFormation.

Để nhanh chóng upload file template lên S3, chúng ta sẽ sử dụng lệnh để  đồng bộ thư mục làm việc hiện tại lên s3 Bucket

```sh
S3_STACK_BUCKET=ws2-s3
aws s3 sync . s3://$S3_STACK_BUCKET/ --exclude ".git/*"
```

- Sau khi upload các template lên s3, chúng ta có thể tạo ECS task bằng lệnh:
  
```sh
STACK_NAME=ws2-ecs
S3_STACK_BUCKET=ws2-s3

# Create CF stack
aws cloudformation create-stack --stack-name $STACK_NAME \
    --template-url https://$S3_STACK_BUCKET.s3.amazonaws.com/main.yml \
    --capabilities CAPABILITY_IAM
```

Kêt quả thu được trên Console:
![CloudFormation Architecture](/images/3.5-ecs-stack.png)
