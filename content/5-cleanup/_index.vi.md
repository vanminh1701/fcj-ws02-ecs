---
title: "Clean up"
date: "`r Sys.Date()`"
weight: 5
pre: "<b>5. </b>"
---

Một ưu điểm khi quản lý hạ tầng bằng CloudFormation là các tài nguyên sẽ được tập trung quản lý trong các *stack*. Vì vậy bạn có thể dễ dàng tắt hệ thống đi khi không cần dùng đến cho hệ thống chỉ chạy trong khung giờ cố định, hoặc không dùng đến hệ thống này nữa.

Các Stack sẽ được xóa đi theo thứ tự ngược với lúc khởi tạo vì CloudFormation sẽ giúp nhận biết Stack Export value đang được dùng ở những Stack khác và không cho phép bạn xóa stack hiện tại đi.
Một ví dụ thông báo như hình bên dưới:

![Cannot Remove stack](/images/4.1-cannot-remove-stack.png)

Khi xóa parent stack, các nested stack sẽ được xóa theo nên bạn chỉ cần xóa ECS stack xong và sau đó xóa VPC stack.

Để xóa stack, bạn truy cập vào CloudFormation Console, chọn Stack cần xóa và chọn **Delete** ở khung bên phải trong phần mô trả chi tiết của stack.

![Remove stack](/images/4.2-remove-stack.png)

Truy cập vào S3 console để xóa bucket đã tạo để chứa CloudFormation đã tạo trước đó. Các bước xóa S3:

1. Vì S3 bucket có chứa các object nên cần làm trống trước khi có thể xoắ bucket đi. Chọn **Empty** để xóa các object bên trong bucket.
2. Chọn **Delete** để xóa bucket

![Remove stack](/images/4.3-remove-s3-bucket.png)
