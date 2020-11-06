# MinIO Spike

해당 문서는 [MinIO](https://min.io/)는 `Kebernetes Native, High Performance Object Storage`이다. <br/>
Performance와 S3 API에 맞춰서 디자인 되었고 100% 오픈 소스입니다. 그외에 아래와 같은 부분을 강화되었다고 한다.

- mission-critical availability
- stringent security requirement
- large, private cloud environments

여기서 몇 가지 중요한 개념을 다시 집고 넘어 가도록 하자.  <br/>

`cloud native`라는 용어는 Bill Wilder의 책 Cloud Architecture Patterns에서 처음으로 사용되었고 아래와 같은 특징을 가진 어플리케이션을 의미한다.
- Use cloud platform services.
- Scale horizontally.
- Scale automatically, using proactive and reactive actions.
- Handle node and transient failures without degrading.
- Feature non-blocking asynchronous communication in a loosely coupled architecture.

그렇다면 해당 솔루션에서 말하고 있는 `Kubernetes Native`는 무엇을 의미할까? <br/>
우선 Cloud Native Technology를 세 가지로 분리하면 아래와 같다. <br/>

- Kubernetes Accommodative Technologies
- Kubernetes Native Technologies
- Non-Kubernetes Technologies

그리고 여기서 `Kubernetes Native`하다라는 것은 쿠버네티스의 시스템에 맞게끔 구동된다는 것이다.<br/>
가령, Operator를 통한 시스템 관리 kubectl를 통한 명령어 제공 등과 같은 Kubernetes 친화적이라는 의미이다.


## Table of Contents
- [MinIO Architecture](#MinIO-Architecture)
- [Install MinIO](#Install-MinIO)
- [MinIO on MAC](#MinIO-on-MAC)
- [MinIO on MiniKube](MinIO-on-MiniKube)
- [OpenFaaS on MiniKube](#OpenFaaS-on-MiniKube)
- [Issues](#Issues)


## MinIO Architecture

아래 그림이 MinIO 구성의 중요한 점을 잘 보여주고 있다. 기존  Storage 서비스를 사용할 경우에는 `Gateway` 방식으로 적용하고 <br/>
독단적인 서버 역할을 할 경우 서버(Standalone or Distributed Servers)로 구성할 수 있다.

![MinIO Architecture](https://min.io/resources/img/products/multi-cloud-gateway.svg)


## MinIO on MAC
우선 맥에서 MinIO 서버를 설치하고 구동하는 법을 살펴보자.

### Install MinIO

```bash
brew install minio/stable/minio
```

### Up and Running MinIO
프로그램을 설치하고 특정 경로의 폴더를 mount 경로로 지정하여 준다.

```bash
minio server /data
```



## MinIO on MiniKube
쿠버네티스 환경에 설치하기 위하여 helm을 사용해 보자.


1. 설치
```
helm install --set accessKey=minioadmin,secretKey=minioadmin --generate-name minio/minio
```

2. 키 받아오기
```
ACCESS_KEY=$(kubectl get secret minio -o jsonpath="{.data.accesskey}" | base64 --decode) and the SECRET_KEY=$(kubectl get secret minio -o jsonpath="{.data.secretkey}" | base64 --decode
```
3. 접근하기
```
export POD_NAME=$(kubectl get pods --namespace default -l "release=minio-1604666686" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 9000 --namespace default
```


## MinIO Client(mc) Commands
MinIO Client 프로그램 명칭은 `mc`이다. 해당 파일의 설정은 `~/.mc/config.json` 이다.

```
cat ~/.mc/config.json
```
### MinIO Event Trigger to OpenFaaS

`MinIO/S3` 이벤트 트리거 방법은 `Kafka` 혹은 `Webhook`를 제안하고 있다.

- MinIO에서 이벤트 종류 확인
```
mc admin config get local/
```

### Publish MinIO event via Webhooks

1. Add Webhook endpoint to MinIO

- 확인
```
$ mc admin config get local notify_webhook
notify_webhook:1 endpoint="" auth_token="" queue_limit="0" queue_dir="" client_cert="" client_key=""
```

- 추가
```
$ mc admin config set local notify_webhook:1 queue_limit="0"  endpoint="http://gateway.openfaas:8080" queue_dir=""
Setting new key has been successful.
Please restart your server with `mc admin service restart local`.
```

- 적용을 위한 재시작
```
$ mc admin service restart local
Restart command successfully sent to `local`.
Restarted `local` successfully.
```

2. Set Wh

> [참고](https://docs.openfaas.com/reference/triggers/)

## OpenFaaS on MiniKube

1. Create OpenFaaS namespace
```
kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml
```

2. Add OpenFaaS helm repo
```
helm repo add openfaas https://openfaas.github.io/faas-netes/
```

3. Update Helm repo
```
helm repo update
```

4. Generate Password
```
export PASSWORD=$(head -c 12 /dev/urandom | shasum| cut -d' ' -f1)
echo $PASSWORD
```

5. Create Secret
```
kubectl -n openfaas create secret generic basic-auth --from-literal=basic-auth-user=admin --from-literal=basic-auth-password="$PASSWORD"
```
- 비밀번호 얻어오기
```
kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo
```

6. Install OpenFaaS
```
helm upgrade openfaas --install openfaas/openfaas --namespace openfaas --set functionNamespace=openfaas-fn --set basic_auth=true
```

7. Expose Gateway
```
kubectl port-forward -n openfaas svc/gateway 8080:8080 &
```

## Issues

1. Minikube Insufficient Memory
> 💀 만약 설치 시, 메모리가 부족하다는 메시지로 설치가 되지 않는다면 해당 로컬 쿠버네티스를 삭제하고 다시 설치해야 한다. <br/>

> ```
> minikube delete
> 🔥  virtualbox 의 "minikube" 를 삭제하는 중 ...
> 💀  "minikube" 클러스터 관련 정보가 모두 삭제되었습니다
> ```

> **메모리 사이즈 증가**
> ```
> minikube --memory 8192 --cpus 2 start
> ...
> 🏄  Done! kubectl is now configured to use "minikube" by default
> ```

> **확인 방법**
> ```
> cat ~/.minikube/config/config.json                                     {
>     "WantReportError": true,
>     "dashboard": true,
>     "ingress": true,
>     "memory": 8192
> }
> ```


## Reference
- [MinIO Reference](http://min.io)
- [OpenFaaS Deployment Guide](https://github.com/openfaas/faas-netes/blob/master/chart/openfaas/README.md)