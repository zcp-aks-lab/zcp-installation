# The zcp-git Installation Guide

## ZCP-git Component 

| Component        | Version           | Image  | etc |
| ------------- |-------------|-----|----|
|Gitea| 1.4 |gitea/gitea:1.4
|Postgres| 9.6.2 |postgres:9.6.2


zcp-git 는 Open Source Gitea 를 사용한다.

zcp-gitea 설치 전에 필요한 것들.

1. https 용 공인 인증서 secret - keycloak 설치 시 생성됨.

## helm client install
zcp-git 는 helm 으로 설치하기 때문에 자신의 OS 에 맞는 helm 2.9.1 client 가 필요하다. 
<https://github.com/helm/helm/releases/tag/v2.9.1/>
```
$ helm init --client-only
``` 

## Clone this project into the desktop
```
$ git clone https://github.com/cnpst/zcp-installation.git
```

## PVC 설치
configuration 파일 디렉토리로 이동한다.

```
$ cd zcp-installation/zcp-git
```

pvc를 설치한다.
```
$ kubectl apply -f zcp-git-pvc.yaml

# AKS 인 경우
$ kubectl apply -f zcp-git-pvc-aks.yaml
```

## Deploy the application
프로젝트 별로 수정해야 하는 파일은 **values** 이다.


### 1. values.yaml 정보 변경
private 환경인 경우 values-ibm.yaml 을 수정한다. (AKS 인 경우 values-ibm.yaml)
`# CAHNGE` 주석이 포함된 라인의 정보를 수정한다.
```
service:
  ingress:
    annotations:
      # ingress.bluemix.net/ALB-ID: private-xxxx-alb1  # CHANGE: Private ALB
    hosts:
      host: git.cloudzcp.io  #CHANGE
    tls:
    - hosts:
      - git.cloudzcp.io  # CHANGE
    
...


```

### 2. helm install로 설치한다.
public 환경인 경우
```
$ helm install gitea-0.6.0.tgz -n zcp-git -f values.yaml --namespace=zcp-system 
```

private 환경인 경우
```
$ helm install gitea-0.6.0.tgz -n zcp-git -f values-ibm.yaml --namespace=zcp-system 
```

AKS 인 경우
```
$ helm install gitea-0.6.0.tgz -n zcp-git -f values-aks.yaml --namespace=zcp-system 
```
