## 나를 소개하는 웹서버 운영하기

### 1. 웹서버 프로그램 선택

- 대표적인 웹서버 프로그램은 다음과 같다.
- **Apache (httpd)**
    - 가장 오래되고 널리 사용되는 웹서버
    - 설정이 쉽고 안정적
    - 전통적인 서버 환경에서 많이 사용
- **Nginx**
    - 가볍고 빠른 웹서버
    - 많은 접속자를 처리하는 데 강점
    - 최근 가장 많이 사용되는 웹서버
- **IIS (Internet Information Services)**
    - Microsoft에서 제공하는 웹서버
    - Windows 환경에서 사용
- **Node.js 기반 서버 (Express 등)**
    - 웹서버 기능을 코드로 직접 구현
    - 개발자들이 많이 사용

👉 이번 실습에서는 가볍고 빠르며 널리 사용되는 **Nginx 웹서버**를 사용한다.

### 2. 웹서버 프로그램 설치

- Rocky Linux 9 에서 Nginx 웹서버 설치하기
    
    ```bash
    sudo -i
    dnf -y install nginx
    
    ```
    
- Nginx  웹 서버 데몬 실행
    
    ```bash
    systemctl enable --now nginx
    systemctl status nginx
    ```
    

### 4.  웹 문서(index.html) 만들기

- 웹문서가 위치한 디렉토리로 이동
    
    ```bash
    cd /usr/share/nginx/html
    ls
    ```
    
- 나를 소개하는 웹문서 만들기
예 : 이모지 사용, 전체적으로 보기 좋게
    
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>My First Web Server</title>
    </head>
    <body>
        <h1>👋 Hello, I'm [Your Name]</h1>
        <p>Welcome to my server!</p>
    
        <h2>📌 About Me</h2>
        <ul>
            <li>이름: 홍길동</li>
            <li>취미: 게임 🎮</li>
            <li>목표: 클라우드 엔지니어</li>
        </ul>
    
        <h2>🔥 Message</h2>
        <p>이 서버는 내가 직접 만든 웹서버입니다 😎</p>
    </body>
    </html>
    ```
    
    - 이름:
    - 취미:
    - 좋아하는 음식:
    - 싫어하는 음식:
    - 올해 나의 목표:
    - 이 클래스에서 나는 000이고 싶다.
    
    ```bash
    cat << EOF > index.html
    
    <이곳에 여러분이 만든 나를 소개하는 웹문서 내용을 담으세요.>
    
    EOF
    ```
    

### 5. 웹페이지 확인

웹브라우저 : http://192.168.100.10
