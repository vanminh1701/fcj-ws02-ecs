---
title : "Monitor"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---

ECS hỗ trợ tính năng xem log của tất cả container được triển khai trong cluster bằng cách bật tính năng *Container Insights*. Bạn có thể cấu hình trong CloudFormation template như sau:

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

Sau khi bật tính năng này, bạn có thể đọc các log của các container trong bảng điều khiển **Logs** bên trong từng service. Ví dụ như xem log của container `web`:

![Container Logs](/images/3.8-container-logs.png)


<!-- Kiểm tra log connect trong sidecar container -->