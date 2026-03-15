# 컨테이너 기술

### **chroot**

chroot는 파일을 분리(격리)하여 독립된 공간을 생성한다. 보안을 위해 새로운 가상 root 디렉토리를 생성하는 명령으로 chroot로 가상 공간을 확보하면 상위 디렉토리로 이동이 불가능하다.
유닉스의 오래된 기술로, www, ftp, dns와 같은 서비스에서 사용된다.

- chroot 사용예(rocky linux)
    
    ```bash
    # tree 패키지(프로그램) 설치
    sudo dnf install -y tree
    
    #애플리케이션 디렉토리 생성후 root 디렉토리로 전환하기
    mkdir appdir
    chroot /home/admin/appdir/
    
    # 애플리케이션과 필요한 라이브러리 검색한 후 복사 저장하기
    ldd /bin/bash
            linux-vdso.so.1 (0x00007fffeade8000)
            libtinfo.so.6 => /lib64/libtinfo.so.6 (0x00007f365ca69000)
            libc.so.6 => /lib64/libc.so.6 (0x00007f365c800000)
            /lib64/ld-linux-x86-64.so.2 (0x00007f365cbff000)
    
    mkdir appdir/bin
    mkdir appdir/lib64
    cp /bin/bash appdir/bin/
    cp /lib64/libtinfo.so.6 /lib64/libc.so.6 /lib64/ld-linux-x86-64.so.2 appdir/lib64/
    
    cp /bin/date appdir/bin/
    tree appdir/
    appdir/
    ├── bin
    │   ├── bash
    │   └── date
    └── lib64
        ├── ld-linux-x86-64.so.2
        ├── libc.so.6
        └── libtinfo.so.6
    
    sudo chroot appdir/
    bash-5.1# pwd
    /
    
    bash-5.1# date
    Sun Mar 15 10:52:08 UTC 2026
    
    bash-5.1# ls
    bash: ls: command not found
    
    bash-5.1# exit
    exit
    ```
    

### **Namespaces**

Docker는 namespaces를 이용해 container간에 독립된 공간(isolated workspace)을 제공한다.

2002년 2.4.19 커널에서 mount namespace로 시작되었고, 컨테이너 지원 기능은 사용자 네임 스페이스의 도입과 함께 커널 버전 3.8에서 완료되었다.

- pid : Process isolation
- mnt : Managing filesystem mount points
- net : Managing network interfaces
- ipc : Managing access to IPC Resources
- uts : Isolating kernel and version identifiers.
- user : account isolation
- cgroup(Kernel 4.6 이상)
- time(Kernel 5.6)

- namespace  구성  정보 보기
    
    ```bash
    # 현재 동작중인 커널 버전 확인
    uname -r
    5.14.0-570.17.1.el9_6.x86_64
    
    # 동작중인 프로세스 폴더 이동
    sudo -i
    cd /proc/
    cd XXX
    ls
    
    # XXX 프로세스가 사용중인 namespace 리스트 확인
    cd ns
    ls
    cgroup  ipc  mnt  net  pid  pid_for_children  time  time_for_children  user  uts
    
    exit
    ```
    

### **cgroups**

Docker 엔진은 cgroup(control group)를 이용해 자원을 제한하거나 할당한다. 메모리나 CPU의 특정한 양을 container에 할당할 수 있다.

- cgroup 설정 정보 보기
    
    ```bash
    sudo -i
    
    cd /sys/fs/cgroup
    ls
    
    # 애플리케이션 디렉토리 생성
    # cgroup은 애플리케이션 별로 각각 cpu, memory, netowrk, io의 사용을 설정할수 있다
    mkdir appdir
    cd appdir
    ls
    exit
    
    ```
