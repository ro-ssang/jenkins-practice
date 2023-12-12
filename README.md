# Jenkins

## 젠킨스 컨테이너 생성

```sh
docker run -d \
           -p 8080:8080 \
           -p 50000:50000 \
           -v jenkins_home:/var/jenkins_home \
           -v /var/run/docker.sock:/var/run/docker.sock \
           my-jenkins
```

### 8080 포트

* 젠킨스 웹 클라이언트 접속을 위한 포트

### 5000 포트

* 젠킨스 agents를 연결하기 위한 포트

### /var/jenkins_home

* 젠킨스에서 사용되는 데이터(플러그인, 설정 등...)들이 저장되어 있는 곳
* 컨테이너 내부에서 사용되는 유저가 호스트 머신의 폴더에 대한 권한이 없을 수도 있기 때문에, 바인드 마운트는 사용하지 않는 것이 좋다.

### /var/run/docker.sock:/var/run/docker.sock

* 젠킨스 내부에서 도커 명령어를 사용하기 위해, docker 소켓을 바인드 마운트 해야한다.

* 또한 이미지를 확장해서 이미지 내부에 도커를 설치해야 한다. (https://medium.com/gdgsrilanka/running-jenkins-on-docker-for-a-newbie-855ad376500b)

  ```dockerfile
  FROM jenkins/jenkins
  
  USER root
  
  RUN curl -sSL https://get.docker.com/ | sh
  ```

## data 백업하기

바인드 마운트 대신 도커 볼륨을 사용했다면, data를 백업하기 위해서는 다음과 같은 명령어를 실행한다.

```sh
docker cp $ID:/var/jenkins_home
```

## 엑시큐터 개수 설정

* 그루비 스크립트를 이용하여 엑시큐터 개수를 설정할 수 있다. 기본적으로 2개의 엑시큐터가 설정되어 있다.
* 단, 컨트롤러에서 엑시큐터의 개수는 0개인 것을 권장한다.

### `executors.groovy`

```groovy
import jenkins.model.*
Jenkins.instance.setNumExecutors(0) // Recommended to not run builds on the built-in node
```

### `Dockerfile`

```dockerfile
FROM jenkins/jenkins:lts
COPY --chown=jenkins:jenkins executors.groovy /usr/share/jenkins/ref/init.groovy.d/executors.groovy
```

## agents 연결하기

* 젠킨스의 컨트롤러 외부에서 빌드를 수행할 수 있으며, 컨트롤러에서의 엑시큐터 개수는 0개인 것을 권장한다.
* 인바운드 TCP 연결을 통해 에이전트를 연결하기 위해서는 포트를 `-p 50000:50000` 에 맵핑해야 한다. `50000` 포트는 에이전트를 컨트롤러에 연결하기 위해 사용된다.

## Reference

* https://github.com/jenkinsci/docker/blob/master/README.md
* https://medium.com/gdgsrilanka/running-jenkins-on-docker-for-a-newbie-855ad376500b

