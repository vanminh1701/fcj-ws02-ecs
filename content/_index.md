---
title : "Intro"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
---

Một ưu điểm khi quản lý hạ tầng bằng CloudFormation là các tài nguyên sẽ được tập trung quản lý trong các *stack*. Vì vậy bạn có thể dễ dàng tắt hệ thống đi khi không cần dùng đến cho hệ thống chỉ chạy trong khung giờ cố định, hoặc không dùng đến hệ thống này nữa.

Các Stack sẽ được xóa đi theo thứ tự ngược với lúc khởi tạo vì CloudFormation sẽ giúp nhận biết Stack Export value đang được dùng ở những Stack khác và không cho phép bạn xóa stack hiện tại đi.
Một ví dụ thông báo như hình bên dưới:

![Cannot Remove stack](/images/4.1-cannot-remove-stack.png)