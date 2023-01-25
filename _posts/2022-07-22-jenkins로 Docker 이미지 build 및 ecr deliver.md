---
title: jenkins로 Docker 이미지 build 및 ecr deliver
date: 2022-07-22
---

## jenkins로 Docker 이미지 build
### jenkins가 설치된 서버에 Docker 설치
```
yum -y update
yum -y install docker
```

### docker 설치 후 docker daemon 실행
```
sudo systemctl start docker.service
```

### jenkins에서 docker를 실행할 수 있도록 docker.sock 권한 수정
```
sudo chmod 666 /var/run/docker.sock
```

#### docker.sock
도커는 크게 서버와 클라이언트로 나뉩니다.  
실제로 컨테이너를 생성하고 실행하며 이미지를 관리하는 주체는 도커 서버이고, 이는 dockerd 프로세스로서 동작합니다. 
도커 엔진은 외부에서 API 입력을 받아 도커 엔진의 기능을 수행하는데, 도커 프로세스가 실행되어 서버로서 입력을 받을 준비가 된 상태를 도커 데몬이라고 합니다.  
도커 데몬은 API 입력을 받아 도커 엔진의 기능을 수행하는데, 이 API를 사용할 수 있도록 CLI를 제공하는 것이 도커 클라이언트입니다.  
사용자가 docker로 시작하는 명령어를 입력하면 도커 클라이언트를 사용하는 것이며, 도커 클라이언트는 입력된 명령어를 로컬에 존재하는 도커 데몬에게 API로서 전달합니다.  
이때 도커 클라이언트는 /var/run/docker.sock에 위치한 유닉스 소켓을 통해 도커 데몬의 API를 호출합니다.

## jenkins pipe line으로 Docker 이미지 build
### jenkins에서 플러그인 설치
젠킨스 설정화면의 플러그인 메뉴에서 Pipeline, Docker Pipeline 플러그인을 설치한다.

### 빌드할 리파지토리에 파이프라인 스크립트 및 이미지 빌드를 위한 도커파일 추가
아래의 스크립트와 도커파일을 리파지토리(ex - member-api)에 추가한다.
```
// jenkins pipeline script
node {
     stage('Clone repository') {
         checkout scm
     }

    stage('Compilation') {
        sh 'chmod +x ~/workspace/docker-image-test/mvnw'
        sh '~/workspace/docker-image-test/mvnw clean package -DskipTests'
    }

     stage('Build image') {
         app = docker.build("364650154543.dkr.ecr.ap-northeast-2.amazonaws.com/member-api")
     }

    // 아래 push image 스텝은 일반적인 도커허브가 아닌 aws ecr에 push하도록 설정되어있음.
     stage('Push image') {
         sh 'rm  ~/.dockercfg || true'
         sh 'rm ~/.docker/config.json || true'

         docker.withRegistry('https://364650154543.dkr.ecr.ap-northeast-2.amazonaws.com/member-api', 'ecr:ap-northeast-2:dev-pub1') {
             app.push("${env.BUILD_NUMBER}")
             app.push("latest")
     }
  }
}
```

```
#Dockerfile
FROM amazoncorretto:17
ADD ./target/member-api-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]

EXPOSE 18081
```

### jenkins job 생성 및 설정
jenkins > 새로운 Item > Pipeline 선택
