## **1. 컨테이너 이미지 관리 명령어**

**컨테이너 이미지 실행**

1. 웹서버 컨테이너인 nginx 컨테이너 이미지를 다운로드 받아 간단하게 실행해보자. 도커 허브에서 이미지 검색
    
    ```
    docker search nginx
    ```
    
2. nginx 웹서버 컨테이너 이미지 다운로드. 이미지를 내려받을 때 사용하는 태그, 레이어, 이미지 고유 식별 값을 확인
    
    ```
    docker pull nginx
    ```
    
3. 다운로드 받은 이미지 확인
    
    ```
    docker images
    ```
    
4. busybox 컨테이너 이미지를 다운로드 받고, 다운로드 받은 이미지를 확인
    
    ```
    docker pull busybox
    docker images
    ```
    
5. 이미지 생성 과정에 대한 자세한 이력 보기 docker history : 이미지의 생성 이력을 확인하는 데 사용. 
이미지가 어떻게 만들어졌는지, Dockerfile의 어떤 명령들이 사용되었는지, 그리고 각 명령이 어떤 레이어를 추가했는지 확인 가능
    
    ```
    docker history nginx
    ```
    
    docker inspect : Docker 이미지나 컨테이너의 세부 정보를 JSON 형식으로 출력.이미지의 메타데이터, 환경 변수, 네트워크 설정, 파일 경로, 레이어 정보 등 다양한 정보를 포함
    
    ```
    docker inspect nginx
    docker inspect web-nginx | grep IPAddress
    ```
    
6. busybox 컨테이너 이미지 삭제하기
    
    ```
    web-nginx 컨테이너 삭제
    docker rm -f web-nginx
    docker ps
    ```
    
    nginx 컨테이너 이미지 삭제
    
    ```
    docker rmi nginx
    docker iamges
    ```
    

---

## **2. 컨테이너 관리 명령어**

**컨테이너 이미지 다운로드**

1. 웹 서버 컨테이너인 nginx 컨테이너 이미지를 다시 다운로드 받아 이번에는 컨테이너로 간단하게 실행해보자.
    
    ```
    docker pull nginx
    ```
    
2. 다운로드 받은 이미지 확인
    
    ```
    docker images
    ```
    

**컨테이너 실행**

1. 다운로드 받은 nginx 컨테이너를 web-nginx라는 이름으로 실행해보자.
    
    ```
    docker run -d --name web-nginx nginx:latest
    ```
    
    - d: Detached mode로 컨테이너를 실행 --name: 컨테이너의 이름을 지정. 이름을 지정하지 않으면 Docker가 임의의 이름을 생성 nginx:latest: 사용할 이미지와 태그를 지정
2. 동작중인 컨테이너 확인
    
    ```
    docker ps
    docker ps -a
    ```
    
3. 동작 중인 컨테이너의 IP Address 확인
    
    ```
    docker inspect web-nginx
    ```
    
4. 동작 중인 컨테이너에 접속 TEST
    
    ```
    curl 172.17.0.2
    ```
    
5. web-nginx 컨테이너를 중지시킨다.
    
    ```
    docker stop web-nginx
    docker ps
    docker ps -a
    ```
    
6. 컨테이너 삭제
    
    ```
    docker rm -f web-nginx
    ```
    
    무엇이 삭제되었나?
    
    ```
    docker ps -a
    docker images
    ```
    

---

## **3. 실습: apache 웹 서버 실행하기**

1. httpd라는 이름의 오피셜 컨테이너 이미지를 검색하여 STARS와 OFFICIAL 상태를 확인하시오.
    
    ```
    
    ```
    
2. httpd 이미지를 Docker Hub에서 로컬로 다운로드합니다.
    
    ```
    
    ```
    
3. 다운로드한 컨테이너 이미지를 확인하시오. 출력 결과에서 httpd 이미지의 full image ID와 size를 확인합니다.
    - full image id :
    - size:
    
    ```
    
    ```
    
4. 다운로드한 아파치 웹서버 컨테이너 이미지를 아래와 같은 명령으로 실행하시오.
 `docker run -d --name web -p 80:80 httpd:latest`
    
    ```
    
    ```
    
5. 현재 실행 중인 컨테이너 목록에서 web 컨테이너가 있는지 확인합니다.
    
    ```
    
    ```
    
