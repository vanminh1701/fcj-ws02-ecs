---
title : "ECS Exec"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

Tương tự như chạy `docker exec` ở các local docker container, bạn có thể chạy các lệnh hoặc truy cập vào container trong các task của ECS service bằng lệnh `aws exec command`. Việc này hỗ trợ trong việc tương tác trực tiếp với container để thực hiện các lệnh kiểm tra container có hoạt động đúng với mong đợi hay không. Bạn có thể tham khảo thêm [tại đây](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-exec.html).

Bài lab này sẽ sử dụng tính năng này để kiểm tra việc cài đặt kết nối mạng giữa service với service bằng cách truy cập vào container này và truy vấn tới container khác bằng tên DNS đã được cấu hình bằng dịch vụ Service Connect được cấu hình [tại đây](https://github.com/vanminh1701/fcj-ws02-cfn-ecs/blob/5d068a323c32613fe7b02be54fee0a62227c4ee6/stacks/task-web.yml#L108)

Điều kiện tiên quyết:

- Bạn cần bật tính năng này khi khởi tạo service cho task bằng cách cấu hình:
  
```yml
Service:
  Type: AWS::ECS::Service
  Properties:
    ServiceName: !Ref ServiceName
    Cluster: !Ref ClusterName
    EnableExecuteCommand: true
    ...
```

- Đồng thời cần phải điều chỉnh task role để cấp quyền cho ECS Exec, tham khảo [IAM role](https://github.com/vanminh1701/fcj-ws02-cfn-ecs/blob/90c4beff8c29749f25e566d775887c3ae63b2186/stacks/ecs.yml#L176)
  
```yml
Effect: Allow
Action:
  - ssmmessages:CreateControlChannel
  - ssmmessages:CreateDataChannel
  - ssmmessages:OpenControlChannel
  - ssmmessages:OpenDataChannel
```
  
- Khi service ở trạng thái `RUNNING`, bạn có thể truy cập vào container bằng lệnh
  
```sh
aws ecs execute-command \
  --cluster workshop-02 \
  --task arn:aws:ecs:us-east-1:471112666427:task/workshop-02/bf23faa8b0ba412bb721c00121caaffe \
  --container api \
  --command "sh" \
  --interactive
```

  Lưu ý thay `arn:aws:ecs:us-east-1:471112666427:task/workshop-02/bf23faa8b0ba412bb721c00121caaffe` bằng *taskId* và `workshop-02` bằng tên của ECS cluster bạn đạ tạo

- Bây giờ đã có thể truy cập vào container của **api** service. Bạn có thể truy vấn đến **web** service
  
```sh
aws ecs execute-command
  --cluster workshop-02 \
  --task arn:aws:ecs:us-east-1:339712716339:task/workshop-02/244892b6fe03415d911e5713af39add8 \
  --container api \
  --command "sh" \
  --interactive
```

Sử dụng `ping -c5 api-svc.workshop-02` để kiểm tra kết nối với API service.

![ECS Exec command](/images/3.6-exec-cmd.png)
