## 1. Docker Hub 이미지 업로드 및 실행

- Dockerfile을 이용해 컨테이너 이미지를 생성
- 생성한 이미지를 Docker Hub에 업로드
- 파트너 이미지를 다운로드하여 컨테이너로 실행
    
    <img width="1838" height="791" alt="Image" src="https://github.com/user-attachments/assets/05280b65-84ba-4f8c-b4ea-84bd20f3b827" />
    

- 실습 디렉토리 생성
    
    ```bash
    mkdir -p ~/registry-test/dockerhub-upload
    cd ~/registry-test/dockerhub-upload
    ```
    
- 웹 페이지 파일 생성 : index.html
    
    ```bash
    cat << 'EOF' > index.html
    <h1>Docker Hub Upload Test</h1>
    <p>This container image was uploaded to Docker Hub.</p>
    <p>Verified by Docker Registry Lab</p>
    EOF
    ```
    
- Dockerfile 생성
    
    ```bash
    cat << 'EOF' > Dockerfile
    FROM nginx:latest
    
    COPY index.html /usr/share/nginx/html/index.html
    
    EXPOSE 80
    EOF
    ```
    
- 컨테이너 이미지 빌드
    
    ```bash
    docker build -t web:v1 .
    
    # 이미지 확인
    docker images
    ```
    
- 컨테이너 실행 테스트
    
    ```bash
    docker run -d --name web-test -p 8080:80 web:v1
    ```
    
    - 실행 확인
        
        ```bash
        curl http://localhost:8080
        ```
        
    - 브라우저에서 확인 : http://서버IP:8080
        
        <img width="800" height="391" alt="Image" src="https://github.com/user-attachments/assets/0e33ee5f-5057-4c3d-b61a-298dafc8657c" />
        
    - 테스트 후 컨테이너 삭제
        
        ```bash
        docker rm -f web-test
        ```
        
- Docker Hub 로그인
    
    ```bash
    #docker login -u smlinux
    docker login -u [DockerHub계정명]
    ```
    
- Docker Hub 업로드용 태그 지정
    - Docker Hub에 이미지를 업로드하려면 이미지 이름 앞에 Docker Hub 계정명을 붙여야 한다.
    
    ```bash
    # docker tag web:v1 smlinux/web:v1
    docker tag web:v1 [DockerHub계정명]/web:v1
    ```
    
    ```bash
    docker images
    ```
    
- Docker Hub로 이미지 업로드
    
    ```bash
    # docker push smlinux/web:v1
    docker push [DockerHub계정명]/web:v1
    ```
    
    - 업로드가 완료되면 Docker Hub 웹 사이트에서 Repository를 확인한다.
    
- 로컬 이미지 삭제
    - 업로드한 이미지를 받아 실행해보기 위해 로컬 이미지를 삭제
    
    ```bash
    docker rmi web:v1
    
    # docker rmi [DockerHub계정명]/web:v1
    docker rmi smlinux/web:v1
    ```
    
    - 이미지 삭제 확인
    
    ```bash
    docker images
    ```
    
- Docker Hub에서 파트너 이미지 다운로드
    
    ```bash
    # docker pull smlinux/web:v1
    docker pull [DockerHub 파트너 계정명]/web:v1
    ```
    
- Pull 받은 이미지로 컨테이너 실행
    
    ```bash
    # docker run -d --name web -p 8080:80 smlinux/web:v1
    docker run -d --name web -p 8080:80 [DockerHub 파트너 계정명]/web:v1
    ```
    
- 결과 확인
    
    ```bash
    curl http://localhost:8080
    ```
    
    - 브라우저에서 확인 : http://서버IP:8080

- 실습 리소스 정리
    
    ```bash
    # 컨테이너 삭제
    docker rm -f web
    
    # 이미지 삭제
    # docker rmi smlinux/web:v1
    docker rmi [DockerHub파트너계정명]/web:v1
    ```
    
<br>

## 2. Quiz. Node.js 앱 이미지 업로드 및 파트너 이미지 실행

### Quiz 시나리오

Node.js 웹 애플리케이션 이미지를 만든다.