6. web 컨테이너의 IP 주소를 확인합니다.
    
    ```
    
    ```
    
7. 컨테이너의 IP 주소로 접속하여 웹 페이지가 제대로 표시되는지 확인합니다.
    
    ```
    
    ```
    
8. docker rm 명령은 실행 중인 컨테이너는 바로 삭제할 수 없습니다. docker rm 명령의 적절한 옵션을 사용하여 강제로 종료 후 삭제하세요.
    
    ```
    
    ```
    

---

## **4. docker run 명령어 사용하기**

### **환경 변수 전달**

1. Hello World 메시지를 환경 변수로 설정하여 출력하기 alpine 이미지를 사용하여 GREETING이라는 환경 변수를 설정하고, 그 값을 컨테이너 내부에서 확인하기
    
    ```
    docker run --name greeting-container -e GREETING="Hello from Docker!" alpine /bin/sh -c 'echo $GREETING'
    ```
    
    - -name greeting-container : 컨테이너 이름 설정.
    - -e GREETING="Hello from Docker!" : GREETING 환경 변수 정의. Value를 Hello from Docker!로 지정.
    - alpine : Alpine Linux 이미지를 사용.
    - /bin/sh -c 'echo $GREETING' : 컨테이너 내에서 echo $GREETING 명령을 실행하여 GREETING 변수의 값을 출력
    
    ```
    docker rm greeting-container
    ```
    

### **Background or Foreground 서비스 동작**

1. background mode로 nginx 웹 서버 동작 시키기
    
    ```
    docker run -d --name web nginx:1.14
    
    docker ps
    curl <Container-IP>
    ```
    
2. foreground mode로 ubuntu 20.04 운영하기 ubuntu 이미지 기반 컨테이너를 터미널 모드에서 실행하여, 컨테이너 내에서 인터랙티브하게 작업하기
    
    ```
    docker run -it --name devel ubuntu:20.04
    root@00000000:/# cat /etc/os-release
    root@00000000:/# hostname
    
    root@00000000:/# cat /etc/hosts
    root@00000000:/# ps -ef
    root@00000000:/# exit
    exit
    ```
    
3. 동작 중인 컨테이너 모두 종료 시키기
    
    ```
    docker ps -a
    docker rm  -f web devel
    ```
    

---

## **5. 컨테이너 조작 명령어**

### **컨테이너 프로세스 실행 : docker exec**

1. 컨테이너 실행
    
    ```
    docker run -d --name web nginx:1.14
    docker ps
    ```
    
2. running 중인 컨테이너에서 새로운 프로세스를 실행시킬 때에는 docker exec 명령을 실행한다.
웹서버처럼 백그라운드에서 실행하는 컨테이너는 docker attach 명령으로 접속할 수 없다. 
이때 컨테이너 내부에 shell을 포함하고 있다면 exec 명령으로 터미널을 열어 접속할 수 있다.
    
    ```
    docker exec -it web /bin/sh
    	/# cat /etc/hosts
    	/# exit
    
    curl 172.17.0.X
    ```
    

### **컨테이너 프로세스 정보보기 : docker top**

1. running 중인 컨테이너 내부에서 실행중인 프로세스를 확인할 때에는 docker top 명령을 실행한다.
    
    ```
    docker top web
    ```
    
2. 컨테이너의 표준 출력 결과 보기: docker logs detached mode로 동작중인 컨테이너에서 출력된 STDOUT의 결과를 컨테이너 외부에서 모니터링할 수 있다. 보통의 경우 로그 정보를 확인할 때 사용한다.
    
    ```
    docker logs web
    docker logs -f web
    docker logs --tail 3  web
    ```
    

---

## **6. 실습: 컨테이너 조작 실습**

1. ubuntu 이미지를 다운 받아 컨테이너로 실행한다. 
    - -i : interactive mode로 STDIN을 오픈. command line interactive 지원 시 필수
    - -t : tty. pseudo-tty를 할당
    
    ```
    docker run -it ubuntu  /bin/bash
    root@00000000:/# hostname
    root@00000000:/# cat /etc/hosts
    root@00000000:/# cat /etc/resolv.conf
    ```
    
