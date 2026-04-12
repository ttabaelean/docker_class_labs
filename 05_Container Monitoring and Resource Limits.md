성미님, 요청하신 내용을 바탕으로 GitHub에 바로 업로드하여 사용할 수 있는 깔끔한 Markdown 포맷의 실습 Lab 문서를 만들어 드립니다. 제목은 영어로 변경하였으며, 업로드하신 이미지의 가이드라인(볼드체, 코드 블록, 설명 구조)을 충실히 따랐습니다.

---

# 05. Container Monitoring and Resource Limits

## 1. 컨테이너 모니터링

### **실습 준비**

**smlinux/stress 컨테이너 이해**
- **stress**는 리눅스 시스템에 의도적으로 부하(Load)를 가하여 시스템의 안정성을 테스트하거나, 모니터링 및 리소스 제한 설정이 잘 작동하는지 확인하기 위해 사용하는 부하 테스트 도구입니다.
- CPU, 메모리, 디스크 I/O 등 특정 자원만 골라서 풀 부하(100%)를 줄 수 있습니다.
    - `--cpu <N>`: N개의 워커(Worker) 프로세스를 생성하여 CPU 코어에 부하를 줍니다.
    - `--vm <N>`: 메모리 할당을 시도하는 N개의 프로세스를 실행합니다.
    - `--vm-bytes <Size>`: 각 메모리 워커가 점유할 메모리 양을 설정합니다. (예: 128m)
    - `--timeout <Sec>`: 지정된 시간(초)이 지나면 부하 테스트를 자동으로 종료합니다.

**#Dockerfile**
```dockerfile
FROM debian
MAINTAINER Seongmi lee <seongmi.lee@gmail.com>
RUN apt-get update; apt-get install stress -y
# 컨테이너 실행과 동시에 8개의 CPU 워커를 가동하여 부하를 발생시킴
CMD ["stress", "--cpu", "8"]
```

**부하 모니터링 도구(htop) 설치**
- **htop**은 리눅스 시스템의 리소스(CPU, 메모리, 프로세스) 사용량을 실시간으로 보여주는 대화형 프로세스 뷰어입니다.
- **주요 단축키**:
    - `F4`: 특정 프로세스 검색 (예: stress 입력)
    - `F6`: 특정 기준(CPU%, MEM% 등)으로 정렬
    - `q`: 프로그램 종료

```bash
# 부하 모니터링 도구 설치
sudo dnf -y install epel-release
sudo dnf -y install htop

# 컨테이너 일괄 삭제용 alias 설정
alias crm='docker rm -f $(docker ps -aq)'
```

---

### **1.1 기본 명령어를 이용한 모니터링**

**1. 실시간 메트릭 확인 (docker stats)**
- stress 도구를 이용한 부하 생성 컨테이너를 실행하고 리소스 사용량을 확인합니다.

```bash
# 컨테이너 실행
# -d: 백그라운드 실행, --cpu 2: CPU 부하, --vm 1: 메모리 부하, --vm-bytes 128m: 메모리 점유량
docker run -d --name monitor-test smlinux/stress stress --cpu 2 --vm 1 --vm-bytes 128m

# 실시간 메트릭 확인
docker stats monitor-test
```
> **확인 사항**: CPU 사용률이 몇 %까지 올라가는지, 메모리 사용량(MEM USAGE)은 얼마인지 확인합니다.

**2. 로그 및 프로세스 확인**
- Nginx 웹 서버를 통해 실시간 접속 로그와 내부 프로세스를 확인합니다.

```bash
# Nginx 컨테이너 실행
docker run -d --name log-test -p 80:80 nginx

# 인위적인 접속 로그 발생 (별도 터미널 권장)
curl localhost
curl localhost/no-page  # 404 에러 유도

# 실시간 로그 확인 (-f: Follow 모드)
docker logs -f log-test

# 컨테이너 내부 실행 프로세스 확인
docker top log-test
```

