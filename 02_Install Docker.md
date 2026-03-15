# 2. 도커 설치와 도커 관리자 생성하기

## 2.1 Install Docker (Rocky Linux 9)

### [**Set up the repository**](https://docs.docker.com/engine/install/rhel/#set-up-the-repository)

Install the **`dnf-plugins-core`** package (which provides the commands to manage your DNF repositories) and set up the repository.

```bash
 sudo dnf -y install dnf-plugins-core
 sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

### [**Install Docker Engine**](https://docs.docker.com/engine/install/rhel/#install-docker-engine)

1. Install the Docker packages. Latest Specific version
    
    To install the latest version, run:
    
    ```bash
    sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
    
2. Start Docker Engine.
    
    ```bash
    sudo systemctl enable --now docker
    ```
    
3. Verify
    
    ```bash
    docker version
    Client: Docker Engine - Community
     Version:           29.3.0
     API version:       1.54
     Go version:        go1.25.7
     Git commit:        5927d80
     Built:             Thu Mar  5 14:28:57 2026
     OS/Arch:           linux/amd64
     Context:           default
    permission denied while trying to connect to the docker API at unix:///var/run/docker.sock
    
    ```


## 2.2 Docker 관리자 권한 설정

## docker 그룹 확인 후 admin 사용자를 docker 그룹에 추가

- Docker 설치 시 자동으로 생성됩니다.
    
    ```bash
    getent group docker
    ```
    
- admin 사용자를 docker 그룹에 추가
    
    ```bash
    sudo usermod -aG docker admin
    ```
    
- 그룹 적용 확인
    
    ```bash
    id admin
    ```
    
- 로그아웃 후 재로그인
그룹 변경은 **로그인 세션을 다시 시작해야 적용**됩니다.
    
    ```bash
    newgrp docker
    ```
    

### Docker 명령 테스트

- docker version 정보보기
    
    ```bash
    docker version
    ```
