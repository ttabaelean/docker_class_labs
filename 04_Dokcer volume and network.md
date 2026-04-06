## 1. Docker Volume LAB: 웹서버 공유 디렉토리

- Docker Volume(`-v`)을 활용하여 **호스트 디렉토리를 컨테이너에 마운트**
- 동일한 데이터를 사용하는 **2개의 웹서버(web1, web2)** 구성
- 두 컨테이너가 **같은 웹페이지를 제공하는지 확인**
- 실습 내용
    - `/webdata` 디렉토리를 생성하고 HTML 파일 작성
    - Nginx 컨테이너 2개 실행
    - 동일한 볼륨을 마운트하여 데이터 공유 확인

---

**1. 공유 디렉토리 생성하기**

- 두 개의 웹 서버 컨테이너(web1, web2)가 같이 사용할 HTML 파일을 저장할 디렉토리를 생성한다.
    
    ```powershell
    sudo -i
    mkdir /webdata
    cd /webdata
    
    cat << EOF > /webdata/index.html
    내가 만든 웹페이지 html 코드
    EOF
    ```
    

1. **web1 웹 서버 컨테이너 실행**
    - web1 컨테이너를 실행하고 `/webdata` 디렉토리를 nginx 웹 디렉토리에 연결한다.
        
        ```powershell
        docker run -d \
        --name web1 \
        -p 8081:80 \
        -v /webdata:/usr/share/nginx/html \
        nginx
        ```
        
        웹브라우저를 실행해서 http://10.100.0.10:8081로 접속해봅니다.
        

1. **web2 웹서버 컨테이너 실행**
    - web2 컨테이너도 동일한 디렉토리를 사용하도록 실행한다.
        
        ```powershell
        docker run -d \
        --name web2 \
        -p 8082:80 \
        -v /webdata:/usr/share/nginx/html \
        nginx
        ```
        
        웹브라우저를 실행해서 http://10.100.0.10:8081로 접속해봅니다.
        
        동일한 웹페이지가 보이나요?
        
2. **파일 수정 후 동기화 확인**
    - 공유 디렉토리의 파일을 수정하면 두 컨테이너 모두 변경된 내용을 보여야 한다.
        
        ```powershell
        cat << EOF >> /webdata/index.html
         html 문서 수정해서 저장해보세요
        EOF
        ```
        
        브라우저 새로 고침 후 web1, web2 모두 변경 확인
        
3. **리소스 정리**
    - 동작중인 모든 컨테이너를 종료하고 삭제한 후, 공유 디렉토리도 삭제합니다.
    
    ```powershell
    docker ps
    docker stop web1 web2
    docker rm web1 web2
    
    rm -rf /webdata
    ```
    

## 실습 : 웹 로그 공유 디렉토리 구성하기

- Docker Volume(`v`)을 활용하여 **컨테이너 로그를 호스트 디렉토리에 저장**
- 웹 서버에서 발생하는 로그를 **외부에서 직접 확인**
- 컨테이너 삭제 후에도 로그가 **유지되는지 확인**
- 실습 내용
    - `/weblog` 디렉토리를 생성하여 로그 저장 위치 구성
    - Nginx 컨테이너 실행 후 로그 디렉토리 마운트
    - 웹 요청 발생 후 로그 생성 확인
    - 컨테이너 삭제 후에도 로그 유지 확인

---

1. 웹 서버 컨테이너의 로그를 저장할 공유 디렉토리를 생성하세요
    
    ```powershell
    
    ```
    
2. nginx 웹 서버 컨테이너를 실행하고, 로그 디렉토리를 호스트와 연결하세요
(힌트: nginx 로그 경로 `/var/log/nginx`, 포트: 8080)
    
    ```powershell
    
    ```
    
3.  웹 브라우저로 접속하여 로그가 생성되도록 웹페이지 문서 요청하요.
    
    • 접속 주소 : 10.100.0.10:포트번호
    
4. 호스트에서 로그 파일이 생성되었는지 확인하세요.
    
    ```powershell
    
    ```
    
5. 생성된 로그 파일의 내용을 확인하세요. (cat 파일이름)
    
    ```powershell
    
    ```
    
6. 컨테이너를 삭제한 후에도 로그 파일이 보존되는지 확인하세요.
    
    ```powershell
    
    ```
    
7. 모든 리소스 삭제
    
    ```powershell
    rm -rf /weblog
    ```
    

## 2-1. 기본 Bridge 네트워크 확인하기

- Docker 기본 bridge 네트워크의 동작 방식 이해
- 컨테이너 간 통신 시 **IP 기반 통신 구조 확인**
- 동일 네트워크에 있는 컨테이너 간 통신 테스트
- 실습 내용
    - Docker 네트워크 목록 및 bridge 네트워크 확인
    - nginx 컨테이너 2개 실행
    - 컨테이너 IP 주소 확인
    - IP 기반 ping 테스트 수행

---

1. 현재 Docker 네트워크 목록을 확인하세요.
    
    ```powershell
    docker network ls
    ```
    
