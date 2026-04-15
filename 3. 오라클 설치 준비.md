## 모바텀으로 연결한다
<img width="826" height="478" alt="모바텀" src="https://github.com/user-attachments/assets/2db978cd-8730-43be-9fa7-761c8f3e954b" />
<img width="792" height="645" alt="터미널" src="https://github.com/user-attachments/assets/50610bbc-b9d4-4413-a64b-e246239f3eb0" />

- 에러가 떴는데 방화벽을 해제한다
<img width="769" height="148" alt="방화벽 해제" src="https://github.com/user-attachments/assets/40f486df-da01-4836-91fe-7d98bf08527e" />

- 그래도 안 돼서 호스트 전용 어댑터에서 어댑터에 브리지로 바꾸니까 성공!
<img width="739" height="502" alt="어댑터에 브리지" src="https://github.com/user-attachments/assets/0b55e827-bf97-489c-8553-ab395a532dd6" />
<img width="804" height="466" alt="성공" src="https://github.com/user-attachments/assets/d8832686-fad5-421e-9906-0198e28d5c57" />

- 호스트 전용은 ip대역이 고정되어 있는데 주소가 안 맞아서 안됐던 것이다



## 호환성 라이브러리 다운로드
```bash
yum -y install binutils compat-libcap1 compat-libstdc++-33 gcc gcc-c++ glibc glibc-devel ksh libgcc libstdc++ libstdc++-devel libaio libaio-devel make sysstat unixODBC unixODBC-devel elfutils-libelf-devel unzip
```
<img width="818" height="437" alt="yum" src="https://github.com/user-attachments/assets/65995230-210c-4dc1-9b44-2429d90a4b53" />

## 커널 매개변수 설정
- 리눅스 시스템의 메모리 할당량과 통신 설정을 오라클 전용으로 바꾸는 작업
```bash
[root@localhost ~]# vi /etc/sysctl.conf
[root@localhost ~]# cat /etc/sysctl.conf
# sysctl settings are defined through files in
# /usr/lib/sysctl.d/, /run/sysctl.d/, and /etc/sysctl.d/.
#
# Vendors settings live in /usr/lib/sysctl.d/.
# To override a whole file, create a new file with the same in
# /etc/sysctl.d/ and put new settings there. To override
# only specific settings, add a file with a lexically later
# name in /etc/sysctl.d/ and put new settings there.
#
# For more information, see sysctl.conf(5) and sysctl.d(5).
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 536870912
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586

-- 즉시 적용
[root@localhost ~]# sysctl -p
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 536870912
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586
```

##  유저 자원 사용 제한 설정
```bash
[root@localhost ~]# vi /etc/security/limits.conf

# 아래 내용 삽입
oracle soft nproc 2048
oracle hard nproc 65536
oracle soft nofile 1024
oracle hard nofile 65536
```
## 유저 생성, 환경변수, 권한 설정
- 오라클을 사용할 유저를 생성하고 패스워드를 설정.

```bash
[root@localhost ~]# groupadd oinstall
[root@localhost ~]# groupadd dba

[root@localhost ~]# passwd oracle
oracle 사용자의 비밀 번호 변경 중
새  암호:
잘못된 암호: 암호는 8 개의 문자 보다 짧습니다
새  암호 재입력:
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.

# 오라클을 설치할 디렉터리를 생성하고 위에서 만든 oracle 계정에 권한을 부여한다

[root@localhost ~]# mkdir -p /u01/app/oracle
[root@localhost ~]# chown -R oracle:oinstall /u01
[root@localhost ~]# chmod -R 775 /u01
```
## 오라클 계정으로 접속하여 .bash_profile 설정
```bash
[oracle@localhost ~]$ vi .bash_profile
[oracle@localhost ~]$ cat .bash_profile
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin

export PATH
export ORACLE_BASE=/u01/app/oracle/
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1
export ORACLE_SID=orcl
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib:/usr/local/lib
export PATH=$PATH:$ORACLE_HOME/bin

[oracle@localhost ~]$ source .bash_profile
```


## /home/oracle/에 6개의 설치파일 올리기 
<img width="1164" height="627" alt="설치파일" src="https://github.com/user-attachments/assets/1fbc3d3c-d0de-4e5c-9f86-544b9bca93be" />

```bash
[oracle@localhost ~]$ ls
V17488-01.zip       V17530-01_2of2.zip  V20606-01.zip  공개      문서      비디오  서식
V17530-01_1of2.zip  V17532-01.zip       V20609-01.zip  다운로드  바탕화면  사진    음악

```

## 오라클 DB 서버 폴더 압축 풀기
```bash
[oracle@localhost ~]$ ls -lh
합계 4.7G
-rw-r--r--. 1 oracle oinstall 613M  4월 15 16:48 V17488-01.zip
-rw-r--r--. 1 oracle oinstall 1.2G  4월 15 16:49 V17530-01_1of2.zip
-rw-r--r--. 1 oracle oinstall 1.1G  4월 15 16:47 V17530-01_2of2.zip
-rw-r--r--. 1 oracle oinstall 674M  4월 15 16:49 V17532-01.zip
-rw-r--r--. 1 oracle oinstall 653M  4월 15 16:50 V20606-01.zip
-rw-r--r--. 1 oracle oinstall 588M  4월 15 16:50 V20609-01.zip

# 가장 용량이 큰 것이 Oracle Database 11.2.0.1 서버

[oracle@localhost ~]$ unzip V17530-01_1of2.zip
Archive:  V77117-01.zip
  inflating: AAA_README_FIRST.TXT
  inflating: unzip_db11204.com
 extracting: db11204.zip

[oracle@localhost ~]$ unzip V17530-01_2of2.zip

```
## 실행
```bash
[oracle@localhost ~]$ ls -F
V17488-01.zip       V17530-01_2of2.zip  V20606-01.zip  database/  다운로드/  바탕화면/  사진/  음악/
V17530-01_1of2.zip  V17532-01.zip       V20609-01.zip  공개/      문서/      비디오/    서식/

[oracle@localhost ~]$ cd database
[oracle@localhost database]$ ls
install  response  rpm  runInstaller  sshsetup  stage  welcome.html

[oracle@localhost database]$ LANG=C ./runInstaller
Starting Oracle Universal Installer...

Checking Temp space: must be greater than 120 MB.   Actual 31716 MB    Passed
Checking swap space: must be greater than 150 MB.   Actual 5119 MB    Passed
Checking monitor: must be configured to display at least 256 colors.    Actual 16777216    Passed
Preparing to launch Oracle Universal Installer from /tmp/OraInstall2026-04-15_05-18-38PM. Please wait ...[oracle@localhost database]$

```


