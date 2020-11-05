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


## MinIO Architecture

![MinIO Architecture](https://min.io/resources/img/products/multi-cloud-gateway.svg)


## Install MinIO

### Installing on Mac OS

```bash
brew install minio/stable/minio
```

```bash
minio server /data
```

### Installing on Kubernetes

```

```