- 이미지 이름: `node-app:v1`
- Docker Hub 업로드 이름: `[DockerHub계정]/node-app:v1`
- 파트너 이미지: `[파트너DockerHub계정]/node-app:v1`
- 컨테이너 내부 포트: `8080`
- 본인 테스트 포트: `8081`
- 파트너 이미지 테스트 포트: `8082`

### 실습

- 실습 디렉토리 생성
    
    ```bash
    mkdir -p ~/registry-test/node-quiz
    cd ~/registry-test/node-quiz
    ```
    
- Node.js 앱 소스 작성
    
    ```bash
    cat << 'EOF' > app.js
    const http = require('http');
    const os = require('os');
    
    const PORT = 8080;
    
    const server = http.createServer((req, res) => {
      res.writeHead(200, {'Content-Type': 'text/html; charset=utf-8'});
    
      res.end(`
      <!DOCTYPE html>
      <html>
      <head>
        <meta charset="UTF-8">
        <title>Node.js Docker Quiz</title>
    
        <style>
          body {
            margin: 0;
            padding: 0;
            font-family: 'Pretendard', sans-serif;
            background: linear-gradient(135deg, #0f172a, #1e293b);
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
          }
    
          .card {
            width: 700px;
            background: rgba(255,255,255,0.08);
            backdrop-filter: blur(10px);
            border-radius: 24px;
            padding: 50px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.4);
            text-align: center;
          }
    
          h1 {
            font-size: 52px;
            margin-bottom: 10px;
          }
    
          .subtitle {
            font-size: 22px;
            color: #cbd5e1;
            margin-bottom: 35px;
          }
    
          .highlight {
            color: #38bdf8;
            font-weight: bold;
          }
    
          .info-box {
            background: rgba(255,255,255,0.12);
            border-radius: 16px;
            padding: 20px;
            margin-top: 25px;
          }
    
          .hostname {
            font-size: 28px;
            font-weight: bold;
            color: #facc15;
          }
    
          .footer {
            margin-top: 35px;
            color: #94a3b8;
            font-size: 15px;
          }
    
          .emoji {
            font-size: 64px;
            margin-bottom: 20px;
          }
    
          .badge {
            display: inline-block;
            margin-top: 15px;
            padding: 10px 18px;
            border-radius: 999px;
            background: #2563eb;
            font-size: 14px;
            font-weight: bold;
          }
        </style>
      </head>
    
      <body>
    
        <div class="card">
    
          <div class="emoji">🚀🐳</div>
    
          <h1>Node.js Docker App</h1>
    
          <div class="subtitle">
            Built with <span class="highlight">Docker</span> + 
            <span class="highlight">Node.js</span>
          </div>
    
          <div class="info-box">
            <p>Container Hostname</p>
            <div class="hostname">${os.hostname()}</div>
          </div>
    
          <div class="badge">
            Docker Hub Upload Quiz
          </div>
    
          <div class="footer">
            Made by Cloud & DevOps Student ☁️
          </div>
    
        </div>
    
      </body>
      </html>
      `);
    });
    
    server.listen(PORT, () => {
      console.log(`Server running on port ${PORT}`);
    });
    EOF
    ```
    
- Dockerfile 작성
    - Base Image: `node:20-alpine`
    - 작업 디렉토리: `/app`
    - `app.js` 복사
    - 컨테이너 포트: `8080`
    - 실행 명령어: `node app.js`
    
    ```bash
    cat << 'EOF' > Dockerfile
    FROM node:20-alpine
    
    WORKDIR /app
    
    COPY app.js .
    
    EXPOSE 8080
    
    CMD ["node", "app.js"]
    EOF
    ```
    
- 이미지 빌드
    
    ```bash
    docker build -t node-app:v1 .
    
    # 이미지 확인
    docker images
    ```
    
- 로컬 실행 테스트
    
    ```bash
    docker run -d --name my-node -p 8081:8080 node-app:v1
    
    # 결과 확인
    curl http://localhost:8081
    ```
    
    <img width="1276" height="1399" alt="Image" src="https://github.com/user-attachments/assets/ec3c6ec6-0418-497e-aefc-bdcec70e5007" />
    
    ```bash
    # 테스트 후 컨테이너 삭제
    docker rm -f my-node-quiz
    ```
    
- smDocker Hub 로그인
    
    ```bash
    docker login
    ```
    
