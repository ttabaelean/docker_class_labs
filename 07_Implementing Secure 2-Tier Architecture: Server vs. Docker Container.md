## 2-tier Application 서비스 운영: Server 기반

- 가상머신 snapshot 구성
- 아키텍처
    
    <img width="1183" height="1330" alt="Image" src="https://github.com/user-attachments/assets/8ccca14d-0f3b-4e2c-b427-15c2e2c79434" />
    
    **① HTTP 요청 → HTTPS로 리다이렉트**
    
    - 사용자가 `http://ServerIP:80` 로 접속 → Nginx 가 이를 감지
    - 자동으로 `https://ServerIP:443` 로 리다이렉트 수행
    
    **② HTTPS 요청 → Web Tier에서 처리**
    
    - 사용자가 `https://ServerIP:443` 로 다시 요청
    - Web Tier (web-proxy 컨테이너)가 받아서 다음 수행
        - SSL 인증서 기반 **암호화 해제 (SSL Termination)**
        - 내부 App으로 요청 전달 (`proxy_pass`)
    
    **③ 내부망 통신 → App Tier(http://server2:8080)에서 요청 처리**
    
    - Web Tier로 부터 받은 요청 확인 후 처리
    - 결과를 다시 Web Tier로 전달

### 1. Application 서버 구성(sever2)- nodejs 앱

- Node.js 설치
    
    ```bash
    sudo -i
    
    # Node.js 설치 (기본 모듈 스트림 사용)
    dnf install -y nodejs
    
    # 설치 확인
    node -v
    npm -v
    
    exit
    ```
    
- source code : app.js
    
    ```bash
    mkdir ~/lab-app
    cd ~/lab-app
    
    # 애플리케이션 코드
    # 8080 포트를 열어서 외부 서비스 요청 들어오면 
    # "Response from App Server (Server 2)" 메세지 전달.
    cat << 'EOF' > app.js
    const http = require('http');
    http.createServer((req, res) => {
      res.writeHead(200, {'Content-Type': 'text/plain; charset=utf-8'});
      res.end("Response from App Server (Server 2)\n");
    }).listen(8080);
    console.log("App Server is running on port 8080...");
    EOF
    ```
    
- 애플리케이션 서비스 동작 및 연결 TEST
    
    ```bash
    # 앱 실행 (백그라운드)
    node app.js &
    
    # 연결 TEST
    netstat -napt | grep 8080
    
    curl localhost:8080
    ```
    
- 방화벽 설정
    
    ```bash
    sudo systemctl status firewalld
    
    # 방화벽 동작 중단
    sudo systemctl stop firewalld
    sudo systemctl disable firewalld
    
    sudo systemctl status firewalld
    
    # 방화벽 동작중 특정 포트만 오픈하기
    #sudo firewall-cmd --permanent --add-port=8080/tcp
    #sudo firewall-cmd --reload
    ```

<br>

### 2. Proxy 서버(server1) 실행 (Nginx + SSL) - Web tier

- `proxy_pass` 설정과 인증서 배치
    - `proxy_pass`는 사용자가 Nginx(프록시 서버)로 보낸 요청을 실제 서비스를 제공하는 다른 서버(Back-end)로 그대로 전달하는 역할을 합니다.
    - **보안 (Security)**: 앱이 어디에 있는지, 몇 번 포트를 쓰는지 외부에 노출하지 않음.
    - 인증 처리 (SSL Termination): 복잡한 SSL 암호화/복호화 작업을 Nginx가 전담하게 하여, 뒤에 있는 앱의 부담을 덜어줌
    - **부하 분산 (Load Balancing)**: 요청이 너무 많으면 여러 명의 요리사에게 나누어 전달할 수도 있음

<br>

- Nginx 설치
    
    ```bash
    sudo -i
    
    # Nginx 설치 및 실행
    dnf install -y nginx
    
    systemctl enable --now nginx
    ```
    
- 인증서 생성
    
    ```bash
    sudo mkdir -p /etc/nginx/ssl
    
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
      -keyout /etc/nginx/ssl/server.key -out /etc/nginx/ssl/server.crt \
      -subj "/C=KR/ST=Seoul/L=Seoul/O=Weplat/CN=192.168.100.10"
      
    ls /etc/nginx/ssl/ -l
    ```
    
- nginx 설정 파일(secure-proxy.conf) 생성. **HTTPS 및 Reverse Proxy 설정**
    
    ```bash
    cat << 'EOF' > /etc/nginx/conf.d/secure-proxy.conf
    server {
        listen 80;
        server_name 192.168.100.10;
        # 모든 HTTP 요청을 HTTPS로 리다이렉트
        return 301 https://$host$request_uri;
    }
    
    server {
        listen 443 ssl;
        server_name 192.168.100.10;
    
        # 인증서 경로 지정
        ssl_certificate /etc/nginx/ssl/server.crt;
        ssl_certificate_key /etc/nginx/ssl/server.key;
    
        location / {
            # /로 접속 요청 들어오면 Server 2의 8080 포트로 연결
            proxy_pass http://192.168.100.20:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
    EOF
    
    ```
    
- 방화벽에서 HTTP(80)와 HTTPS(443) 포트 오픈
    
    ```bash
    sudo systemctl status firewalld
    
    # 방화벽 동작중 특정 포트만 오픈하기
    firewall-cmd --permanent --add-service=http
    firewall-cmd --permanent --add-service=https
    firewall-cmd --reload
    
    firewall-cmd --list-services
    ```
    
- 웹서버 재시작
    
    ```bash
    systemctl restart nginx
    ```

<br>

### 3. 연결 TEST

- 리다이렉트 되는지 확인
    
    ```bash
    # -I 옵션: 헤더만 출력
    **curl -I http://192.168.100.10**
    ```
    
- **동작 확인 (HTTPS)**
    
    ```bash
    # -L 옵션: 리다이렉트 따라감
    # -I 옵션: 헤더만 출력
    # -k 옵션: 사설 인증서 경고를 무시합니다.
    curl -LkI http://192.168.100.10
    ```
    
- 외부 연결 확인 : https://192.168.100.10
    
    
    <img width="404" height="129" alt="Image" src="https://github.com/user-attachments/assets/d690ca78-7070-46f7-8e9e-a4a3a0c5da54" />
    
- snapshot 되돌리기

---
<br>

## 2-tier Application 서비스 운영: Container 기반

- 아키텍처
    
    <img width="1194" height="1317" alt="Image" src="https://github.com/user-attachments/assets/7ab3be80-389e-40ce-ab28-3173dac06c51" />
    
    **① HTTP 요청 → HTTPS로 리다이렉트**
    
    - 사용자가 `http://ServerIP:80` 로 접속 → Nginx 가 이를 감지
    - 자동으로 `https://ServerIP:443` 로 리다이렉트 수행
    
    **② HTTPS 요청 → Web Tier에서 처리**
    
    - 사용자가 `https://ServerIP:443` 로 다시 요청
    - Web Tier (web-proxy 컨테이너)가 받아서 다음 수행
        - SSL 인증서 기반 **암호화 해제 (SSL Termination)**
        - 내부 App으로 요청 전달 (`proxy_pass`)
    
    **③ 내부망 통신 → App Tier에서 요청 처리**
    
    - Web Tier로 부터 받은 요청 확인 후 처리
    - 결과를 다시 Web Tier로 전달

<br>

### **1. 도커 네트워크 생성**

- 컨테이너들이 서로 이름으로 통신할 수 있는 가상 네트워크를 먼저 만듭니다.

```bash
mkdir ~/build-test/app
cd ~/build-test/app

docker network create app-net
docker network ls
```

<br>

### 2. App 컨테이너 빌드 및 실행 (Node.js) - Application tier

- source code : app.js
    
    ```bash
    const http = require('http');
    http.createServer((req, res) => {
      res.writeHead(200);
    	  res.ecat appnd("Build Comparison Test\n");
    }).listen(8080);
    console.log("Server running on port 8080");
    
    ```
    
- **Container build**
    - **Dockerfile 생성: Dockerfile.app**
        - **Multi-stage Build**와 **Non-root User** 원칙을 적용
        
        ```bash
        # Stage 1: Build
        FROM node:20-alpine AS builder
        WORKDIR /app
        COPY app.js .
        
        # Stage 2: Run
        FROM node:20-alpine
        WORKDIR /app
        COPY --from=builder /app/app.js .
        USER node
        EXPOSE 8080
        CMD ["node", "app.js"]
        ```
        
    - container build
        
        ```bash
        # 이미지 빌드
        docker build -t node-app:v1 -f Dockerfile.app .
        docker images node-app:v1
        ```
        
    - container 실행 후 TEST
        
        ```bash
        docker run -d --name my-app --network app-net -p 8080:8080 node-app:v1
        curl localhost:8080
        
        docker rm -f my-app
        ```
        
    - 안전한 구성으로 동작. 외부 포트 개방 안 함 - 보안 강화
        
        ```bash
        docker run -d --name my-app --network app-net node-app:v1
        docker ps
        
        # 접속 될까요?
        curl localhost:8080
        
        # 어떻게 하면 접속 될까요?
        
        ```
        
<br>

### 3. Proxy 컨테이너 빌드 및 실행 (Nginx + SSL) - Web tier

- `proxy_pass` 설정과 인증서 배치
    - `proxy_pass`는 사용자가 Nginx(프록시 서버)로 보낸 요청을 실제 서비스를 제공하는 다른 서버(Back-end)로 그대로 전달하는 역할을 합니다.
    - **보안 (Security)**: 앱이 어디에 있는지, 몇 번 포트를 쓰는지 외부에 노출하지 않음.
    - 인증 처리 (SSL Termination): 복잡한 SSL 암호화/복호화 작업을 Nginx가 전담하게 하여, 뒤에 있는 앱의 부담을 덜어줌
    - **부하 분산 (Load Balancing)**: 요청이 너무 많으면 여러 명의 요리사에게 나누어 전달할 수도 있음
- 인증서 생성
    
    ```bash
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
      -keyout server.key -out server.crt \
      -subj "/C=KR/ST=Seoul/L=Seoul/O=MyCOMPANY/CN=localhost"
    ```
    
- Nginx 설정 파일(default.conf ) 생성
    
    ```bash
    server {
        listen 80;
        return 301 https://$host$request_uri; # HTTP 접속 시 HTTPS로 강제 리다이렉트
    }
    
    server {
        listen 443 ssl;
        server_name localhost;
    
        ssl_certificate /etc/nginx/ssl/server.crt;
        ssl_certificate_key /etc/nginx/ssl/server.key;
    
        location / {
            # 컨테이너 이름을 도메인처럼 사용합니다.
            proxy_pass http://my-app:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
    ```
    
- Dockerfile 생성(Dockerfile.proxy)
    
    ```bash
    FROM nginx:alpine
    
    # 설정 파일 및 인증서 복사
    RUN mkdir -p /etc/nginx/ssl
    COPY default.conf /etc/nginx/conf.d/default.conf
    COPY server.crt /etc/nginx/ssl/server.crt
    COPY server.key /etc/nginx/ssl/server.key
    
    EXPOSE 80 443
    ```
    
- 이미지 빌드
    
    ```bash
    docker build -t web-proxy:v1 -f Dockerfile.proxy .
    docker images web-proxy:v1
    ```
    
- 컨테이너 실행. 외부 443 포트 개방
    
    ```bash
    docker run -d --name web-proxy --network app-net -p 80:80 -p 443:443 web-proxy:v1
    
    docker ps
    ```
    
<br>

### 4. 연결 TEST

- **동작 확인 (HTTPS)**
    
    ```bash
    curl -k https://localhost
    # "Build Comparison Test" 응답이 오면 성공!
    ```
    
- **HTTP -> HTTPS 리다이렉션 확인**
    
    ```bash
    curl -I http://localhost
    # "HTTP/1.1 301 Moved Permanently"와 함께 Location: https://... 가 보이면 성공!
    ```
    
- **네트워크 격리 확인. 8080으로 외부 접속 금지됨**
    
    ```bash
    curl localhost:8080
    # "Failed to connect"가 나와야 함 (이것이 의도된 보안 격리임을 강조!)
    ```
    
- 외부 연결 확인
    
    
    <img width="832" height="558" alt="Image" src="https://github.com/user-attachments/assets/9d8e2861-8c79-493b-a8fb-6d6a20d94833" />
    
    <img width="751" height="574" alt="Image" src="https://github.com/user-attachments/assets/2893efdd-3d7a-46e4-81dc-b4b1efcad16a" />
    
    https://192.168.100.10:8080/ 접속 가능할까?
