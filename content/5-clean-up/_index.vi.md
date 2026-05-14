---
title : "Dọn dẹp tài nguyên"
date: 2024-01-01
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

**Dọn dẹp Tài nguyên**

Để thu dọn các tài nguyên AWS đã được khởi tạo trong phần thực hành, chúng ta sẽ tiến hành lần lượt các bước dưới đây.

**Nội dung**
- [Chạy shell script ```cleanup.sh```](#chạy-shell-script-cleanupsh)
- [Xoá Secret tại Secret Manager](#xoá-secret-tại-secret-manager)
- [Xoá CloudFormation Stack](#xoá-cloudformation-stack)

#### Chạy shell script ```cleanup.sh```
1. Truy cập vào dịch vụ [Systems Manager](https://us-east-1.console.aws.amazon.com/systems-manager)
2. Tại thanh điều hướng bên tay trái, chọn **Session Manager**.
3. Chọn EC2 instance tương ứng với giá trị **BastionIP**.
4. Nhấn nút `Start Session`.
5. Ở thanh cửa sổ mới, chúng ta sẽ tiến hành chạy các lệnh cần thiết.

{{% notice note %}}
Trước khi chạy các scripts, chúng ta cần đảm bảo tài khoản được sử dụng là `ec2-user`
{{% /notice %}}.

Tiến hành chuyển qua tài khoản `ec2-user` bằng câu lệnh sau:

```bash
sudo su - ec2-user
```

![shell-cleanup](/images/5-shell-cleanup.png)

Tiến hành dọn dẹp ECS Cluster và Amazon ECR với shell script sau:

```bash
./cleanup.sh
```

#### Xoá Secret tại Secret Manager
1. Truy cập vào dịch vụ [Secrets Manager](https://us-east-1.console.aws.amazon.com/secretsmanager).
2. Xoá *Secret* đã được tạo ở phần trước.

#### Xoá CloudFormation Stack
1. Truy cập vào dịch vụ [CloudFormation](https://us-east-1.console.aws.amazon.com/cloudformation/home)
2. Tìm kiếm Stack có tên với định dạng sau `-RDSMySQLRotationSingleUser-`. Đây là Stack được dùng cho quá trình **Key Rotation**
3. Tiến hành xoá CloudFormation Stack trên.
4. Sau khi quá trình hoàn tất, chúng ta sẽ tiến hành xoá Stack được dựng ở phần chuẩn bị môi trường.
5. Tìm kiếm Stack có tên với định dạng sau `smdemo`.
6. Tiến hành xoá CloudFormation Stack trên.

![cloudformation-stacks](/images/5-cloudformation-stacks.png)