- Docker Hub 업로드 용 태그 지정
    
    ```bash
    docker tag node-app:v1 [본인DockerHub계정]/node-app:v1
    ```
    
- Docker Hub에 이미지 업로드
    
    ```bash
    docker push [본인DockerHub계정]/node-app:v1
    ```
    
- 파트너와 이미지 주소 교환 후 파트너 이미지 다운로드, 실행
    
    ```bash
    docker pull [파트너DockerHub계정]/node-app:v1
    
    # 이미지 실행
    docker run -d --name node-quiz -p 8082:8080 [파트너DockerHub계정]/node-app:v1
    
    #파트너 앱 결과 확인
    curl http://localhost:8082
    ```
    
    - 브라우저 확인: http://서버IP:8082
- 실습 리소스 정리
    
    ```bash
    docker rm -f node-quiz
    docker rmi [파트너DockerHub계정]/node-app:v1
    ```
    
<br>

## 3. Rocky Linux 9 환경에서 **Harbor**를 설치

### **준비 사항**

- **Server 1 (Harbor)**: IP `192.168.100.10`
- **Server 2 (Client)**: IP `192.168.100.20`

### Harbor Repository 서버 구성 (Server 1: 192.168.100.10)

- Server 1  필수 도구 설치 (Docker Compose)
    - 동작중인 모든 컨테이너 서비스 중지
        
        ```bash
        docker rm -f $(docker ps -aq)
        ```
        
    - Harbor는 여러 개의 컨테이너가 유기적으로 돌아가는 구조라 **Docker Compose**가 반드시 필요합니다.
        
        ```bash
        # Docker 공식 저장소 등록 및 설치
        #sudo dnf install -y yum-utils
        #sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
        #sudo dnf install -y docker-ce docker-ce-cli containerd.io
        
        # Docker 서비스 시작
        #sudo systemctl enable --now docker
        
        # Docker Compose V2 설치 확인 (최신 버전은 플러그인 형태)
        docker compose version
        ```
        
- Harbor 설치
    
    ```bash
    # 설치 패키지 다운로드 및 압축 해제
    wget https://github.com/goharbor/harbor/releases/download/v2.10.0/harbor-online-installer-v2.10.0.tgz
    tar xvzf harbor-online-installer-v2.10.0.tgz
    cd harbor
    ```
    
- Harbor 설정 (`harbor.yml`)
    - **hostname**: `192.168.100.10`으로 설정
    - **http**: `port 80` 확인
    - **https**: 실습을 위해 주석 처리 (`#`)
        
        ```bash
        # 1. 템플릿 파일 복사
        cp harbor.yml.tmpl harbor.yml
        
        # 2. 설정 파일 수정 (vi 편집기)
        vi harbor.yml
        ```
        
        ```bash
        # Configuration file of Harbor
        
        # The IP address or hostname to access admin UI and registry service.
        # DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
        hostname: 192.168.100.10
        
        # http related config
        http:
          # port for http, default is 80. If https enabled, this port will redirect to https port
          port: 80
        
        # https related config
        #https:
          # https port for harbor, default is 443
          #  port: 443
          # The path of cert and key files for nginx
          #  certificate: /your/certificate/path
          # private_key: /your/private/key/path
        ...
        harbor_admin_password: Harbor12345
        
        ```
        
    - **설치 실행**: `sudo ./install.sh`
        - 스크립트를 실행하여 필요한 이미지들을 내려받고 컨테이너를 구동합니다.
            
            ```bash
             sudo ./install.sh 
            ```
            
        - `docker ps` 명령어로 Nginx, Core, Jobservice 등 약 7~9개의 컨테이너가 돌아가는지 확인하세요.
            
            ```bash
            docker ps
            ```
            
- 방화벽 활성화
    - Rocky Linux는 기본적으로 방화벽이 매우 엄격합니다. Server 1에서 80번 포트가 열려 있는지 확인해야 합니다.
        
        ```bash
        # 80번 포트(http)가 허용되어 있는지 확인
        sudo firewall-cmd --list-services
        
        # 만약 http가 없다면 추가
        sudo firewall-cmd --permanent --add-service=http
        sudo firewall-cmd --reload
        ```
        
