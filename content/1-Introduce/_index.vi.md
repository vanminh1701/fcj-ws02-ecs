---
title : "Giới thiệu ECS"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---

#### Giới thiệu về dịch vụ ECS

ECS là một container Orchestration được quản lý bởi AWS giúp cho dễ dàng triển khai, quản lý và mở rộng các ứng dụng container hóa. ECS được tích hợp với một số công cụ thứ 3 như ECR và docker giúp cho team tập trung phát triển ứng dụng hơn là setup môi trường. Bạn có thể tạo và chạy các ứng dụng trên nhiều Region, on-premise mà không cần quan tâm về phần điều khiển.

#### Kiến trúc hệ thống ECS

ECS gồm có 3 layer:

- Capacity: hệ thống nơi bạn triển khai các container, có thể là EC2, Fargate hoặc on-premise
- Controller: phần AWS quản lý giúp triển khai và quản lý các container
- Provisioning: Các công cụ để tương tác với controller: console, CLI, SDK, Copilot, CDK
  
![ECS 3 layer Architecture](/images/1.1-ecs-3-layers.png)

Đối với Capacity layer, ECS cung cấp 3 tùy chọn EC2, Fargate và On-premise. Tuy vào nhu cầu mục đích mà bạn lựa chọn loại capacity cho phù hợp:

|          | Đặc tính | Ưu điểm | Nhược điểm |
| ---------|---------|---------|---------|
| EC2      | Sử dụng dịch vụ AWS EC2 để chạy các máy ảo chứa cái ứng dụng container | Tối ưu về chi phí khi hệ thống đã vận hành ổn định | Tốn nổ lực vận hành, giám sát, nâng cấp hệ thống |
| Fargate   | Là 1 dịch vụ container Serverless mà AWS quản lý về hệ thống | Không cần quan tâm về hệ thống bên dưới, khả năng mở rộng lớn | Giá thành cao hơn khi dùng EC2 |
|On-premise | kết nối các máy chủ bên ngoài AWS, phục vụ cho mục đích đặc thù | | |

#### Vòng đời ứng dụng

Hình dưới đây mô tả cách một úng dụng container hoạt động với ECS: 

![Application lifecycle](/images/1.2-ecs-application-lifecycle.png)

  1. Ứng dụng của bạn phải được đóng gói thành các docker image, có thể được lưu trữ trên ECR hoặc docker registry nào đó để ECS có thể truy cập để tải docker image.
  2. Sau khi có image, bạn cần tạo một *Task definition,* là một bản thảo cho cấu hình cài đặt như: thông tin image, các thông số của OS, port sử dụng, bộ lưu trữ, thông số kết nối, …
  3. Để triển khai các container, bạn tạo một Service/ Task trong các cluster. Cluster được hiểu là các service hoặc task được tập hợp với nhau được vận hành trên các Capacity Provider đã đăng ký với nó. Cluster sẽ đọc task definition và triển khai các container tương ứng. Bạn có thể chạy nhiều task bằng ECS service để đảm duy trì và đảm bảo số lượng task chạy đồng thời trong cluster.
Ngoài ra đối với EC2 Capacity, ECS sẽ cần có một *container agent* trên từng máy chủ với mục đích là gửi các thông tin về task, tài nguyên sử dụng của container để  chạy và tắt các task cũng như giám sát thông số hệ thống.

Sau khi container đã hoạt động, bạn có thể  giám sát việc triển khai và ứng dụng bằng các công cụ được AWS hỗ trợ: CloudWatch, Runtime Monitoring.

#### Cluster

Cluster là một nhóm các task hoặc service liên quan với nhau có thể về chức năng hoặc toàn bộ dự án của bạn, nó bao gồm:

- Cơ sở hạ tầng có thể là dự kết hợp giữa EC2, Fargate và on-prem.
- Cấu hình network trong 1 VPC
- Kết nối với Namespace để hỗ trợ giao tiếp giữa các service trong và ngoài cluster.
- Tuy chọn giám sát bằng *Container Insight* để thu thập, tổng hợp và tổng kết các thông số và log.

Cluster bao gồm nhiều capability provider, được hiểu là một nhóm hệ thống các máy chủ độc lập có chung cấu hình đặc thù và khả năng mở rộng, có thể là một Auto Scaling Group, Fargate provider.

| Fargate | EC2 Auto Scaling Group |
| --------| ---------------------- |
| Quản lý task bằng Fargate hoặc Fargate Spot (dịch vụ Fargate có giảm giá nhưng không ổn định). Provider này bị một số hạn chế về hỗ trợ OS, thao tác trên Console. | Cluster cung cấp tính năng tự động quản lý số lượng instance trong group và quá trình mở rộng đáp ứng nhu cầu của các service |

