# The zcp-jenkins Installation Guide

## ZCP-jenkins Component 

| Component        | Version           | Image  | etc |
| ------------- |-------------|-----|----|
|Jenkins| 1.121.3 |jenkins/jenkins:1.121.3


## 개요

zcp-jenkins 설치 전에 필요한 것들.

1. zcp-system-admin Service Account. - zcp-iam 설치 시 생성됨.
2. https 용 공인 인증서 secret - keycloak 설치 시 생성됨.

## helm client install
zcp-jenkins 는 helm 으로 설치하기 때문에 자신의 OS 에 맞는 helm 2.9.1 client 가 필요하다. 
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
$ cd zcp-installation/zcp-jenkins
```

Jenkins home pvc를 설치한다.
```
$ kubectl apply -f zcp-jenkins-pvc.yaml
# AKS 인 경우
$ kubectl apply -f zcp-jenkins-pvc-aks.yaml
```

Maven local repository pvc를 설치한다.
```
$ kubectl apply -f zcp-jenkins-maven-pvc.yaml
# AKS 인 경우 
$ kubectl apply -f zcp-jenkins-maven-pvc-aks.yaml
```

## Deploy the application
프로젝트 별로 수정해야 하는 파일은 **values** 이다.


### 1. values.yaml 정보 변경
private 환경인 경우 values-ibm.yaml(AKS 는 values-aks.yaml) 을 수정한다.
`# CAHNGE` 주석이 포함된 라인의 정보를 수정한다.
```
Master:
  HostName: jenkins.cloudzcp.io   # CHANGE
...

...

Master:
  Ingress:
    Annotations:
      # ingress.bluemix.net/ALB-ID: private-xxxx-alb1  # CHANGE: Private ALB (IKS 인 경우만 수정한다.)
    TLS:
    - hosts:
      - jenkins.cloudzcp.io  # CHANGE
    
...

```

### 2. helm install로 설치한다.
public 환경인 경우
```
$ helm install jenkins-0.14.3.tgz -n zcp-jenkins -f values.yaml --namespace=zcp-system 
```

private 환경인 경우
```
$ helm install jenkins-0.14.3.tgz -n zcp-jenkins -f values-ibm.yaml --namespace=zcp-system 
```
AKS 인 경우
```
$ helm install jenkins-0.14.3.tgz -n zcp-jenkins -f values-aks.yaml --namespace=zcp-system 
```

### 3. Jenkins Plugins 복사
검증이 완료 된 버전의 Jenkins plugin 을 사용하기 위해 values 파일의 InstallPlugins 를 사용하지 않고 Plugin 을 직접 Jenkins pod 에 복사한다.

Jenkins pod 에 완전히 기동되었는지 확인한다. Running 상태여야 파일 복사가 가능하다.
```
$ kubectl get po -l app=zcp-jenkins -n zcp-system
NAME                           READY     STATUS    RESTARTS   AGE
zcp-jenkins-86c545fc87-n84vl   1/1       Running   0          1d
```

git 에서 plugin 파일을 받는다. 용량이 커서 별도의 repository 로 관리하고 있다.
```
$ cd ../..
$ git clone https://github.com/cnpst/zcp-jenkins-plugins.git
```

plugin을 jenkins pod 으로 복사한다.
```
$ cd zcp-jenkins-plugins
$ kubectl cp plugins  `(kubectl get po -l app=zcp-jenkins -o jsonpath='{.items[0].metadata.name}' -n zcp-system)`:var/jenkins_home -n zcp-system
```

Jenkins 를 재기동한다.
```
$ kubectl scale deploy zcp-jenkins --replicas=0 -n zcp-system
$ kubectl scale deploy zcp-jenkins --replicas=1 -n zcp-system
```