- Harbor 운영 및 웹 콘솔 정보
    - **URL**: `http://192.168.100.10` 접속
    - **Admin Account**: `admin` / `Harbor12345`
        
        <img width="1471" height="1311" alt="Image" src="https://github.com/user-attachments/assets/1d740c30-e9a7-40bc-82d6-74a9f34e8d8c" />
        
    - **[New Project]**: `polytech` (Public 프로젝트 생성)
        - Project : 도커 이미지들을 용도나 팀별로 묶어서 관리하는 가장 상위 개념의 격리된 저장 공간
        - Access Level - Public or Private 선택
        - Project quota limits :  프로젝트 할당 디스크 사용량. -1은 제한 없음.
        - Proxy Cache : 현재 서버를 외부 저장소(예:Docker hub)의 Proxy Cache 서버로 사용할지 선택
        
        <img width="557" height="385" alt="Image" src="https://github.com/user-attachments/assets/5d225025-19f6-4021-9c50-2b6602cd4be9" />
        

### Client 설정 (Server 2: 192.168.100.20)

- Docker 설치
    - Harbor는 여러 개의 컨테이너가 유기적으로 돌아가는 구조라 **Docker Compose**가 반드시 필요합니다.
        
        ```bash
        # Docker 공식 저장소 등록 및 설치
        sudo dnf install -y yum-utils
        sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
        sudo dnf install -y docker-ce docker-ce-cli containerd.io
        
        # Docker 서비스 시작
        sudo systemctl enable --now docker
        
        # Docker Compose V2 설치 확인 (최신 버전은 플러그인 형태)
        docker compose version
        
        # dokcer 관리자 등록
        sudo usermod -G docker user
        ```
        
- Docker 구성 및 Insecure Registry 설정
    - HTTP 저장소를 사용하기 위해 도커 데몬 설정.
    
    ```bash
    # 비보안 저장소 등록
    sudo -i
    cat << 'EOF' > /etc/docker/daemon.json
    {
      "insecure-registries": ["192.168.100.10", "192.168.100.10:80"]
    }
    EOF
    
    systemctl stop docker.socket
    systemctl start docker.socket
    exit
    
    # 확인
    docker info | grep -A 10 "Insecure Registries"
    ```
    
- Docker Login
    - `admin` / `Harbor12345`
    
    ```bash
    docker login 192.168.100.10:
    # Username/Password(Harbor12345) 입력 후 'Login Succeeded' 확인
    
    ```
    
    ```bash
    Username: admin
    Password: Harbor12345
    
    WARNING! Your credentials are stored unencrypted in '/root/.docker/config.json'.
    Configure a credential helper to remove this warning. See
    https://docs.docker.com/go/credential-store/
    
    Login Succeeded
    ```
    
- 컨테이너 Push & Pull 실습
    
    ```bash
    # 1. 이미지 태그 생성 (규칙: [RegistryIP]/[Project]/[Image]:[Tag])
    docker pull alpine:latest
    docker tag alpine:latest 192.168.100.10:80/polytech/alpine-image:v1
    docker images
    
    # 2. Push (업로드)
    docker push 192.168.100.10:80/polytech/alpine-image:v1
    
    # 3. Pull (삭제 후 다운로드)
    docker rmi 192.168.100.10:80/polytech/alpine-image:v1
    docker images
    
    docker pull 192.168.100.10:80/polytech/alpine-image:v1
    docker images
    
    ```
    
- 다운로드 받은 이미지 TEST
    
    ```bash
    docker run --name alpine -it 192.168.100.10:80/polytech/alpine-image:v1
    / # cat /etc/os-release
    / # exit
    
    ```
    
- 이미지 삭제
    
    ```bash
    # 컨테이너 삭제
    docker rm -f $(docker ps -aq)
    
    # 이미지 삭제
    docker rmi 192.168.100.10:80/polytech/alpine-image:v1
    ```
    
<br>

## 4. Docker Hub 이미지 업로드 및 실행

- Dockerfile을 이용해 컨테이너 이미지를 생성
- 생성한 이미지를 **Harbor**에 업로드(Push)
- 로컬 이미지를 삭제한 후, Harbor로부터 이미지를 다시 다운로드(Pull)하여 정상 동작하는지 검증
    
    <img width="1944" height="775" alt="Image" src="https://github.com/user-attachments/assets/310f9073-0199-46db-98c9-b832b40e340e" />
    
