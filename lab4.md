# Lab 4 - Go deeper with functions

## Triển khai biến môi trường
Cùng tìm hiểu sâu hơn quá trình xây dựng và tạo ra 1 function

```
faas-cli deploy --name env --fprocess="env" --image="functions/alpine:latest"
```

output:
```
root@ubuntu:~# faas-cli deploy --name env --fprocess="env" --image="functions/alpine:latest"
WARNING! Communication is not secure, please consider using HTTPS. Letsencrypt.org offers free SSL/TLS certificates.
Handling connection for 8080

Deployed. 202 Accepted.
URL: http://127.0.0.1:8080/function/env

```

Truy vấn HTTP headers
```
root@ubuntu:~# echo "" | faas-cli invoke env --query workshop=1
Handling connection for 8080
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=env-5fb9768c77-ckrnx
fprocess=env
KUBERNETES_SERVICE_HOST=10.43.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.43.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1
KUBERNETES_PORT_443_TCP_PORT=443
ENV_SERVICE_HOST=10.43.169.141
KUBERNETES_SERVICE_PORT_HTTPS=443
ENV_SERVICE_PORT=8080
ENV_PORT=tcp://10.43.169.141:8080
ENV_PORT_8080_TCP=tcp://10.43.169.141:8080
ENV_PORT_8080_TCP_PROTO=tcp
ENV_PORT_8080_TCP_PORT=8080
KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
ENV_SERVICE_PORT_HTTP=8080
ENV_PORT_8080_TCP_ADDR=10.43.169.141
HOME=/home/app
Http_Accept_Encoding=gzip
Http_Content_Type=text/plain
Http_X_Forwarded_For=127.0.0.1:36184
Http_X_Forwarded_Host=127.0.0.1:8080
Http_User_Agent=Go-http-client/1.1
Http_Method=POST
Http_ContentLength=-1
Http_Content_Length=-1
Http_Query=workshop=1
Http_Path=/
Http_Host=env.openfaas-fn.svc.cluster.local:8080
```

Trong Python có hàm hỗ trợ để lấy `os.getenv("Http_Query")`

```
root@ubuntu:~# curl -X GET $OPENFAAS_URL/function/env/some/path -d ""
Handling connection for 8080
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=env-5fb9768c77-ckrnx
fprocess=env
KUBERNETES_SERVICE_HOST=10.43.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.43.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1
KUBERNETES_PORT_443_TCP_PORT=443
ENV_SERVICE_HOST=10.43.169.141
KUBERNETES_SERVICE_PORT_HTTPS=443
ENV_SERVICE_PORT=8080
ENV_PORT=tcp://10.43.169.141:8080
ENV_PORT_8080_TCP=tcp://10.43.169.141:8080
ENV_PORT_8080_TCP_PROTO=tcp
ENV_PORT_8080_TCP_PORT=8080
KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
ENV_SERVICE_PORT_HTTP=8080
ENV_PORT_8080_TCP_ADDR=10.43.169.141
HOME=/home/app
Http_User_Agent=curl/7.58.0
Http_Accept=*/*
Http_Content_Type=application/x-www-form-urlencoded
Http_X_Forwarded_For=127.0.0.1:36676
Http_X_Forwarded_Host=127.0.0.1:8080
Http_Accept_Encoding=gzip
Http_Method=GET
Http_ContentLength=0
Http_Content_Length=0
Http_Path=/some/path
Http_Host=env.openfaas-fn.svc.cluster.local:8080
```

> Bạn có thể tận dụng điều này để code trong chương trình của bạn

Giờ ta lấy header

```
curl $OPENFAAS_URL/function/env --header "X-Output-Mode: json" -d ""
```

Trong Python cũng có hàm hỗ trợ, ta dùng `os.getenv("Http_X_Output_Mode")`

## Security: read-only filesystems

Tạo một hàm mới
```
faas-cli new --lang python3 ingest-file --prefix="rtbvt"
```

