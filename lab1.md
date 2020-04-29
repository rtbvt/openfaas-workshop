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

output:

```
root@ubuntu:~# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: rtbvt
Password: 
WARNING! Your password will be stored unencrypted in /home/rt/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
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
root@ubuntu:~# k3d create
INFO[0000] Created cluster network with ID b8940c99953550e7263c851d969ed2c3589fd2d075f89e7cf45822115eb8cc12 
INFO[0000] Created docker volume  k3d-k3s-default-images 
INFO[0000] Creating cluster [k3s-default]               
INFO[0000] Creating server using docker.io/rancher/k3s:v1.17.3-k3s1... 
INFO[0000] Pulling image docker.io/rancher/k3s:v1.17.3-k3s1... 
INFO[0019] SUCCESS: created cluster [k3s-default]       
INFO[0019] You can now use the cluster with:

export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"
kubectl cluster-info 
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
output:
```
root@ubuntu:~# arkade install openfaas --load-balancer
Using kubeconfig: /home/rt/.config/k3d/k3s-default/kubeconfig.yaml
Using helm3
Node architecture: "amd64"
Client: "x86_64", "Linux"
2020/04/28 02:43:09 User dir established as: /home/rt/.arkade/
https://get.helm.sh/helm-v3.1.2-linux-amd64.tar.gz
/home/rt/.arkade/bin/helm3/linux-amd64 linux-amd64/
/home/rt/.arkade/bin/helm3/helm linux-amd64/helm
/home/rt/.arkade/bin/helm3/README.md linux-amd64/README.md
/home/rt/.arkade/bin/helm3/LICENSE linux-amd64/LICENSE
2020/04/28 02:43:13 extracted tarball into /home/rt/.arkade/bin/helm3: 3 files, 0 dirs (2.894247938s)
"openfaas" has been added to your repositories
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "openfaas" chart repository
Update Complete. ⎈ Happy Helming!⎈ 
VALUES values.yaml
Command: /home/rt/.arkade/bin/helm3/helm [upgrade --install openfaas openfaas/openfaas --namespace openfaas --values /tmp/charts/openfaas/values.yaml --set clusterRole=false --set queueWorker.replicas=1 --set faasnetes.imagePullPolicy=Always --set basicAuthPlugin.replicas=1 --set gateway.replicas=1 --set ingressOperator.create=false --set basic_auth=true --set gateway.directFunctions=true --set operator.create=false --set openfaasImagePullPolicy=IfNotPresent --set serviceType=LoadBalancer]
Release "openfaas" does not exist. Installing it now.
NAME: openfaas
LAST DEPLOYED: Tue Apr 28 02:43:18 2020
NAMESPACE: openfaas
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
To verify that openfaas has started, run:

  kubectl -n openfaas get deployments -l "release=openfaas, app=openfaas"
=======================================================================
= OpenFaaS has been installed.                                        =
=======================================================================

# Get the faas-cli
curl -SLsf https://cli.openfaas.com | sudo sh

# Forward the gateway to your machine
kubectl rollout status -n openfaas deploy/gateway
kubectl port-forward -n openfaas svc/gateway 8080:8080 &

# If basic auth is enabled, you can now log into your gateway:
PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
echo -n $PASSWORD | faas-cli login --username admin --password-stdin

faas-cli store deploy figlet
faas-cli list

# For Raspberry Pi
faas-cli store list \
 --platform armhf

faas-cli store deploy figlet \
 --platform armhf

# Find out more at:
# https://github.com/openfaas/faas

Thanks for using arkade!
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