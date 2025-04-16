
System Activity Report의 약자로, 시스템의 CPU, 메모리, 입출력 사용량 정보를 수집하고 리포팅하는 명령어
시스템의 리소스 사용 이력을 모니터링하고 파일에 저장한 후 리포팅할 때 많이 사용한다.

`/var/log/sysstat` 에 일일 단위로 저장된다.


```
[root@localhost ~]# yum install sysstat
```

```
sar -옵션 n1 n2
```

n1초 간격으로 n2번 옵션을 추적한다.

* -u : CPU
* -r : 메모리 사용률
* -n DEV(네트워크 디바이스) : 네트워크 사용률
	* eth0, ens33 와 같은 네트워크 디바이스의 패킷 전송·수신 상태
* -q : 시스템 로드(load average) 및 프로세스(queue)
* -p -u -P {CPU번호} : 특정 CPU 코어의 성능 데이터 조회
	* sysstat 11.x 이상의 경우(현재 시스템)
* ==-d : 디스크 I/O 상태 확인== 
	* dev8-0 이렇게 보이는데 8은 major, 0은 minor값임. 자세한 디스크 상황을 보려면proc disk +tab키 , iostat? =>어떤 디스크가 쓰고 잇는지도 알 수 있다

```
[root@localhost ~]# sar -d
Linux 4.18.0-553.el8_10.x86_64 (localhost.localdomain)  04/08/2025      _x86_64_        (2 CPU)

19:09:25     LINUX RESTART      (2 CPU)

19:45:15     LINUX RESTART      (2 CPU)

20:00:23     LINUX RESTART      (2 CPU)

22:16:33     LINUX RESTART      (2 CPU)

```
리부팅 한 기록만 남아 있음.
`/etc/sysconfig/sysstat` 에서 `SADC_OPTIONS=" -S DISK"` 옵션으로 디스크 활동에 대한 로그 파일은 기록하고 있을 것임

```
[root@localhost ~]# sadc -S DISK 1 3 /var/log/sa/sa$(date +%d)
```

`sadc`로 raw 데이터 수집한다.
1초마다 3번 측정
`/var/log/sa/sa$(date +%d)` # sa 로그 위치