#### Task definition

Là một bản thảo giúp cấu hình các thông số để triển khai 1 hoặc nhiều container trong ECS cluster của bạn, một thông số có thể thay đổi như:

- Launch type: xác định ứng dụng sẽ chạy trên loại provider nào
- Thông tin docker image
- Thông số CPU, memory cần thiết
- Hệ điều hành của server
- Chế độ mạng dùng để kết nối với container
- Điều kiện task thực thi nhiệm vụ nào đó và tự tắt đi
- IAM Role giúp tương tác với các dịch vụ AWS khác
- Cấu hình logging, kho lưu trữ

Sau khi khởi tạo xong, *Task definition* có thể được áp dụng dưới 2 hình thức:

- Task: là một container để thực thi 1 nhiệm vụ cụ thể  nào đó và tự động tắt đi sau khi thực thi xong
- Service: là một cơ chế quản lý các container, giúp đảm bảo số lượng container mong muốn thực thi. Nếu 1 task được phát hiện bị lỗi, service schedular của ECS sẽ thực thi cơ chế tạo 1 task mới thay thế và tắt task bị lỗi đi.
  
Chế  độ mạng của *task* dùng để cấu hình các cách giao tiếp của container, đặc biệt khi được chạy trong các EC2 instance. Nếu không có yêu cầu đặc thù nào, AWS khuyến khích sử dụng chế độ *awsvpc*:

- Dối với Fargate, bạn chỉ có thể cấu hình ché độ *awsvpc*
- Đôi với EC2, bạn co nhiều chế độ hơn và *awsvpc* giúp đơn giản hóa cáu hình mạng để bạn chỉ cần tập trung vào phát triển ứng dụng.

Bảng liệt kê các chế độ mạng

|       Chế  độ          | Mô tả |
| :--------------------: | ----- |
|   awsvpc (default)     |  Task được cấp phát 1 ENI và 1 địa chỉ IPv4 riêng, nó cho phép task có đặc tính mạng giống với EC2 instance  |
|   bridge               | Sử dụng docker network mặc định của dịch vụ docker chạy trong instance |
|   host                 | Task sẽ bỏ qua Docker network để map container port trực tiếp vào ENI của EC2 instance. Điều này khiển các task giống nhau không thể được triển khai trên cùng 1 instance  |
|   none                 | Task sẽ không nhận kết nối từ bên ngoài |

#### Triển khai một ứng dụng lên ECS cluster

ECS hỗ trợ triển khai ứng dụng theo 3 tùy chọn:

- Rolling update: được ECS quản lý bằng service scheduler
- Blue/Green: được kết hợp với dịch vụ CodeDeploy
- External: sử dụng các dịch vụ bên thứ 3 để gọi API
  
Khi sử dụng Console để tạo một service, AWS sẽ chọn mặc định tùy chọn *rolling update* và các cấu hình liên quan để tạo service. Bạn có thể thay đổi tùy chọn trong mục **Deployment options**

***Rolling update***

Đây là tùy chọn để ECS quản lý quá trình triển khai ứng dụng. Khi bạn thực hiện thay đổi service và kích hoạt triển khai, service scheduler sẽ khỏi tạo một số lượng task mới và có thể tắt một số task cũ đi tùy theo bạn quy định. Sau khi task mới đã ở trạng thái ổn định, ECS sẽ tiến hành tăt hoàn toàn các task cũ và tiến trình hoàn tất. Nếu bạn triển khai dịch vụ kết hợp với một ELB, sau khi task sẵn sàng nhận traffic, ECS sẽ tự động đăng ký task với ELB để chuyển dần traffic sang task mới. Vì vập cần lưu ý việc kiểm trang sức khoẻ của task dể đảm bảo service mới sẵn sàng nhận traffic.

Bạn có thể cấu hình triển khai theo 2 thông số sau để quy định số lượng task mới và cũ trong quá trình triển khai:

- *minimumHealthPercent*: số lượng task cũ tối thiểu được giữ lại. Ví dụ giá trị là 75% và đang có 4 task, service scheduler sẽ tắt đi 1 task, giữ lại 3 task và tiến hành tạo task mới
- *maximumPercent*: xác định số lượng task tối đa cho service, bao gồm task cũ và mới. Ví dụ giá trị 200% và số task mong muốn là 4 thì service scheduler có thể tạo được tối đa là 8 task nên sẽ tạo thêm được 4 task mới chạy song song với 4 task cũ.

Khi triển khai ứng dụng, Ứng dụng có thể bị lỗi dẫn tới triển khai ứng dụng không thành công. ECS cung cấp 2 phương pháp để xác định lỗi và tiến hành rollback ứng dụng lại phiên bản cũ là circuit breaker và CloudWatch alarms. Bạn có thể kết hợp cả hai cách để có thể xác định lỗi tốt nhất cho ứng dụng.

