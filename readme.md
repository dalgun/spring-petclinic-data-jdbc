# DEVSOPS 사전과제
> 사전과제 제출용으로  특정 기한 이후 제거 예정입니다

## Installation

OS X :

```sh
brew install minikube
```


## Prepared Minikube

Kubernetes Host 관련 작업을 선행합니다.

미니큐브 시작

```sh
minikube start
minikube ssh
$> sudo su -
#> mkdir /logs
#> chown -R 1000:1000 /logs
```


## Development setup

### git clone


```sh
git clone https://github.com/dalgun/spring-petclinic-data-jdbc.git
```

### docker hub push
#### Maven 사용 

Master Branch 그대로 명령어 실행

```sh
cd spring-petclinic-data-jdbc
mvn clean install
```

#### gradle 사용

Gradle Branch 로 변경 후 빌드

```sh
cd spring-petclinic-data-jdbc
git checkout gradle
gradle jib
```

## Apply for Kubernets local
```sh
cd spring-petclinic-data-jdbc/manifest
kubectl apply -f mysql.yml
kubectl apply -f petclinic.yml
kubectl apply -f ingress.yml
```

## Browser
```sh
minikube ip
```

![](complete.png)

## Check Point
1.gradle을 사용하여 어플리케이션과 도커이미지를 빌드한다.
> grade branch 에 작업하였습니다.
Jib 플러그인으로 도커 빌드 및 푸시,
다만, wro4j plugin 미적용으로 css 파일은 maven 빌드 후 나오는 파일을 직접 init 하여 작업했습니다.


2.어플리케이션의 log는 host의 /logs 디렉토리에 적재되도록 한다.
> /manifest 폴더에 petclinic.yml : volumeMount host-path 사용
  Logback fileappender minikube profile 일시 mount folder spring-boot-logging.log 생성


3.정상 동작 여부를 반환하는 api를 구현하며, 10초에 한번 체크하도록 한다. 3번 연속 체크에 실패하 면 어플리케이션은 restart 된다
> actuator health api 사용
    /manifest 폴더에 petclinic.yml 에 livenessProbe, periodSeconds 10초, failureThreshold 3 작성

4.종료 시 30초 이내에 프로세스가 종료되지 않으면 SIGKILL로 강제 종료 시킨다
> /manifest 폴더에 petclinic.yml : terminationGracePeriodSeconds: 30 적용


5.배포 시와 scale in/out 시 유실되는 트래픽이 없어야 한다.
>  /manifest 폴더에 petclinic.yml : RollingUpdate 전략, rollingUpdate default 값. 사용

6.어플리케이션 프로세스는 root 계정이 아닌 uid:1000으로 실행한다.
>  /manifest 폴더에 petclinic.yml  : 
 securityContext:
    runAsUser: 1000
    fsGroup: 1000

7.DB도 kubernetes에서 실행하며 재 실행 시에도 변경된 데이터는 유실되지 않도록 설정한다.
> /manifest/mysql.yml: PersistentVolumeClaim 사용

8. 어플리케이션과 DB는 cluster domain을 이용하여 통신한다.
> application-minikube.yml : jdbc:mysql://petclinic-mysql/petclinic 사용

9. nginx-ingress-controller를 통해 어플리케이션에 접속이 가능하다.
> /manifest/ingress.yml : ingress 적용

10. namespace는 default를 사용한다
> /manifest/petclinic.yml : namespace default 적용

11. README.md 파일에 실행 방법을 기술한다.
> GitHub Readme 작성

