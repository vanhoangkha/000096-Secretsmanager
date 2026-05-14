---
title : "Sử dụng trên Fargate"
date: 2024-01-01
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

**Sử dụng Secrets Manager tương tác với Fargate**

{{% notice note %}}
Bạn cần phải hoàn thành phần **RDS Phase** trước khi có thể tiếp tục nội dung dưới đây.
{{% /notice %}}

#### Tổng quan
Ở phần này, chúng ta sẽ học cách sử dụng dịch vụ **Secrets Manager** với **AWS Fargate**.

**AWS Fargate** là một `compute engine` được sử dụng cho dịch vụ **Amazon ECS (Elastic Container Service)** cho phép chúng ta khởi tạo các Containers mà không cần phải quản lý các tài nguyên về tính toán hay phải cấu hình các VMs. Đây là một bước tiến giúp chúng ta giảm tải trong việc cân nhắc loại *EC2 Instance Type* nào là phù hợp để chạy ứng dụng của mình cũng như đưa ra các quyết định về *Scaling*. Khi đó, chúng ta sẽ tập trung hơn về thiết kế và xây dựng ứng dụng hiện tại.

CloudFormation Template được dùng cho bài thực hành này đã chuẩn bị cho chúng ta các shell scripts cần thiết để xây dựng **Docker Image** và **Amazon ECR (Elastic Container Registry)**. Bài thực hành sẽ tập trung về **Secrets Manager** hơn là về **Docker**, **Amazon ECS** hay **Amazon ECR**.

Về mô hình kiến trúc, các tài nguyên AWS được chuẩn bị sẽ có liên kết như sau:
![fargate-architecture](/images/4-fargate-architecture.png?width=40pc)