- 실습 디렉토리 생성 : server2
    
    ```bash
    # 실습을 위한 독립된 디렉토리 생성 및 이동
    mkdir -p ~/registry-test/harbor-upload
    cd ~/registry-test/harbor-upload
    ```
    
- 웹 페이지 파일 생성 : index.html
    
    ```bash
    cat << 'EOF' > index.html
    <h1>Harbor Private Registry Test</h1>
    <p>This container image was uploaded to our Private Harbor Server.</p>
    <p>Verified by Politech Infrastructure Lab</p>
    EOF
    ```
    
- Dockerfile 생성
    
    ```bash
    cat << 'EOF' > Dockerfile
    FROM nginx:latest
    
    COPY index.html /usr/share/nginx/html/index.html
    
    EXPOSE 80
    EOF
    ```
    
- 컨테이너 이미지 빌드
    
    ```bash
    docker build -t web:v1 .
    
    # 이미지 확인
    docker images | grep web
    ```
    
- 컨테이너 실행 테스트
    
    ```bash
    docker run -d --name web-test -p 8080:80 web:v1
    ```
    
    - 실행 확인
        
        ```bash
        curl http://localhost:8080
        ```
        
    - 브라우저에서 확인 : http://서버IP:8080
    - 테스트 후 컨테이너 삭제
        
        ```bash
        docker rm -f web-test
        ```
        
- Docker Hub 로그인
    
    ```bash
    #docker login -u smlinux
    #docker login -u [DockerHub계정명]
    
    docker login 192.168.100.10:80
    # Username: admin
    # Password: Harbor12345 입력 후 'Login Succeeded' 확인
    ```
    
- Docker Hub 업로드용 태그 지정
    - Harbor에 이미지를 올리려면 이미지 이름 앞에 `[HarborIP:포트]/[프로젝트명]/`이 반드시 접두사로 붙어야  한다.
    
    ```bash
    # 규칙: docker tag [원본이미지명:태그] [HarborIP:포트]/[프로젝트명]/[이미지명:태그]
    docker tag web:v1 192.168.100.10:80/polytech/web:v1
    ```
    
    ```bash
    # 태그 생성 확인
    docker images | grep polytech
    ```
    
- Harbor로 이미지 업로드
    
    ```bash
    docker push 192.168.100.10:80/polytech/web:v1
    ```
    
    
    - 업로드가 완료되면 Harbor 웹 콘솔(`http://192.168.100.10`)에 접속하여 `polytech` 프로젝트 안에 `web` 리포지토리가 생성되었는지 웹 화면으로 확인
        
        <img width="710" height="447" alt="Image" src="https://github.com/user-attachments/assets/7f1505fe-b1ba-4fc2-8e2b-520dde36e9ec" />

        
    
- 로컬 이미지 삭제 후 Harbor를 통해 다운로드 (Pull)
    - Harbor에서 이미지를 새로 내려받는 과정을 검증하기 위해, 존재하는 기존 이미지들을 모두 지웁니다.
    
    ```bash
    # 원본 이미지 및 태그 이미지 삭제
    docker rmi web:v1
    docker rmi 192.168.100.10:80/polytech/web:v1
    
    # 이미지 삭제 확인 (목록에 없어야 함)
    docker images
    ```
    
- Docker Hub에서 파트너 이미지 다운로드
    
    ```bash
    docker pull 192.168.100.10:80/polytech/web:v1
    
    # 이미지 다운로드 확인
    docker images
    ```
    
- 다운로드받은 이미지로 컨테이너 실행
    
    ```bash
    docker run -d --name web-harbor -p 8080:80 192.168.100.10:80/polytech/web:v1
    ```
    
- 결과 확인
    
    ```bash
    curl http://localhost:8080
    ```
    
    - 브라우저에서 확인 : http://서버IP:8080

- 실습 리소스 정리
    
    ```bash
    # 컨테이너 강제 삭제
    docker rm -f web-harbor
    
    # 로컬 이미지 삭제
    docker rmi 192.168.100.10:80/polytech/web:v1
    ```
