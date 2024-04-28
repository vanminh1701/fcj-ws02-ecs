---
title : "Monitor"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---

Sau khi đã hoàn thành triển khai cụm ECS, chúng ta có nhiều phương pháp để giám sát sức khỏe cũng như hiệu năng của hệ thống. Bài viết này xin giới thiệu 3 phương thức giám sát hệ thống dùng Container Insights để ghi lại các thông số hiện tại của hệ thống và các log của từng container, từ đố xây dựng cơ chế thông báo khi có lỗi ứng dụng được log ghi lại và cuối cùng là cảm báo cho nhà phát triển khi có những trạng thái thay đổi xảy ra trong cụm sử dụng công cụ EventBridge. Các thông báo sẽ được tổng hợp lại và sau đó gửi về ứng dụng Slack giúp cho nhóm phát triển dễ dàng nhận thông báo.

#### Giám sát bằng Container Insights

ECS hỗ trợ tính năng xem log của tất cả container được triển khai trong cluster bằng cách bật tính năng *Container Insights*. Nó cho phép bạn tổng hợp các thông số đo lường cho cụ thể cho từng container đang hoạt động, ví dụ như thông số Bộ nhớ, CPU, số lần khởi động lại bị lỗi, ... Bạn có thể cấu hình trong CloudFormation template như sau:

```yml
Cluster:
  Type: AWS::ECS::Cluster
  Properties:
    ClusterName: !Ref ClusterName
    ClusterSettings:
      - Name: containerInsights
        Value: enabled
```

Bạn có thể kiểm tra tính năng trên Console:

![ECS container insights](/images/3.7-container-insights.png)

Ngoài ra, bạn có thể đọc các log của các container trong bảng điều khiển **Logs** bên trong từng service. Container Log được thu thập và gửi lên CloudWatch Log khi tiến hành tạo TaskDefinition. Để cấu hình, ta thêm phần `LogConfiguration` cho từng container trong Task:

```yml
  TaskDefinition:
    Type: AWS::ECS::TaskDefinition
    Properties:
      ContainerDefinitions:
        - Name: !Ref ServiceName
          LogConfiguration:
            LogDriver: awslogs
            Options:
              mode: non-blocking
              max-buffer-size: 25m
              awslogs-group: !Ref LogGroup
              awslogs-region: !Ref AWS::Region
              awslogs-stream-prefix: !Ref ServiceName
```

Ví dụ như xem log của container `web`:

![Container Logs](/images/3.8-container-logs.png)

#### Thông báo lỗi ứng dụng bằng CloudWatch Alarm

Container Insights giúp tự động ghi các log từ container vào dịch vụ CloudWatch Log group. Từ đó ta có thể thiết lập cơ chế kiểm tra các lỗi ứng dụng được ghi lại bằng mẫu sự kiện được thiết lập trước. Dịch vụ này sẽ đọc tất cả các log được ghi lại và phát cảnh báo khi có lỗi xảy ra.

Trong bài này chúng ta sẽ tạo một Metric Filter dựa theo mẫu `%ERROR%` để lọc các dòng log lỗi ứng dụng, và cấu hình thêm CloudWatch Alarms cho từng Metric Filter để  gửi thông báo về nhà phát triển

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::LanguageExtensions'
Description: ECR repository
Parameters:
  SNSTopic:
    Type: String
    Default: arn:aws:sns:ap-southeast-1:761048055120:NotiSlack
Mappings:
  LogGroupNames:
    Task:
      API: filter
      Web: for-metric-filter

Resources:
  'Fn::ForEach::MetricFilters':
    - Identifier
    - [API, Web]
    - 'MetricFilter${Identifier}':
        Type: AWS::Logs::MetricFilter
        Properties:
          LogGroupName: !FindInMap [LogGroupNames, Task, !Ref Identifier]
          FilterPattern: '%ERROR%'
          MetricTransformations:
            - MetricValue: 1
              MetricNamespace: ws02
              MetricName: !Join ["-" ,["ws02", !FindInMap [LogGroupNames, Task, !Ref Identifier]]]
              Unit: Count
              DefaultValue: 0
      'Alarm${Identifier}':
        Type: AWS::CloudWatch::Alarm
        DependsOn: !Join ["", [MetricFilter, !Ref Identifier]]
        Properties:
          AlarmName: !Join ["-" ,["ws02", !FindInMap [LogGroupNames, Task, !Ref Identifier]]]
          AlarmDescription: Alarm for metric filter
          AlarmActions:
            - !Ref SNSTopic
          MetricName: !Join ["-" ,["ws02", !FindInMap [LogGroupNames, Task, !Ref Identifier]]]
          Namespace: ws02
          Statistic: Sum
          Period: 10
          EvaluationPeriods: 2
          DatapointsToAlarm: 2
          Threshold: 10
          ComparisonOperator: GreaterThanOrEqualToThreshold
```

#### Thông báo trạng thái thay đổi bằng EventBridge

AWS EventBridge là một công cụ mạnh mẽ hỗ trợ bắt các sự kiện cho hầu hết các dịch vụ AWS. Trong bài viết này chúng ta sẽ sử dụng công cụ này để bắt các sự kiện thay đổi trạng thái trong cụm ECS và thông báo cho nhà phát triển. Ví dụ như khi có 1 task gặp lỗi và tự động tắt đi hoặc khi số lượng traffic tăng lên dẫn đến việc mở rộng thêm nhiều đơn vị tính toán.

```yaml
  ECSClusterStateChangeRule:
    Type: AWS::Events::Rule
    Properties:
      Description: EventBridge rule for ECS cluster state changes
      EventBusName:  !Ref EventBusName 
      EventPattern: 
        source: 
          - aws.ecs
        detail-type: 
          - ECS Task State Change
          - ECS Container Instance State Change
          - ECS Deployment State Change
        detail: 
          clusterArn: 
            - !Ref ClusterARN
      Name: ws02-ecs-cluster-state-change
      State: ENABLED
      Targets: 
        - Arn: arn:aws:sns:ap-southeast-1:761048055120:NotiSlack
          Id: NotiSlack
        - Id: CWLog
          Arn: >-
            arn:aws:logs:ap-southeast-1:761048055120:log-group:/aws/events/ecs-ws02
```
