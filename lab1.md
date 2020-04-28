# Lab 1 - Prepare for OpenFaaS

## Yêu cầu trong lab này.

Cài đặt được Docker, OpenFaaS CLI và Kubernetes

## Docker

**Cài đặt**

> Nên sử dụng quyền cao nhất trong Ubuntu `root`

Gỡ cài đặt phiên bản cũ của Docker
```
apt-get remove docker docker-engine docker.io containerd runc
```

Thực hiện từng lệnh 
```
apt-get update

apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

Cài đặt Docker
```
apt-get update

apt-get install docker-ce docker-ce-cli containerd.io
```

Chạy thử Docker hello-world
```
docker run hello-world
```

> Để chạy Docker trên user mà không dùng tài khoản root. Dùng lệnh sau, chú ý `logout` hoặc `reboot` để lưu cấu hình.

```
usermod -aG docker your-user
```

Gỡ cài đặt Docker

```
apt-get purge docker-ce docker-ce-cli containerd.io

rm -rf /var/lib/docker
```

Tham khảo cài đặt [ở đây](https://docs.docker.com/engine/install/ubuntu/)

## OpenFaaS CLI

**Cài đặt**

```
curl -sLSf https://cli.openfaas.com | sudo sh
```

Kiểm tra version

```
faas-cli version
```

## Đăng nhập Docker Hub

> Bạn phải có tài khoản Docker Hub. Đăng ký [ở đây](https://hub.docker.com)

Đăng nhập

```
docker login
```

Khi thực hiện các bài lab sẽ tạo ra các images, để các images đó được đồng bộ lên Docker Hub bạn cần thực hiện

> Chỉnh sửa `~/.bashrc` hoặc `~/.bash_profile` - nếu không có hệ thống sẽ tự tạo

Sau đó thêm đường dẫn sau vào

```
# Populate with your Docker Hub username
export OPENFAAS_PREFIX="username docker-hub"
```

## Kubernetes

**Cài đặt kubectl**

```
export VER=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)

curl -LO https://storage.googleapis.com/kubernetes-release/release/$VER/bin/linux/amd64/kubectl

chmod +x kubectl

mv kubectl /usr/local/bin/
```

**Thiết lập cluster Kubernetes**

Cài đặt k3d 

```
curl -s https://raw.githubusercontent.com/rancher/k3d/master/install.sh | bash
```

*Tham khảo [ở đây](https://github.com/rancher/k3d)*

Tạo cluster

```
k3d create
```
ouput:
```

```

## Deploy OpenFaaS

**Cài đặt OpenFaaS**

Sử dụng `arkade`

```
curl -SLsf https://dl.get-arkade.dev/ | sudo sh
```

Chạy OpenFaaS app

```
arkade install openfaas --load-balancer

hoặc

arkade install openfaas
```

## Đăng nhập vào OpenFaaS Gateway

Kiểm tra gateway có sẵn sàng không

```
kubectl rollout status -n openfaas deploy/gateway
```

Thực hiện lệnh sau để forward ra trình duyệt

```
kubectl port-forward svc/gateway -n openfaas 8080:8080
```

URL gateway: `http://127.0.0.1:8080`

> Lưu URL vào `~/.bashrc` hoặc `~/.bash_profile`

```
# Populate as above
export OPENFAAS_URL=""
```

Lấy mật khẩu đăng nhập Gateway

```
PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)

echo -n $PASSWORD 

# Đăng nhập Gateway trong faas-cli
faas-cli login --username admin --password=$PASSWORD
```

Kiểm tra

```
faas-cli list
```

Giờ qua [Lab2](lab2.md)