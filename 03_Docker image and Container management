# 03. Docker Image and Container Management

---

# 1. 컨테이너 이미지 관리 명령어

## 1.1 컨테이너 이미지 다운로드

웹서버 컨테이너인 nginx 이미지를 검색한다.

```bash
docker search nginx

nginx 이미지를 다운로드한다.

docker pull nginx

다운로드한 이미지를 확인한다.

docker images

busybox 이미지를 다운로드하고 확인한다.

docker pull busybox
docker images

이미지 생성 이력을 확인한다.

docker history nginx

이미지 상세 정보를 확인한다.

docker inspect nginx
1.2 이미지 삭제

※ 컨테이너에서 사용 중인 이미지는 삭제할 수 없다.

docker rmi busybox
docker rmi nginx
2. 컨테이너 관리 명령어
2.1 컨테이너 이미지 다운로드
docker pull nginx
docker images
2.2 컨테이너 실행

nginx 컨테이너를 실행한다.

docker run -d --name web-nginx -p 80:80 nginx:latest

옵션 설명:

-d : Detached mode (백그라운드 실행)
--name : 컨테이너 이름 지정
-p 80:80 : 호스트 포트와 컨테이너 포트 연결

컨테이너 상태 확인

docker ps
docker ps -a

컨테이너 상세 정보 확인

docker inspect web-nginx

웹 접속 테스트

curl localhost

컨테이너 중지

docker stop web-nginx
docker ps
docker ps -a

컨테이너 삭제

docker rm -f web-nginx

삭제 확인

docker ps -a
docker images
3. 실습: Apache 웹 서버 실행하기
httpd 이미지를 검색하고 OFFICIAL 여부를 확인한다.
docker search httpd
httpd 이미지를 다운로드한다.
docker pull httpd
이미지 정보를 확인한다.
docker images
컨테이너 실행
docker run -d --name web -p 80:80 httpd:latest
실행 확인
docker ps
웹 접속 확인
curl localhost
컨테이너 삭제
docker rm -f web
4. docker run 명령어 활용
4.1 환경 변수 전달
docker run --name greeting-container -e GREETING="Hello from Docker!" alpine /bin/sh -c 'echo $GREETING'

컨테이너 삭제

docker rm greeting-container
4.2 Background / Foreground 실행
Background 실행
docker run -d --name web nginx:latest
docker ps
Foreground 실행
docker run -it --name devel ubuntu:22.04

컨테이너 내부 명령어

cat /etc/os-release
hostname
ps -ef
exit

※ 컨테이너는 PID 1 프로세스 종료 시 함께 종료됨

컨테이너 삭제

docker rm -f web devel
5. 컨테이너 조작 명령어
5.1 docker exec
docker run -d --name web nginx:latest
docker exec -it web /bin/sh
5.2 docker top
docker top web
5.3 docker logs
docker logs web
docker logs -f web
docker logs --tail 3 web
6. 실습: 컨테이너 내부 탐색
6.1 컨테이너 실행
docker run -it ubuntu /bin/bash

컨테이너 내부 명령어

hostname
cat /etc/hosts
cat /etc/resolv.conf
ps
cat /etc/os-release

패키지 설치

apt-get update
apt-get install net-tools -y
ifconfig

apt-get install iproute2 -y
ip addr

컨테이너 종료

exit
6.2 컨테이너 상태 확인
docker ps
docker ps -a
docker ps -l
6.3 변경사항 확인
docker diff <CONTAINER_ID>
docker logs <CONTAINER_ID>
6.4 컨테이너 네이밍
docker run --name c1 -it ubuntu /bin/bash
exit

컨테이너 시작

docker start c1

컨테이너 접속

docker attach c1
6.5 Apache 컨테이너 실행
docker search apache
docker run -d --name web httpd:latest
docker ps

접속 확인

curl localhost
6.6 로그 확인 실습
docker run --name test -d ubuntu /bin/sh -c "while true; do date; sleep 3; done"

로그 확인

docker logs test
docker logs -f test
docker logs --tail 10 test
6.7 컨테이너 중지
docker stop web
docker stop <CONTAINER_ID>
6.8 inspect 활용
docker inspect test
docker inspect --format='{{ .State.Status }}' test
docker inspect --format='{{ .NetworkSettings.IPAddress }}' test
6.9 alias 설정
vim ~/.bashrc
alias cip="docker inspect --format='{{ .NetworkSettings.IPAddress }}'"
alias crm='docker rm -f $(docker ps -aq)'
source ~/.bashrc

사용 예시

cip test

---
