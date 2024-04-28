---
title : "Debug with ECS Exec"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

Similar to running `docker exec` in local docker containers, you can run commands or access containers in ECS service tasks with `aws exec command`. This aids in interacting directly with the container to execute commands to check whether the container is working as expected or not. You can refer to this for more information [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-exec.html).

This lab will use this feature to test the service-to-service network connection settings by accessing one container and querying another container using the configured DNS name using the configured Service Connect service.[here](https://github.com/vanminh1701/fcj-ws02-cfn-ecs/blob/5d068a323c32613fe7b02be54fee0a62227c4ee6/stacks/task-web.yml#L108)

#### Prerequisites

- You need to enable this feature when initializing the service for the task by configuring:
  
```yml
Service:
  Type: AWS::ECS::Service
  Properties:
    ServiceName: !Ref ServiceName
    Cluster: !Ref ClusterName
    EnableExecuteCommand: true
    ...
```

- At the same time, it is necessary to adjust the task role to grant permissions to ECS Exec, refer to [IAM role](https://github.com/vanminh1701/fcj-ws02-cfn-ecs/blob/90c4beff8c29749f25e566d775887c3ae63b2186/stacks/ecs.yml#L176)
  
```yml
Effect: Allow
Action:
  - ssmmessages:CreateControlChannel
  - ssmmessages:CreateDataChannel
  - ssmmessages:OpenControlChannel
  - ssmmessages:OpenDataChannel
```
  
- When the service is in the `RUNNING` state, you can access the container using the command
  
```sh
aws ecs execute-command \
  --cluster workshop-02 \
  --task arn:aws:ecs:us-east-1:471112666427:task/workshop-02/bf23faa8b0ba412bb721c00121caaffe \
  --container api \
  --command "sh" \
  --interactive
```

Note, replace `arn:aws:ecs:us-east-1:471112666427:task/workshop-02/bf23faa8b0ba412bb721c00121caaffe` with *taskId* and `workshop-02` with the name of the ECS cluster you created

- The **api** service container can now be accessed. You can query the **web** service
  
```sh
aws ecs execute-command
  --cluster workshop-02 \
  --task arn:aws:ecs:us-east-1:339712716339:task/workshop-02/244892b6fe03415d911e5713af39add8 \
  --container api \
  --command "sh" \
  --interactive
```

Use `ping -c5 api-svc.workshop-02` to test the connection to the API service.

![ECS Exec command](/images/3.6-exec-cmd.png)