그냥 `sadc`로 하면 실행이 안됨(`command not found.)`. `sysstat` 패키지를 사용 중이기 때문에 `sadc`가 안깔려있는 건 아님. 
따라서 ==직접 `/usr/lib64/sa` 경로를 쳐서 `sadc` 명령어 사용==하거나
심볼릭 링크 만들어서 사용한다.
`ln -s /usr/lib64/sa/sadc /usr/local/bin/sadc`


```
[root@localhost ~]# sar -d -f /var/log/sa/sa$(date +%d)
Linux 4.18.0-553.el8_10.x86_64 (localhost.localdomain)  04/08/2025      _x86_64_        (2 CPU)

19:09:25     LINUX RESTART      (2 CPU)

19:45:15     LINUX RESTART      (2 CPU)

20:00:23     LINUX RESTART      (2 CPU)

22:16:33     LINUX RESTART      (2 CPU)

10:20:01 PM       DEV       tps     rkB/s     wkB/s   areq-sz    aqu-sz     await     svctm     %util
10:28:38 PM  dev259-0      0.20      1.28      1.75     14.93      0.00      1.59      0.83      0.02
10:28:38 PM  dev259-3      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
10:28:38 PM  dev259-4      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
...(생략)
```
* DEV : 디바이스 이름
* tps : 초당 I/O 요청 횟수 (read + write 합계)
* %util : 디바이스 사용률
	* 높으면 병목 가능성
* rkB/s : 초당 읽은 데이터 양(kb)
* wkB/s : 초당 쓴 데이터 양(kb)
* areq-sz : 평균 요청 크기(kb)
* aqu-sz : 평균 대기 큐 길이(I/O 요청이 대기 중인 수)
	* 크면 큐에 많이 쌓인다 -> 
* await : 평균 대기 시간(ms) -> 요청~완료까지 걸린 시간
	* 높으면 처리 지연 가능성
* svctm : 서비스 시간(ms) -실제 디바이스가 I/O 처리에 소요한 시간
* %util : 디바이스 사용률(%) 

```          6
[root@localhost ~]# sar -q 2 3
Linux 4.18.0-553.el8_10.x86_64 (localhost.localdomain)  04/02/2025      _x86_64_        (2 CPU)

12:52:33 AM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
12:52:35 AM         1       176      0.00      0.03      0.04         0
12:52:37 AM         0       176      0.00      0.03      0.04         0
12:52:39 AM         0       176      0.00      0.03      0.04         0
Average:            0       176      0.00      0.03      0.04         0

```

	runq-sz: 실행 대기 중인 프로세스 수(CPU를 기다림)
	plist-sz: 현재 실행 중이거나 대기 중인 프로세스 총 개수
	ldavg-1 : 최근 1분간의 Load Average
	5는 5분, 15는 15분..

* runq-sz 값 > CPU core 개수일 경우 과부하 상태
* ldavg 값이 지속적으로 높으면 과부하 가능성 

** `ldavg`값은 CPU 코어 갯수 따라 다름 -> 여기선 2개이기 때문에 2.00이면 100% 사용하는 것.
과부하 가능성, [[하이퍼스레딩]]으로 인해 `ldavg` * 2 = cpu core 개수로 생각해야 된다


```
[root@localhost ~]# sar -p -u -P 0 1 2
Linux 4.18.0-553.el8_10.x86_64 (localhost.localdomain)  04/02/2025      _x86_64_        (2 CPU)

01:13:33 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
01:13:34 AM       0      0.00      0.00      2.04      0.00      0.00     97.96
01:13:35 AM       0      1.00      0.00      4.00      0.00      0.00     95.00
Average:          0      0.51      0.00      3.03      0.00      0.00     96.46

[root@localhost ~]
01:12:49 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
01:12:50 AM       1      1.00      0.00      2.00      0.00      0.00     97.00
01:12:51 AM       1      0.00      0.00      0.00      0.00      0.00    100.00
Average:          1      0.51      0.00      1.01      0.00      0.00     98.48

[root@localhost ~]# sar -p -u -P 2 1 2
Linux 4.18.0-553.el8_10.x86_64 (localhost.localdomain)  04/02/2025      _x86_64_        (2 CPU)

01:13:07 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle

```

ex. `sar -p -u -P n 1 2`
CPU n번 코어에 대한 정보 출력

=>  `sar -p -u 1 2`. 이렇게 씀 -P는 쓰지 않음. 그냥 한 방에 확인하는 게 정석



| 필드            | 설명                                                                                                                                            |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `CPU`         | 물리적 CPU 번호 (0부터 시작)                                                                                                                           |
| `%user`       | **사용자 프로세스**(CPU) 사용량<br>(커널x, 유저 o, 애플리케이션 o)                                                                                                |
| `%nice`       | `nice` 우선순위로 실행된 프로세스의 CPU 사용량<br>(높을수록 우선순위 낮다)                                                                                              |
| `%system`     | 시스템 **커널 모드**에서 사용된 CPU 비율<br>ex. 디스크 쓰기, 네트워크 요청 처리                                                                                          |
| ==`%iowait`== | I/O 대기 시간 (디스크 입출력 때문에 대기 중인 비율)<br>ex. 대량의 로그 기록, DB 쿼리 실행, 백업 작업 등<br>-> ==디스크에 문제가 있을 때 확인하기 좋다.==                                         |
| ==`%steal`==  | 가상화 환경에서 다른 VM에 의해 CPU가 사용된 비율<br>말 그대로 다른 가상 머신에 의해 훔쳐진 시간.<br>VM이 호스트 머신의 리소스를 공유할 때 나타남.<br>ex. 한 hyp에 여러 대 vm 띄웠을 때, 호스트 자체의 CPU가 부족할 때.. |
| `%idle`       | 유휴(Idle) 상태 비율                                                                                                                                |
- `%user`가 높으면 애플리케이션 부하가 높다.
- `%system`이 높으면 커널에서 CPU를 많이 사용 중
- `%iowait`이 높으면 디스크 I/O 병목 가능성


```
[root@localhost ~]# sar -u 5 3
Linux 4.18.0-553.el8_10.x86_64 (localhost.localdomain)  03/25/2025      _x86_64_        (2 CPU)

01:23:04 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
01:23:09 PM     all      0.61      0.00      2.74      0.00      0.00     96.65
01:23:14 PM     all      0.51      0.00      2.22      0.00      0.00     97.27
01:23:19 PM     all      0.41      0.00      2.34      0.00      0.00     97.25
Average:        all      0.51      0.00      2.43      0.00      0.00     97.06

```

	%user: 일반 프로세스가 사용한 CPU 사용률
	%nice: 우선순위가 변경된 프로세스가 사용한 CPU 사용률
	%system : 커널 레벨 프로세스가 사용한 CPU 사용률
	%iowait : I/O 대기시간
	%idle: CPU가 쉬는 시간


```
[root@localhost ~]# sar -r 5 3
Linux 4.18.0-553.el8_10.x86_64 (localhost.localdomain)  03/25/2025      _x86_64_        (2 CPU)

01:36:53 PM kbmemfree   kbavail kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
01:36:58 PM   2108516   2365248    712076     25.25      4204    376312    297076      6.04    271256    197760         0
01:37:03 PM   2108664   2365396    711928     25.24      4204    376312    297076      6.04    271256    197656         0
01:37:08 PM   2108576   2365308    712016     25.24      4204    376312    296816      6.04    271256    197724         0
Average:      2108585   2365317    712007     25.24      4204    376312    296989      6.04    271256    197713         0

```
* kbmemfree : 시스템에서 현재 사용 가능한 여유 메모리 크기
* kbavail : 실제 사용 가능한 메모리
* 실제 사용중인 메모리
* 사용중인 메모리의 퍼센트
* kbbuffers: 리눅스 커널이 블록 장치 작업을 위해 사용하는 버퍼 영역의 크기 - 잠시 저장하는 곳 <- IO 최적화
* kbcached : 디스크에서 읽은 데이터를 잠시 캐싱해두는 곳<- IO 최적화
* kbcommit : 시스템에서 현재 할당(commit)된 메모리 크기
* 퍼센트
* kbactive : 현재 활성 상태인 메모리 크기
* kbinact: 안쓰는 메모리 크기
* kbdirty 아직 디스크에 기록되지 않은 수정된 페이지(Dirty Page)의 크기
	* dirty는 메모리가 디스크에 올리기 전에 사용하는 값이다.

```
[root@localhost ~]# sar -n DEV 5 3
Linux 4.18.0-553.el8_10.x86_64 (localhost.localdomain)  03/25/2025      _x86_64_        (2 CPU)

01:37:32 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
01:37:37 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:37:37 PM    ens160      4.80      7.80      0.34      0.93      0.00      0.00      0.00      0.00
01:37:37 PM    ens224      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:37:37 PM     bond0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

01:37:37 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
01:37:42 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:37:42 PM    ens160      5.60      9.20      0.40      1.07      0.00      0.00      0.00      0.00
01:37:42 PM    ens224      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:37:42 PM     bond0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

01:37:42 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
01:37:47 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:37:47 PM    ens160      4.20      7.00      0.30      0.80      0.00      0.00      0.00      0.00
01:37:47 PM    ens224      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:37:47 PM     bond0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
Average:           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:       ens160      4.87      8.00      0.35      0.93      0.00      0.00      0.00      0.00
Average:       ens224      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:        bond0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

```

* IFACE : 네트워크 인터페이스 이름
* rxpck/s; 초당 수신 패킷 수
* txpck/s: 초당 전송 패킷 수
* rxkB/s: 초당 수신되는 데이터 양
* txkB/s: 초당 전송되는 데이터 양
* 수신/전송되는 압축된 패킷 수
* 초당 수신되는 멀티캐스트 패킷 수
* 해당 네트워크 인터페이스의 대역폭 사용률 - 네트워크 속도 대비 얼마나 사용중인지?

## sysstat 패키지 내 명령어
#### sadc
System Activity Data Collector - 데이터 수집기
시스템 데이터를 주기적으로 수집하여 저장한다.
이 성능 데이터를 `saXX`에 저장한다.

##### `saXX` 의 위치

* **Rocky Linux 7**
	* `/var/log/sa/saXX`
```
[root@localhost ~]# cd /var/log/sa/
[root@localhost sa]# ls
sa01

```
* Rocky Linux 8
	* `/var/log/sysstat/saXX`

#### sa
System Activity Analyzer - 데이터 요약
`sadc`가 수집한 데이터 파일(`saXX`)을 읽어서 요약 정보를 보여준다.
#### sar
System Activity Reporter
실시간 or 기록된 데이터를 사람이 보기 쉽게 출력하는 명령어


## sar config의 위치

Rocky Linux 7 -> 8에서 `sar`의 설정 파일 경로가 변경되었다.
* **Rocky Linux 7**
	* `/etc/sysconfig/sysstat`
* Rocky Linux 8
	* `/etc/sysstat/sysstat`

### sar 지표 옵션 수정 및 로그 저장 일자

```
[root@localhost ~]# vi /etc/sysconfig/sysstat

# sysstat-11.7.3 configuration file.

# How long to keep log files (in days).
# If value is greater than 28, then use sadc's option -D to prevent older
# data files from being overwritten. See sadc(8) and sysstat(5) manual pages.
HISTORY=28

# Compress (using xz, gzip or bzip2) sa and sar files older than (in days):
COMPRESSAFTER=31

# Parameters for the system activity data collector (see sadc manual page)
# which are used for the generation of log files.
SADC_OPTIONS=" -S DISK"

# Directory where sa and sar files are saved. The directory must exist.
SA_DIR=/var/log/sa

# Compression program to use.
ZIP="xz"
	
# By default sa2 script generates yesterday's summary, since the cron job
# usually runs right after midnight. If you want sa2 to generate the summary
# of the same day (for example when cron job runs at 23:53) set this variable.
#YESTERDAY=no

# By default sa2 script generates reports files (the so called sarDD files).
# Set this variable to false to disable reports generation.
#REPORTS=false

```

* HISTORY : sar 파일 보관 일자 설정 -> 28일
* COMPRESSAFTER : n일 뒤에 압축한다.
	* 디스크 공간 절약을 위해 일정 기간이 지난 후 압축하도록 한다.
* SADC_OPTIONS : sar에 추가 제공 옵션. 공백일 경우 기본 지표만 수집한다.
	* `-S ~` 를 하면 특정 데이터를 선택적으로 수집 가능하다.
	* ex. `-S DISK/CPU/ALL` - 각각 디스크만/CPU만/디스크, CPU, 메모리, 네트워크 ...
* SA_DIR : `saXX`, `sar`파일이 저장되는 곳
* ZIP : 압축 방식
	* xz말고도 gzip, zip, bzip2 등 있음
* YESTERDAY
	* yes : 어제의 데이터 기준(default)
	* no : 오늘의 데이터를 기준으로 데이터 생성
* REPORTS
	* true: sar 파일(report) 생성 여부(default)

[[cron]]
