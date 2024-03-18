---
title : "Monitor"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---

ECS supports log viewing of all containers deployed in the cluster by enabling *Container Insights*. You can configure the CloudFormation template as follows:

```yml
Cluster:
  Type: AWS::ECS::Cluster
  Properties:
    ClusterName: !Ref ClusterName
    ClusterSettings:
      - Name: containerInsights
        Value: enabled
```

You can check whether enable the feature on Console:

![ECS container insights](/images/3.7-container-insights.png)

After enabling this feature, you can read the container logs in the **Logs** panel within each service. For example, look at the log of the `web` container:

![Container Logs](/images/3.8-container-logs.png)
