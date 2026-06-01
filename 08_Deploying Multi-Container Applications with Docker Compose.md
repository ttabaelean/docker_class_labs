## 1. Docker Compose로 WordPress(Web/App)와 MySQL(Database)연동 사용

### 💡 실습 목적

- 다중 컨테이너 환경의 대표적인 웹 서비스 구조인 WordPress(Web/App)와 MySQL(Database)을 정의합니다.
- 단 한 장의 설정 파일(`docker-compose.yml`)을 사용하여 다중 컨테이너 서비스 인프라를 일괄 구축하고 서비스를 연동합니다.
- 데이터 영속성 관리를 위한 **볼륨(Volume)** 옵션과 컨테이너 간 기동 순서를 조율하는 **의존성(`depends_on`)** 키워드를 실무 관점에서 학습합니다.

<br>

### 1.1 실습 디렉토리 생성 및 Workspace 이동

- Docker Compose 전용 실습 디렉토리 생성 및 이동
    
    ```bash
    mkdir -p my_wordpress
    cd my_wordpress
    ```
    
<br>


### 1.2 Docker Compose 파일 구성 (docker-compose.yml)

- `cat` 명령어를 사용하여 멀티 티어(Multi-Tier) 인프라 설계도 파일을 작성합니다.
    
    ```bash
    cat << 'EOF' > docker-compose.yml
    version: "3.9"
    
    services:
      db:
        image: mysql:5.7
        volumes:
          - db_data:/var/lib/mysql
        restart: always
        environment:
          MYSQL_ROOT_PASSWORD: somewordpress
          MYSQL_DATABASE: wordpress
          MYSQL_USER: wordpress
          MYSQL_PASSWORD: wordpress
    
      wordpress:
        depends_on:
          - db
        image: wordpress:latest
        volumes:
          - wordpress_data:/var/www/html
        ports:
          - "8000:80"
        restart: always
        environment:
          WORDPRESS_DB_HOST: db
          WORDPRESS_DB_USER: wordpress
          WORDPRESS_DB_PASSWORD: wordpress
          WORDPRESS_DB_NAME: wordpress
    
    volumes:
      db_data: {}
      wordpress_data: {}
    EOF
    ```
    
    - **services.db / services.wordpress**: 컴포즈 파일 내부에서 사용할 컨테이너 서비스 인프라의 식별자 이름입니다.
    - **volumes**: 컨테이너가 삭제되어도 데이터베이스 정보와 웹 콘텐츠 데이터가 삭제되지 않고 호스트 서버에 영구 보존되도록 도커 볼륨을 자동 매핑합니다.
    - **depends_on**: 서비스 간 선후 기동 관계를 지정합니다. `db` 인프라 컨테이너가 먼저 준비된 것을 인식한 뒤 `wordpress` 서비스가 부팅되도록 순서를 보장합니다.
    - **environment**: 서비스 연동에 필요한 컨테이너 내부 데이터베이스 환경 변수를 선언합니다.

<br>

### 1.3 Docker Compose 로 서비스 가동, 운영, 종료

- 멀티 컨테이너 환경을 백그라운드로 일괄 실행
- 인프라 서비스 일괄 생성 및 기동
    
    ```bash
    docker compose up -d
    ```
    
- 구동 상태 대시보드 및 포트 맵핑 모니터링 확인
    
    ```bash
    docker compose ps
    ```
    
- 웹 서비스 연동 및 결과 검증 : http://192.168.100.10:8000
    - 브라우저 창을 열고 Client 서버의 IP주소와 8000 포트로 접속하여 WordPress 초기 설치 및 세팅 화면이 정상 표출되는지 확인합니다.
    
   <img width="631" height="792" alt="Image" src="https://github.com/user-attachments/assets/68c08e42-bb6b-4872-8584-501d6b4a9ac2" />
    

- 현재 실행 중인 WordPress 웹 서버와 MySQL 데이터베이스가 주고받는 로그를 실시간으로 확인
    
    ```bash
    docker compose logs -f
    ```
    
- 컨테이너 내부 진입 및 명령어 실행 (exec)
    - `docker run`과 달리 컴포즈 환경에서는 가동 중인 서비스 이름을 지정하여 컨테이너 내부에 간편하게 진입할 수 있습니다. Nginx 기반의 워드프레스 파일 경로를 확인해 봅니다
    - wordpress 컨테이너 내부의 bash 셸로 진입
        
        ```bash
        docker compose exec wordpress bash
        ```
        
    - 워드프레스 소스 파일 위치 확인
        
        ```bash
        ls -l /var/www/html
        ```
        
    - 셸 빠져나오기
        
        ```bash
        exit
        ```
        
- db 서비스의 환경 변수 목록만 조회하기
    
    ```bash
    docker compose exec db env | grep MYSQL
    ```
    
- 전체 컨테이너 일시 정지
    
    ```bash
    docker compose ps
    ```
    
    - 외부 웹 브라우저(http://192.168.100.20:8000)에서 접속이 멈추는지 확인합니다.
    - 전체 컨테이너 동결 해제 및 서비스 재개
        
        ```bash
        docker compose unpause
        ```
        
- 인프라 환경 재기동 (restart)
    
    ```bash
    docker compose restart
    ```
    
- 정상 구동 상태 재검증
    
    ```bash
    docker compose ps
    ```
    
- 실습 인프라 리소스 일괄 정리
    - 컴포즈로 가동한 인프라 네트워크 환경과 컨테이너 인스턴스를 하나씩 지울 필요 없이 단 하나의 명령어로 깨끗하게 초기화합니다.
    - 실행 중인 웹 서비스 및 데이터베이스 인프라 완전 폐기
        
        ```bash
        docker compose down
        ```
        
    - (선택) 볼륨 데이터까지 완전히 깨끗하게 지우고 싶을 때 옵션
        
        ```bash
        docker compose down -v
        ```
        

<br>


# 2. QUIZ: Docker Compose 기반 보안 강화형 2-Tier Application 서비스 운영

### 💡 실습 목적

- Web Tier(Nginx)와 Application Tier(Node.js)가 분리된 2-Tier 웹 인프라를 정의합니다.
- 사용자의 일반 HTTP(80) 요청을 HTTPS(443) 보안 채널로 강제 전환하는 **SSL 리다이렉션**을 구현합니다.
- WAS(Node.js)의 외부 포트를 완전히 닫고 프록시 서버만 개방하여 내부망을 격리하는 **인프라 보안**을 컴포즈 설계도 한 장으로 구현합니다.
    
    <img width="1194" height="1317" alt="Image" src="https://github.com/user-attachments/assets/6562f65b-db55-4530-8042-889d7ae380d5" />
    

힌트:  dockercompose.yaml

```bash
cat << 'EOF' > docker-compose.yml
version: "3.9"

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress

volumes:
  db_data: {}
  wordpress_data: {}
EOF
```