**Contents**
- [Tổng quan](#tổng-quan)
- [Chuẩn bị Docker Images cho Amazon ECR](#chuẩn-bị-docker-images-cho-amazon-ecr)
- [Cấu hình AWS Fargate](#cấu-hình-aws-fargate)
- [Truy cập đến Fargate Containers](#truy-cập-đến-fargate-containers)
- [Truy cập vào RDS Database](#truy-cập-vào-rds-database)
- [Kết luận và hiểu rõ hơn về quá trình](#kết-luận-và-hiểu-rõ-hơn-về-quá-trình)

#### Chuẩn bị Docker Images cho Amazon ECR

---
**Truy cập vào Bastion Host với Session Manager**

1. Truy cập vào dịch vụ [Systems Manager](https://us-east-1.console.aws.amazon.com/systems-manager)
2. Tại thanh điều hướng bên tay trái, chọn **Session Manager**.

![ssm-session-manager](/images/3-ssm-session-manager.png)

3. Nhấn nút `Start Session`.
4. Chọn EC2 instance tương ứng với giá trị **BastionIP** và nhấn nút `Start Session`.

![ssm-session-manager-start-session](/images/3-ssm-session-manager-start-session.png)

5. Ở thanh cửa sổ mới, chúng ta sẽ tiến hành chạy các lệnh cần thiết.

{{% notice note %}}
Trước khi chạy các scripts, chúng ta cần đảm bảo tài khoản được sử dụng là `ec2-user`.
{{% /notice %}}

Tiến hành chuyển qua tài khoản `ec2-user` bằng câu lệnh sau:

```bash
sudo su - ec2-user
```

![ssm-session-manager-new-session-bastion](/images/3-ssm-session-manager-new-session-bastion.png)

---
**Dựng Docker Image**

Chúng ta tiến hành dựng **Docker Image** bằng shell script `dockerbuild.sh`. Đoạn mã này sẽ tiến hành tạo ra Docker Image dựa trên `Dockerfile` được chuẩn bị sẵn (bởi CloudFormation).

```bash
./dockerbuild.sh
```

![shell-docker-build](/images/4-shell-docker-build.png)

{{% notice note %}}
Quá trình dựng Docker Image sẽ mất một vài phút để hoàn thành.
{{% /notice %}}

Sau khi quá trình hoàn tất, chúng ta kiểm tra Docker Image có tên với định dạng `-ecrre-` bằng câu lệnh sau:

```bash
docker images
```

![shell-docker-images](/images/4-shell-docker-images.png)

Sau khi đã xác nhận, chúng ta tiến hành đẩy Docker Image này đến Amazon ECR bằng shell script `dockertagandpush.sh`.

```bash
./dockertagandpush.sh
```

![shell-docker-push](/images/4-shell-docker-push.png)

{{% notice note %}}
Quá trình đẩy Docker Image đến Amazon ECR sẽ mất một vài phút để hoàn thành.
{{% /notice %}}

1. Truy cập vào [Amazon ECR](https://console.aws.amazon.com/ecr/repositories?region=us-east-1)
2. Xác nhận rằng Image mới nhất đã được đẩy lên.

![ecr-repository](/images/4-ecr-repository.png)

Sau khi quá trình hoàn tất, chúng ta tiến hành cấu hình **AWS Fargate** dựa trên **Amazon ECR**.

#### Cấu hình AWS Fargate
Để cấu hình **AWS Fargate**, trước tiên chúng ta cần cấu hình **Task Definition**.

1. Truy cập vào dịch vụ [Amazon ECS](https://us-east-1.console.aws.amazon.com/ecs).
2. Ở thanh điều hướng bên tay trái, chọn **Clusters**.

![ecs-cluster](/images/4-ecs-cluster.png)

3. Nhấn chọn Cluster có tên với định dạng sau `-ECSCluster-`.
4. Ở thanh điều hướng bên tay trái, chọn **Task Definitions**, chọn Task có tên với định dạng sau `-TaskDefinition-`.
5. Tiến hành tạo một phiên bản mới, nhấn nút `Create new revision`.

![ecs-task-definition](/images/4-ecs-task-definition.png)

6. Nhán nút `Configure via JSON`.

![ecs-task-definition-configure-json](/images/4-ecs-task-definition-configure-json.png)

7. Kéo đến phần **secrets**, thay thế `SECRETNAME` với *Secret* mà chúng ta đã tạo ở phần trước.

![ecs-task-definition-replace-secret](/images/4-ecs-task-definition-replace-secret.png)

8. Nhấn nút `Save`.
9. Nhấn nút `Create` để tiến hành khởi tạo.

![ecs-task-definition-new-revision](/images/4-ecs-task-definition-new-revision.png)

10. Nhấn nút `Actions`, chọn `Run Task`.

![ecs-task-definition-run-task](/images/4-ecs-task-definition-run-task.png)

11. Tiến hành chọn các giá trị sau:
   ![ecs-task-definition-run-task-config](/images/4-ecs-task-definition-run-task-config.png)
   1. `Launch Type`: **Fargate**
   2. `Cluster VPC`: Chọn VPC đã được chuẩn bị sẵn với CIDR là `10.200.0.0/16`.
   3. `Subnets`: Chọn tất cả.
   4. `Security Group`: Chọn tên có định dạng sau `-BastionSG-`.
12. Nhấn nút `Run Task`.
13. Đợi trạng thái từ **PROVISIONING**, chuyển sang **PENDING** và cuối cùng là **RUNNING**.

![ecs-cluster-tasks](/images/4-ecs-cluster-tasks.png)

Sau khi các Tasks đều ở trạng thái **RUNNING**, chúng ta tiến hành truy cập đến **Fargate Container**. Trước hết, chúng ta cần phải ghi chú lại địa chỉ IP address của một Fargate Container.

![ecs-cluster-tasks-detailed-networking](/images/4-ecs-cluster-tasks-detailed-networking.png)

#### Truy cập đến Fargate Containers
Từ Bastion Host, tiến hành truy cập vào Fargate Container. Khi bị hỏi mật khẩu, chúng ta sử dụng giá trị **EC2UserPassword**.

```bash
ssh <FARGATE_TASK_PRIVATE_IP>
```

![ssm-session-manager-remote-fargate](/images/4-ssm-session-manager-remote-fargate.png)

#### Truy cập vào RDS Database
Ở phần này, chúng ta sẽ sử dụng cách mới thông qua shell script `mysql.newway.sh`.

1. Chúng ta tiến hành thực thi `mysql.newway.sh` để truy cập vào RDS Database.
   ```bash
   ./mysql.newway.sh smdemo
   ```
2. Sau đó, thực thi các câu lệnh SQL để kiểm tra dữ liệu hiện có.
   ```sql
   use smdemo;
   show tables;
   select * from bookinfo;
   quit;
   ```
3. Kết quả sẽ tương tự như sau.

![shell-mysql-new-fargate](/images/4-shell-mysql-new-fargate.png)

#### Kết luận và hiểu rõ hơn về quá trình
Sau khi truy cập thành công vào RDS Database từ Fargate Container, chúng ta cần phải hiểu rõ rằng có nhiều cách để truy cập vào **RDS Database**, tuy nhiên, phương án nào là bảo mật và phù hợp với ứng dụng của chúng ta mới là điều quan trọng.

Đối với việc truy cập từ Bastion Host, chúng ta thấy được lợi ích khi sử dụng shell script để truy cập vào **RDS Database**, nếu như chúng ta sử dụng cách thức cũ là `hard-coded password` thì sau khi thực hiện quá trình **Secret Rotation**, cách thức này sẽ không đáp ứng được.

**Đối với việc truy cập từ Fargate Container:**

1. Đầu tiên, chúng ta sử dụng **Secrets Manager ARN** với vai trò là `Environment Variable` trong **Task Definition**, khi đó các Fargate Container có thể dễ dàng biết và thực hiện truy cập đến *Secret* đã tạo trước đó.
2. Về phía **Docker Image**, **Dockerfile** được sử dụng có vai trò kích hoạt shell script `startprocesses.sh`. Đoạn mã này sẽ tiến hành truyền các tham số `Environment Variable` vào bên trong Fargate Container.
3. Khi đó các Fargate Container có thể sử dụng giá trị thu được từ tham số `Environment Variable` là `TASKDEF_SECRET` để thực hiện truy cập đến **RDS Database**.

4. Sau đây là nội dung đoạn mã của `startprocesses.sh`.

```sh
touch /etc/profile.d/ecs.sh
chmod 644 /etc/profile.d/ecs.sh

env | \
  grep "^TASKDEF_" | \
  awk -F= '{printf "export %s=%c%s%c\n", $1, 39, $2, 39 }' \
  >> /etc/profile.d/ecs.sh
```

5. Từ Task Definition, chúng ta có thể biết được các `Envionment Variable` được định nghĩa của Fargate Container.

![ecs-task-definitions-container-environment-variable](/images/4-ecs-task-definitions-container-environment-variable.png)

6. Từ Fargate Container, chúng ta có thể xác nhận các giá trị của `Environment Variable` với lệnh sau:

```bash
env|grep TASKDEF_SECRET
```

![shell-grep-environment-variable](/images/4-shell-grep-environment-variable.png)

7. Kết quả sẽ có định dạng tương tự như sau:
```bash
TASKDEF_SECRET={
   "username": "<DBUser>",
   "password": "<DBPassword>",
   "engine": "mysql",
   "host": "<RDS_ENDPOINT>",
   "port": 3306,
   "dbname": "<RDS_DATABASE>",
   "dbInstanceIdentifier": "<RDS_ID>"
}
```