2. 컨테이너 내부 탐색 Process Namespace 격리 확인
    
    ```
    root@00000000:/# ps
    ```
    
    호스트 이름은 컨테이너 ID가 된다.
    
    ```
    root@00000000:/# hostname
    ```
    
    /etc/hosts 파일을 보면 호스트 엔트리도 추가되었다.
    
    ```hcl
    root@00000000:/# cat /etc/hosts
    ```
    
    컨테이너 운영체제 정보 확인
    
    ```
    root@00000000:/# cat /etc/os-release
    ```
    
    설치 패키지 업데이트
    
    ```
    root@00000000:/# apt-get update
    root@00000000:/# apt-get install net-tools
    root@00000000:/# ifconfig
    
    root@00000000:/# apt-get install iproute2 -y
    root@00000000:/# ip addr
    ```
    
    컨테이너에서 exit : 컨테이너 중지된다. 왜?
    
    ```
    root@00000000:/# exit
    ```
    
3. docker 호스트에서 컨테이너 상태 모니터링 컨테이너 상태 확인
    
    ```
    docker ps
    docker ps -a
    ```
    
    가장 최근에 작업했던 컨테이너 만 확인
    
    ```
    docker ps -l
    ```
    
4. 컨테이너에서 변경된 파일 정보 확인 
    
    `docker diff ID or docker diff NAME`
    
    ```
    docker diff 00000000
    ```
    
    컨테이너 내부에서 있었던 작업 내용을 출력
    
    `docker logs ID or docker logs NAME`
    
    ```
    docker logs 00000000
    ```
    
5. 컨테이너 네이밍
    - 이름 없이 컨테이너 생성하면 임시로 16HEX값으로 이름을 생성한다.
    - 이름규칙: 영숫자, _, 마침표,
    - 이름은 컨테이너와 애플리케이션 사이의 논리적 연결을 인식하고 구축하는데 유용.
    - 의미가 있는 이름을 사용하는 것이 좋다(web, db 등).
    - 동일 이름의 컨테이너 생성 불가.
    
    ```
    docker run  --name c1 -it ubuntu /bin/bash
    root@00000000:/# exit
    ```
    
    중지된 컨테이너 시작
    
    ```
    docker start c1
    ```
    
    실행 중인 컨테이너에 접속하기
    
    ```
    docker attach c1
    root@00000000:/# exit
    ```
    
6. hub.docker.com에서 registry 검색 후 image 실행
    
    ```
    docker search apache
    ```
    
    컨테이너 실행 시 백그라운드 모드(-d)로 동작. 데몬 컨테이너 실행 시 적용.
    
    ```
    docker run -d --name web httpd:2.4
    docker ps
    ```
    
    컨테이너 IP 확인
    
    ```
    docker inspect web
    curl http://172.17.0.X
    ```
    
    컨테이너 내부에서 실행 중인 프로세스 확인
    
    ```
    docker top web
    ```
    
7. 백그라운드에서 실행되는 컨테이너 STDOUT 확인
    
    ```
    docker run --name test -d ubuntu /bin/sh -c "while true; do date; sleep 3; done"
    ```
    
    logs : 컨테이너의 로그를 가져온다.
    
    ```
    docker logs test
    docker logs test -f		## 실시간조회
    docker logs --tail 10  test	## 마지막 10개 라인만 출력
    ```
    
8. 호스트에서 컨테이너 중지 이름으로 컨테이너 중지
    
    ```
    docker stop web
    ```
    
    ID로 중지
    
    ```
    docker stop 00000000
    ```
    
9. 컨테이너의 자세한 정보 얻기 : inspect
    
    ```
    docker inspect test
    ```
    
    daemon_test 컨테이너의 inspect 항목 중 일부분만 출력
    
    ```
    docker inspect --format='{{ .State.Status }}' test
    docker inspect --format='{{ .NetworkSettings.IPAddress }}' test
    ```
    
    홈 디렉토리의 .bashrc 파일은 로그인할 때 자동으로 명령어를 실행하는 startup script
    
     이 파일에 cip, crm 명령을 등록해서 이후 로그인했을 때에도 자유롭게 사용할 수 있도록 설정
    
    ```hcl
    vim ~/.bashrc
    ...
    alias cip="docker inspect --format='{{ .NetworkSettings.IPAddress }}'"
    alias crm='docker rm -f $(docker ps -aq)'
    
    source ~/.bashrc
    alias
    cip test
    ```
