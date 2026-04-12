# Python-Docker-Web-Game-Deployment

본 실습에서는 내가 만든 파이썬 코드를 도커 컨테이너로 복사하여 웹 서비스를 실행하고, 코드를 수정하여 서비스에 즉시 반영하는 배포(Deployment) 과정을 학습합니다.

<br>

### 1단계: 실습 환경 준비 (호스트 OS)
작업할 디렉토리를 만들고 이동합니다.

```bash
# 1. 작업 디렉토리 생성
mkdir ~/weplat-lab 
cd ~/weplat-lab
```

<br>


파이썬 게임 코드 파일 생성

```bash
cat <<EOF > box_game_py.py
from http.server import BaseHTTPRequestHandler, HTTPServer

# [Lab 포인트] 이 변수들을 수정해서 게임의 난이도를 조절하세요!
GAME_TITLE = "Weplat Python Game Server"
BOX_SPEED = 5      # 박스 이동 속도
SPAWN_RATE = 800   # 박스 생성 주기(ms)

GAME_HTML = f"""
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>{GAME_TITLE}</title>
    <style>
        body {{ background: #222; color: white; text-align: center; font-family: sans-serif; }}
        #game {{ width: 400px; height: 500px; border: 2px solid white; margin: 20px auto; position: relative; overflow: hidden; }}
        #player {{ width: 30px; height: 30px; background: #4caf50; position: absolute; bottom: 10px; left: 185px; }}
        .box {{ width: 40px; height: 40px; background: #f44336; position: absolute; }}
    </style>
</head>
<body>
    <h1>{GAME_TITLE}</h1>
    <p>파이썬 변수 조절 실습 (속도: {BOX_SPEED}, 주기: {SPAWN_RATE}ms)</p>
    <div id="game"><div id="player"></div></div>
    
    <script>
        const speed = {BOX_SPEED};
        const spawnRate = {SPAWN_RATE};
        const game = document.getElementById("game");
        const player = document.getElementById("player");
        let px = 185;
        let gameOver = false;

        document.addEventListener("keydown", (e) => {{
            if(gameOver) return;
            if(e.key === "ArrowLeft" && px > 0) px -= 20;
            if(e.key === "ArrowRight" && px < 370) px += 20;
            player.style.left = px + "px";
        }});

        function createBox() {{
            if(gameOver) return;
            const b = document.createElement("div");
            b.className = "box";
            b.style.left = Math.random() * 360 + "px";
            game.appendChild(b);
            
            let by = 0;
            const move = setInterval(() => {{
                if(gameOver) {{ clearInterval(move); b.remove(); return; }}
                by += speed;
                b.style.top = by + "px";

                const pRect = player.getBoundingClientRect();
                const bRect = b.getBoundingClientRect();

                if (bRect.bottom >= pRect.top && bRect.top <= pRect.bottom &&
                    bRect.left <= pRect.right && bRect.right >= pRect.left) {{
                    gameOver = true;
                    alert("💥 GAME OVER!");
                    location.reload();
                }}
                if(by > 500) {{ b.remove(); clearInterval(move); }}
            }}, 20);
        }}
        setInterval(createBox, spawnRate);
    </script>
</body>
</html>
"""

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html; charset=utf-8')
        self.end_headers()
        self.wfile.write(GAME_HTML.encode('utf-8'))

print(f"[{GAME_TITLE}] 서버가 80포트에서 시작되었습니다.")
HTTPServer(('0.0.0.0', 80), Handler).serve_forever()
EOF
```

```bash
ls
cat box_game_py.py
```

<br>


### 2단계: 컨테이너 생성 및 코드 배포 (Deploy)
작성한 파이썬 파일을 컨테이너 안으로 밀어 넣고 실행합니다.

```bash
# 1. 기존 컨테이너가 있다면 삭제 (alias crm 사용)
crm

# 2. 파이썬 컨테이너 실행 (8089 포트 연결)
docker run -itd --name box-server -p 8089:80 python:3.9-slim

# 3. 코드 복사
docker cp box_game_py.py box-server:/box_game_py.py

# 4. 게임 실행
docker exec -d box-server python3 /box_game_py.py
```

<br>


**결과 확인:** 웹 브라우저를 열고 `http://192.168.100.10:8089`에 접속합니다.

### 3단계: 코드 수정 및 재배포 실습
파이썬 변수를 수정하여 실제 서비스에 반영해 봅니다.

**파일 수정:**
`vi box_game_py.py` 명령어로 `BOX_SPEED`를 20으로, `SPAWN_RATE`를 300으로 수정하고 저장합니다.

<br>


**파일 재복사:** 수정된 파일을 컨테이너에 다시 덮어씁니다.

```bash
docker cp box_game_py.py box-server:/box_game_py.py
```

<br>


**서버 재시작:** 컨테이너를 재시작하여 바뀐 코드가 실행되도록 합니다.

```bash
docker restart box-server

# -d 옵션을 빼고 실행해서 에러가 나는지 직접 확인해 봅니다.
docker exec -d  box-server python3 /box_game_py.py
```

<br>


**결과 확인:** 브라우저를 새로고침하여 박스가 훨씬 빠르고 많이 내려오는지 확인합니다.

<br>


### 4단계: 모니터링 및 실습 종료
서비스 중인 컨테이너를 관리하고 정리합니다.

```bash
# 1. 현재 실행 중인 게임 서버의 자원 사용량 확인
docker stats box-server

# 2. 게임 서버의 로그 확인 (접속 기록 등)
docker logs box-server

# 3. 실습 종료 후 컨테이너 강제 삭제
docker rm -f box-server
```
