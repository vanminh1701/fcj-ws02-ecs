---
title : "Giới thiệu"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
---

## Xây dựng ứng dụng trên Amazon ECS

### Tổng quan

Trong bài viết này sẽ giới thiệu về  tổng quan về dịch vụ Amazon ECS, các thành phần xây dựng nên dịch vụ như cách triển khai CD cho một ứng dụng mà dịch vụ này hỗ trỡ, cách kết nối với ứng dụng từ internet và kết nối giữa ứng dụng với ứng dụng mà không cần đi ra internet. Ngoài ra bài biết cũng giới thiệu về dịch vụ CloudFormation giúp xây dựng hệ thống bằng IaC một cách nhanh chóng và dễ dàng. Chúng ta cũng sẽ xây dựng một ý tưởng về khả năng phục hồi khi có thảm họa với chiến lược "Đèn hoa tiêu"

### Content

1. [Giới thiệu ECS](1-introduce/)
2. [Giới thiệu CloudFormation](2-intro-cloudformation/)
3. [Ứng dụng Demo](3-demo)
4. [Chiến lược khắc phục thảm họa](4-dr/)
5. [Clean up](5-cleanup/)

![Architecture](/images/ws02-Architecture.drawio.svg)