2. 기본 bridge 네트워크의 상세 정보를 확인하세요.
    
    ```powershell
    docker network inspect bridge
    ```
    
3. nginx 컨테이너 2개를 실행하세요. (포트 다르게)

    
    ```powershell
    docker run -d --name net1 -p 8081:80 nginx
    docker run -d --name net2 -p 8082:80 nginx
    ```
    
4. 각 컨테이너의 IP 주소를 확인하세요.
    
    ```powershell
    docker inspect net1 | grep IPAddress
    docker inspect net2 | grep IPAddress
    ```
    
5. net1 컨테이너에서 net2 컨테이너로 ping 테스트를 수행하세요.
    
    ```powershell
    docker exec -it net1 ping <net2 IP>
    ```
    
6. 모든 리소스를 삭제하세요.
    
    ```powershell
    docker rm -f net1 net2
    ```
    

## 2-2. User Defined Network 구성하기

- 사용자 정의 네트워크 생성 및 구성 방법 이해
- 네트워크 생성 시 **IP 대역과 gateway 직접 설정**
- 컨테이너 이름 기반 통신(DNS) 확인
- 실습 내용
    - 사용자 정의 네트워크(test-net) 생성
    - subnet 및 gateway 설정 확인
    - 컨테이너를 네트워크에 연결
    - 컨테이너 이름으로 ping 테스트 수행

---

1. 현재 Docker 네트워크 목록을 확인하세요
    
    ```powershell
    docker network ls
    ```
    
2. 기본 bridge 네트워크의 상세 정보를 확인하세요.
    
    ```powershell
    docker network inspect bridge
    ```
    
3. bridge 타입의 사용자 정의 네트워크(test-net)를 생성하세요.
    - 네트워크 타입 : bridge
    - IP 대역 :192.168.10.0/24
    - 게이트웨이 :192.168.10.1
    
    ```
    docker network create \
    --driver bridge \
    --subnet 192.168.10.0/24 \
    --gateway 192.168.10.1 \
    test-net
    ```
    
4. 생성한 네트워크를 확인하세요.
    
    ```powershell
    docker network ls
    ```
    
5. 네트워크 상세 정보를 확인하여 subnet과 gateway가 정상 적용되었는지 확인하세요.
    
    ```powershell
    docker network inspect test-net
    ```
    
6. web1, web2 컨테이너를 생성하고 test-net 네트워크에 연결하세요.
    
    ```powershell
    docker run -d --name web1 --network test-net nginx
    docker run -d --name web2 --network test-net nginx
    ```
    
7. 각 컨테이너의 IP 주소를 확인하세요.
    
    ```powershell
    docker inspect web1 | grep IPAddress
    docker inspect web2 | grep IPAddress
    ```
    
8. web1 컨테이너에서 web2 컨테이너로 ping 테스트를 수행하세요.
    
    ```powershell
    docker exec -it web1 ping web2
    ```
    
    통신 결과 확인 : web1 → web2 ping : 정상 동작 확인
    
9.  모든 리소스 삭제
    
    ```powershell
    docker rm -f web1 web2
    docker network rm test-net
    ```
    

## 실습: User Defined Network + 포트 포워딩 구성하기

- 내부 네트워크 통신과 외부 접근 방식의 차이 이해
- 포트 포워딩(`p`)을 이용한 **외부 접근 구성**
- 하나의 컨테이너는 외부 접근 가능, 다른 컨테이너는 내부 전용으로 구성
- 실습 내용
    - 사용자 정의 네트워크 생성 (IP 대역 직접 설정)
    - web1 컨테이너에 포트 포워딩 설정
    - web2 컨테이너는 내부 네트워크 전용으로 구성
    - 내부 통신(ping)과 외부 접속 차이 확인

---

1. bridge 타입의 사용자 정의 네트워크(test-net)를 생성하세요.
    - 네트워크 타입 : bridge
    - IP 대역 : 10.10.0.0/24
    - 게이트웨이 : 10.10.0.1
    
    ```powershell
    
    ```
    
2.  생성한 네트워크의 상세 정보를 확인하세요. 생성된 subnet과 gateway 값 확인
    
    ```powershell
    
    ```
    
3. web1 컨테이너를 생성하고 test-net 네트워크에 연결하세요.
또한 외부에서 접속할 수 있도록 포트 포워딩을 설정하세요. (포트: 8080)
    
    ```powershell
    
    ```
    

1. web2 컨테이너를 생성하고 test-net 네트워크에 연결하세요.  (외부 포트는 연결하지 않음)
    
    ```
    
    ```
    
2. web1, web2 컨테이너의 IP 주소를 확인하세요. IP가 10.10.0.x 대역으로 할당되었는지 확인
    
    ```powershell
    
    ```
    
3. web1 컨테이너에서 web2 컨테이너로 ping 테스트를 수행하세요.
    
    ```powershell
    
    ```
    
4. 외부 접속 및 내부 통신을 확인하세요.
    - web1 접속 주소 : _______________________________
    - web2 외부 접속 가능 여부 : __________________
    - 이유 : ________________________________________

1. 모든 리소스를 삭제하세요.
    
    ```powershell
    
    ```