**3. 호스트 OS 관점의 모니터링 (htop)**
```bash
htop
```
> **확인 사항**: 상단 CPU 바에서 어떤 코어가 사용 중인지 확인하고, `F4`로 stress 프로세스를 검색하여 관찰합니다.

---

### **1.2 고급 분석 도구 실습 (cAdvisor)**

구글의 **cAdvisor**를 실행하여 웹 브라우저에서 시각화된 데이터를 확인합니다.

```bash
# TEST 컨테이너 여러 개 실행
docker run --name testcon1 -d smlinux/stress 
docker run -m 500m -c 2048 --name testcon2 -d smlinux/stress
docker run -c 512 --name testcon3 -d smlinux/stress
docker run -c 512 --name testcon4 -d smlinux/stress

# cAdvisor 설치 및 실행
VERSION=0.55.1
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  ghcr.io/google/cadvisor:$VERSION

# 실습 종료 후 삭제
crm
```
> **확인**: `http://[서버-IP]:8080` 접속 후 Docker Containers 메뉴에서 그래프를 확인합니다.

---

## 2. 컨테이너 리소스 제한 실습

### **2.1 메모리 제한 실습: OOM Killer 확인**
- 메모리는 **압축 불가능한 리소스**임을 확인합니다.

```bash
# c1: 정상 작동 범위 (128MB 제한, 100MB 부하)
docker run -d --name mem-ok -m 128m smlinux/stress stress --vm 1 --vm-bytes 100m

# c2: OOM Killer 유도 (128MB 제한, 150MB 부하)
docker run --name mem-oom -m 128m smlinux/stress stress --vm 1 --vm-bytes 150m

# 상태 확인 (Exited 여부 확인)
docker ps -a

# 시스템 로그 확인 (OOM Killer 작동 여부 검증)
sudo journalctl -r
```

---

### **2.2 CPU 개수 제한 실습: 특정 코어 할당 (cpuset-cpus)**
```bash
# 0번 코어만 사용하여 부하 프로세스 실행
docker run -d --name cpu-pin --cpuset-cpus 0 smlinux/stress stress --cpu 2

# htop으로 0번 CPU만 가동되는지 확인
htop
crm
```

---

### **2.3 CPU 가중치 실습: 상대적 배분 (cpu-shares)**
```bash
# 가중치 2:1 비율 설정 (1024 vs 512)
docker run -d --name cpu-high --cpu-shares 1024 smlinux/stress stress --cpu 2
docker run -d --name cpu-low --cpu-shares 512 smlinux/stress stress --cpu 2

# CPU % 비율이 약 2:1로 유지되는지 확인
docker stats
crm
```

---

### **2.4 CPU Quota 실습: Throttling 확인**
```bash
# 전체 CPU 중 0.5개 분량만 사용하도록 제한
docker run -d --name cpu-limit --cpus 0.5 smlinux/stress stress --cpu 2

# CPU %가 50% 근처에서 고정되는지 확인
docker stats
crm
```

---

## 3. 종합 실습: Nginx 웹 서비스 운영 및 모니터링

### **3.1 서비스 배포 및 네트워크 확인**
```bash
# 1. Nginx 1.18 이미지 다운로드 및 실행
docker pull nginx:1.18
docker run -d -p 8001:80 --name=webserver1 nginx:1.18

# 2. 호스트 포트 점유 확인
sudo netstat -nlp | grep 8001
```

### **3.2 서비스 관제 (Monitoring)**
```bash
# 접속 테스트 및 실시간 리소스/로그 확인
curl localhost:8001
docker stats webserver1
docker logs -f webserver1
docker top webserver1
```

### **3.3 서비스 제어 및 상태 변화 관찰**
```bash
# 컨테이너 정지 및 접속 거부 확인
docker stop webserver1
curl localhost:8001

# 서비스 재개
docker start webserver1
```

### **3.4 고급 시각화 도구 활용**
- **cAdvisor 접속**: `http://[호스트IP]:8080`
- **확인 사항**: 브라우저 새로고침 시 **Network Throughput** 그래프의 변화를 확인합니다.

