---
title : "Các bước chuẩn bị"
date: 2024-01-01
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

**Chuẩn bị tài nguyên với AWS CloudFormation**

Để tiến hành triển khai hạ tầng, chúng ta sẽ sử dụng dịch vụ **AWS CloudFormation** thông qua AWS Console hoặc AWS CLI.

**Nội dung**
- [CloudFormation Stack](#cloudformation-stack)
- [Template URL](#template-url)
- [Parameters](#parameters)
- [Stack Outputs](#stack-outputs)

#### CloudFormation Stack
{{% notice note %}}
Bài thực hành sẽ được thiết lập ở **us-east-1 (North Virginia)**.
{{% /notice %}}

1. Đăng nhập vào AWS Console, truy cập vào dịch vụ [CloudFormation](https://us-east-1.console.aws.amazon.com/cloudformation/home)
2. Tiến hành tạo một Stack mới bằng việc nhấn nút `Create Stack`.
3. Ở trang `Specify template`, chúng ta có 2 cách:
   1. Nhập đường dẫn của **Template URL** vào ô `Amazon S3 URL`.
   2. Tiến hành tải lên Temlate có sẵn bằng cách dùng nút `Upload a template file`.
4. Ở trang `Parameters`, chúng ta sẽ nhập một số thông tin bắt buộc.

![cloudformation-create-stack-parameters](/images/2-cloudformation-create-stack-parameters.png?featherlight=false&width=90pc)

5. Ở trang `Specify Stack Details`, chọn nút `Next`.
6. Ở trang `Configure stack options`, chọn nút `Next`.
7. Ở trang `Capabilities`, tiến hành chấp nhận (Acknowledge) cho phép Template tạo các IAM role, cuối cùng chọn nút `Create Stack`.

![cloudformation-stack-status](/images/2-cloudformation-stack-status.png?featherlight=false&width=90pc)

{{% notice info %}}
Quá trình trên sẽ diễn ra trong vòng 5-10 phút cho tới khi chúng ta thấy được trạng thái của Stack là `CREATE_COMPLETE`.
{{% /notice %}}

#### Template URL
| Thành Phần | Giá Trị (Bắt buộc) |
| ---------- | ------- |
| Stack Name | smdemo |
| Template URL | [RDSFargate.yml](https://s3.amazonaws.com/sa-security-specialist-workshops-us-east-1/secrets-manager-workshop/RDSFargate.yml) |

Hoặc tải về template bên dưới:

{{%attachments title="Template" pattern=".*(yml)"/%}}

#### Parameters
{{% notice info %}}
Các giá trị dưới đây đã được định nghĩa là giá trị mặc định tại Template.
{{% /notice %}}

| Thành Phần | Giá Trị (Bắt buộc) |
| ---------- | ------- |
| NamePrefix | smdemo |
| ProjectTag | smproj |
| DBName | smdemo |
| DBPort | 3306 |
| AmazonLinux2AmiId | /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 |

#### Stack Outputs
Sau khi CloudFormation Stack đã được chuẩn bị hoàn tất, chúng ta tiến hành ghi chú tất cả các giá trị ở mục *Outputs*.

Về cơ bản, các **Output Key** sau sẽ được sử dụng:
| Output Key | Định Nghĩa |
| ---------- | ---------- |
| BastionIP | Địa chỉ IP của *Bastion Host* |
| DBInstance | ID của RDS Instance |
| DBUser | Master User của RDS database |
| DBPassword | Master User's Password của RDS database |
| EC2UserPassword | Password của `ec2-user` |
| ECRRepository | ECR repository |
| ECSCluster | ECS cluster |

![cloudformation-stack-output](/images/2-cloudformation-stack-output.png?featherlight=false&width=90pc)

{{% notice tip %}}
Chúng ta cần lưu ý và ghi chú các giá trị trên bởi chúng sẽ được sử dụng cho các phần kế tiếp.
{{% /notice %}}