Update the handler:
```
import os
import time

def handle(req):
    # Read the path or a default from environment variable
    path = os.getenv("save_path", "/home/app/")

    # generate a name using the current timestamp
    t = time.time()
    file_name = path + str(t)

    # write a file
    with open(file_name, "w") as f:
        f.write(req)
        f.close()

    return file_name
```

Build ví dụ này bằng
```
faas-cli up -f ingest-file.yml
```

Kiểm tra thử
```
echo "Hello function" > message.txt

cat message.txt | faas-cli invoke -f ingest-file.yml ingest-file
```

output: File sẽ được lưu ở `/home/app/`
```
root@ubuntu:~/lab4# cat message.txt | faas-cli invoke -f ingest-file.yml ingest-file
Handling connection for 8080
/home/app/1589202075.313212
```

Giờ chỉnh sửa ingest-file.yml và chỉ định cho hàm `read-only`

```
...
functions:
  ingest-file:
    lang: python3
    handler: ./ingest-file
    image: alexellis2/ingest-file:latest
    readonly_root_filesystem: true
```

Deploy lại:
```
faas-cli up -f ingest-file.yml
```

Giờ thử nghiệm
```
echo "Hello function" > message.txt

cat message.txt | faas-cli invoke -f ingest-file.yml ingest-file
```

output:
```
root@ubuntu:~/lab4# cat message.txt | faas-cli invoke -f ingest-file.yml ingest-file
Handling connection for 8080
Server returned unexpected status code: 500 - exit status 1
Traceback (most recent call last):
  File "index.py", line 19, in <module>
    ret = handler.handle(st)
  File "/home/app/function/handler.py", line 13, in handle
    with open(file_name, "w") as f:
OSError: [Errno 30] Read-only file system: '/home/app/1589202394.9454262'
```

Để lưu error vào temp thì sửa biến môi trường `save_path`
```
...
functions:
  ingest-file:
    lang: python3
    handler: ./ingest-file
    image: alexellis2/ingest-file:latest
    readonly_root_filesystem: true
    environment:
        save_path: "/tmp/"
```

Kiểm tra lại lần nữa, tất cả mọi error được chuyển hết vào `/tmp/`
```
root@ubuntu:~/lab4# cat message.txt | faas-cli invoke -f ingest-file.yml ingest-file
Handling connection for 8080
/tmp/1589202623.9797423
```

## Chức năng đăng nhập

Tận dụng hàm `hello-openfaas` ở Lab3

Sửa code trong `handler.py`

```
import sys
import json

def handle(req):

    sys.stderr.write("This should be an error message.\n")
    return json.dumps({"Hello": "OpenFaaS"})
```

Build and deploy
```
faas-cli up -f hello-openfaas.yml
```

Kiểm tra
```
echo | faas-cli invoke hello-openfaas
```

Và kết quả là
```
This should be an error message.
{"Hello": "OpenFaaS"}
```

Thêm biến môi trường trong `hello-openfaas.yaml`
```
environment:
      combine_output: false
```

Deploy lại lần nữa, kiểm tra trong logs
```
kubectl logs deployment/hello-openfaas -n openfaas-fn
```

output:
```
root@ubuntu:~/lab3/1# kubectl logs deployment/hello-openfaas -n openfaas-fn
2020/05/11 13:22:07 Version: 0.18.1	SHA: b46be5a4d9d9d55da9c4b1e50d86346e0afccf2d
2020/05/11 13:22:07 Timeouts: read: 5s, write: 5s hard: 0s.
2020/05/11 13:22:07 Listening on port: 8080
2020/05/11 13:22:07 Writing lock-file to: /tmp/.lock
2020/05/11 13:22:07 Metrics listening on port: 8081
2020/05/11 13:24:35 Forking fprocess.
2020/05/11 13:24:35 stderr: This should be an error message.
2020/05/11 13:24:35 Wrote 22 Bytes - Duration: 0.131372 seconds
```