- Circuit breaker: ECS sẽ đánh giá trạng thái của từng task và dừng việc triển khai khi số lỗi đạt tới một ngưỡng nhất định.
- CloudWatch Alarms: đánh giá dựa vào các thông số của ứng dụng. ECS sẽ theo dõi thông số cảnh báo mà bạn cài đặt trước. Khi thông số CloudWatch ở trạng thái ALARM, ECS sẽ rollback ứng dụng của bạn.

***Blue/Green với CodeDeploy***

Tùy chọn này được kết hợp với dịch vụ CodeDeploy để quản lý và giám sát quá trình triển khai. Với tùy chọn này bạn sẽ có nhiều lựa chọn hơn trong việc phân phối traffic giữa task mới và cũ theo 3 loại sau:

- Canary: chuyển 1 phần traffic sang task mới và sau 1 khoảng thời gian nhất định, chuyển toàn bộ traffic còn lại sang task mới
- Linear: chuyển traffic dần theo thời gian. Ví dụ 20% mỗi 5 phút
- All-at-once: sau khi các task mới đã ởn định, chuyển toàn bộ traffic cùng lúc

Một số cân nhắc khi dùng Blue/Green:

- Bạn cần tạo một vài tài nguyên bên dịch vụ CodeDeploy và liên kết với ECS.
- ECS cần tạo thêm thành phần *task set* để cấu hình cho quá trình triển khai, làm tăng tính phức tạp khi cấu hình.
- Bắt buộc phải kết nối với một load balancer để có thể quy định phân phối traffic
- Quá trình này sẽ bị xung đột với việc mở rộng dẫn tới những kết quả không lường trước được. Cho nên cần trành việc triển khai khi hệ thống đang chịu tải lớn.

***External***

ECS cung cấp các API để tích hợp với các dịch vụ thứ 3 điều khiển, giám sát quá trình triển khai, ví dụ như Jenkins, ...

#### Tự động mở rộng service

Đây là khả năng tự tăng giảm số lược task mong muốn một cách tự động. ECS tận dụng dịch vụ Application AUto Scaling để cung cấp tính năng này. ECS sẽ xuất bản các thông số CloudWatch về sử dụng CPU và bộ nhớ của service và bạn có thể dụng những thông số này để mở rộng service của bạn. Application Auto Scaling hỗ trợ 3 loại mở rộng sau:

- *Target tracking metric* - Tăng hoặc giảm số lượng task dựa trên một thông số nhất định. Cơ chế tương tự cách hoạt động của nhiệt kế, đảm bảo cho thông số của service duy trì ở mức nhỏ hơn hoặc bằng một số nhất định.
- *Step scaling* - thay đổi số lượng task theo một loạt điều chỉnh mở rộng dựa vào kích thước của mức độ vượt quá cảnh báo. Ví dụ vượt quá 20% thì tăng thêm 1 task, 50% thì tăng 3 task. 
- *schedule scaling* - thay đổi dựa theo thời gian. Ví dụ chạy 5 task vào ban ngày và 1 task vào ban đêm.

#### Kết nối giữa các service

Một trong những cân nhắc khi dùng ECS là làm thế nào để kết nối vào các service bên trong cluster. Đối với truy cập từ internet, bạn có thể kết hợp với 1 Elastic Load Balancer. Và khi bạn muốn kết nối này không đi ra ngoài internet, bạn có thể dùng tính năng kết nối nội bộ bằng Service Connect hoặc Service Discovery.

- Service Connect
  
  Dùng cách này để cung cấp khả năng kết nối service tới service bằng việc thiết lập một service discovery và service mesh để tạo một hê thống network cho cluster. Điểm vượt trội của tính năng này là nó dùng service mesh để quản lý network và có thể chuyển kết nối sang các service ở trạng thái tốt khi một service bị lỗi một cách tức thời.
  
- Service Discovery
  
  Đây là một cách kết nối truyền thống, nó sẽ tạo ra một hệ thống DNS để lưu IP của từng task và gắn vào một tên miền cụ thể. Bởi vì các DNS này sẽ được cache lại IP mỗi khi truy vấn nên khi có task bị lỗi, service cần đợi tới thời gian hệ hiệu lực của cache và truy vấn thông tin DNS mới nhất thì mới có thể dùng tiếp được. Một điểm nổi bật của hej thống này là cho phép bạn có thể kết nối vào service từ bên ngoài cluster. Ví dụ như bạn có thể gọi đến service từ một EC2 instance không cùng chung network.
