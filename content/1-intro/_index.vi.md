---
title : "Giới thiệu"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Giới thiệu

#### Kịch Bản
Chúng ta sẽ tiến hành tích hợp dịch vụ **Secrets Manager** với **Amazon RDS** và **AWS Fargate**.
1. Phần đầu tiên của bài thực hành nhắm đến việc truy cập đến **RDS Database** sẽ được quản lý thông qua **Secrets Manager**.
2. Phần thứ 2, chúng ta tiến hành quá trình **Secret Rotation** nhằm thay đổi mật khẩu định kỳ và thực hiện một số thử nghiệm với Shell Scripts.
3. Phần cuối, chúng ta sẽ thử truy cập vào **RDS Database** từ tầng ứng dụng với **Fargate Container**.

Bài thực hành này sẽ cần một số kiến thức về **Docker** cũng như **Linux**.
- Cấp độ: 300
- Thời lượng: 1-2 tiếng

#### Điều kiện cần:
  - Tài khoản AWS
  - IAM User (Admin)
  - Kiến thức về Linux
  - Kiến thức về Docker và AWS Fargate

#### Các chức năng của **CSF** (Cybersecurity Framework):
  - Phòng chống (Prevent)

#### Các góc nhìn bảo mật của **CAF** (Cloud Adoption Framework):
  - Khả năng phòng chống (Preventative)

#### Các dịch vụ AWS được sử dụng:
  - Amazon EC2
  - Amazon RDS
  - Amazon ECS
  - Amazon ECR
  - AWS Secrets Manager
  - Amazon VPC

#### Mô Hình Kiến Trúc
Mô hình kiến trúc mạng sẽ là một VPC với 2 Subnets. Trong đó, một **Bastion Host** dựa trên hệ điều hành *Amazon Linux 2*, sẽ được dùng để chạy các shell scripts lẫn AWS CLI. Ngoài ra, một **Private RDS Instance** là mục tiêu chính của bài thực hành, bởi chúng ta sẽ sử lưu trữ **RDS credential** với dịch vụ **Secrets Manager**. Bên cạnh đó, chúng ta sẽ triển khai **AWS Fargate** để có thể thử nghiệm kết nối tới **RDS database** lẫn từ **EC2 instance** và **Fargate Container**.

![workshop-architecture](/images/1-workshop-architecture.png?featherlight=false&width=60pc)
