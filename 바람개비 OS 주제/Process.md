- 현재 실행 중인 프로세스를 확인하는 명령어들
- 프로세스 별 리소스 점유율 확인
* PID, PPID 와 같은 명칭 의미


### ps

UNIX 계열 운영 체제에서 현재 실행 중인 프로세스 목록을 표시하는 데 사용된다.

#### `ps -ef`

주로 System V 계열의 UNIX 시스템에서 사용된다.

프로세스 간의 관계를 파악하고 싶을 때 주로 사용한다.

- 옵션
 ![[Pasted image 20250226012017.png]]
-e : 모든 프로세스 나열
-f : 전체 포맷을 사용, 각 프로세스에 대한 상세한 정보를 보여준다.
-eo : 원하는 컬럼만 보기

![[Pasted image 20250226012522.png]]
  `UID       PID      PPID C STIME TTY       TIME        CMD`
  
- 출력 내용
    - UID : 사용자 ID
    - PID : 프로세스 ID
    - PPID : 부모 프로세스 ID
    - C : 프로세서 사용량
    - STIME : 시작 시간
    - TTY : 터미널 타입
    - TIME : 총 CPU 사용 시간
    - CMD : 실행 명령어

#### `ps aux`

BSD 계열의 UNIX 시스템에서 사용되며, Linux 시스템에서도 널리 사용되고 있다.
시스템의 CPU와 메모리 사용 상황을 보고 싶을 때 주로 사용

- 옵션

a : 다른 사용자의 프로세스를 포함한 모든 프로세스를 나열
u : 프로세스의 소유자에 대한 정보를 나타내는 사용(user)
![[Pasted image 20250226012618.png]]
`USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND`  
- 출력 내용
    - USER : 소유자
    - PID : 프로세스 ID
    - %CPU : CPU 사용량 비율
    - MEM : 메모리 사용량 비율
    - VSZ : 가상 메모리 사이즈
    - RSS : 실제 메모리 사이즈
    - TTY : 터미널 타입
    - STAT : 프로세스 상태
    - START : 프로세스 시작 시간
    - TIME : 총 CPU 사용 시간
    - COMMAND : 실행 명령어


### 메모리 지표

##### VSZ(Virtual Memory Size)
* 가상 메모리 사용량 (KB)
* 프로세스가 점유한 전체 가상 메모리 크기
* 실제 물리 메모리 사용과는 일치하지 않을 수도 있다. (공유 라이브러리, swap, mapping된 파일을 포함)
* 메모리 누수가 어디서 되는지 참고할 수 있다.

##### RSS(Resident Set Size)
* 실제 물리 메모리 사용량(KB)
* 프로세스가 현재 물리 메모리에 적재되어 있는 양
* 가장 많이 참조되는 지표
##### %MEM
* 시스템 전체 메모리 대비 해당 프로세스의 메모리 사용 비율
* RSS 기준으로 산출

##### VSZ, RSS 차이
`[root@localhost ~]# ps aux`
`USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND`
`root           1  0.0  0.3 175104 13400 ?        Ss   19:23   0:03 /usr/lib/syst`
`root           2  0.0  0.0      0     0 ?        S    19:23   0:00 [kthreadd]`
`root           3  0.0  0.0      0     0 ?        I<   19:23   0:00 [rcu_gp]`
`root           4  0.0  0.0      0     0 ?        I<   19:23   0:00 [rcu_par_gp]`
`root           5  0.0  0.0      0     0 ?        I<   19:23   0:00 [slub_flushwq`
`root           7  0.0  0.0      0     0 ?        I<   19:23   0:00 [kworker/0:0H`
`root          10  0.0  0.0      0     0 ?        I<   19:23   0:00 [mm_percpu_wq`
`root          11  0.0  0.0      0     0 ?        S    19:23   0:00 [rcu_tasks_ru`
`root          12  0.0  0.0      0     0 ?        S    19:23   0:00 [rcu_tasks_tr`
`root          13  0.0  0.0      0     0 ?        S    19:23   0:00 [ksoftirqd/0`

`[root@localhost ~]# ps -eo pid,comm,pmem,rss,vsize`
    `PID COMMAND         %MEM   RSS    VSZ`
      `1 systemd          0.3 13400 175104`
      `2 kthreadd         0.0     0      0`
      `3 rcu_gp           0.0     0      0`
      `4 rcu_par_gp       0.0     0      0`





### =TTY 종류=
프로세스와 연결된 터미널(콘솔)을 나타낸다.

* tty1 등 : 물리적 콘솔 터미널
```
root       10742  0.0  0.1 318960  5912 tty1     S    19:45   0:00 su - shchoi
shchoi     10743  0.0  0.1 226148  5144 tty1     S    19:45   0:00 -bash
root       14020  0.0  0.1 318960  5772 tty1     S    19:54   0:00 su - root

```
* pts/0 pts/1 등 : 원격 ssh나 터미널 에뮬레이터에서 할당된 가상 터미널(pseudo-terminal)
```
root        5856  0.0  0.1 226284  5332 pts/0    Ss   19:30   0:00 -bash
```

* ? : 터미널에 연결 x (데몬 등)
```
root       15364  0.0  0.0 234796  2556 ?        Ss   20:01   0:00 /usr/sbin/ana
root       20244  0.0  0.2 128616 10516 ?        Ss   20:22   0:00 sshd: shchoi
shchoi     20403  0.0  0.2  89656  9640 ?        Ss   20:22   0:00 /usr/lib/syst
shchoi     20407  0.0  0.1 228512  4708 ?        S    20:22   0:00 (sd-pam)

```

#### =STAT 종류=

프로세스의 현재 상태와 특성을 나타낸다.
* R : Running or Runnable
`root       32600  0.0  0.1 257504  3732 pts/0    R+   20:57   0:00 ps aux
`
* S : Sleeping
`root           1  0.0  0.3 175104 13400 ?        Ss   19:23   0:03 /usr/lib/syst`

* I : Interruptable Sleep  - 주로 I/O 요청 등으로 인터럽트할 수 없는 상태
`root         475  0.0  0.0      0     0 ?        I<   19:23   0:00 [scsi_tmf_16]`

* T : sTopped - 신호에 의한 중단, 디버깅

* Z : Zombie : 프로세스 종료 후 부모 프로세스가 수집하지 않은 상태
프로세스 종료 시 커널이 프로세스의 Exit status를 보관해야 하는데, 이를 부모가 수집하지 않게 될 때.


**플래그 문자**
* <: 고우선순위 프로세스(nice 값이 양수)
`root         434  0.0  0.0      0     0 ?        I<   19:23   0:00 [scsi_tmf_0]
`
* N : 우선순위 낮음(nice값이 음수)
`root          35  0.0  0.0      0     0 ?        SN   19:23   0:00 [khugepaged]
`
* L :일부 메모리 페이지가 잠겨 있다

* s : Session leader : 해당 프로세스가 세션의 리더이다
`root        5818  0.0  0.1 226152  4960 tty1     Ss   19:28   0:00 -bash
`

* l : multi-Thread : 멀티스레드
`root         952  0.0  0.5 590684 18848 ?        Ssl  19:23   0:00 /usr/sbin/Net`

* + :Foreground process group : 포어그라운드 프로세스 그룹에 속한다
`root       32600  0.0  0.1 257504  3732 pts/0    R+   20:57   0:00 ps aux
`

### nice
프로세스의 CPU 스케줄링 우선순위를 조정하는 값
시스템에서 여러 프로세스가 실행될 때 CPU 사용 우선순위를 제어할 수 있다.

nice 값은 커널 값보다 먼저 가게 설정할 수도 있으니 건들지 말자.

-20 ~ 19 까지의 값, 기본값은 0

`nice -n 숫자 command`
command 명령어의 우선순위 조정

`sudo renice 숫자 -p 1234`
PID 1234인 프로세스의 nice 값을 '숫자'로 변경한다.


### =Background 작업을 Foreground로 전환하기=

#### 1. fg
가장 최근에 백그라운드로 실행한 작업을 포어그라운드로 가져온다.

`fg %[작업번호]`

작업 번호를 확인하기 위해선 ===`jobs`=== 명령어로 확인해야 한다.

`jobs`


```
[root@localhost ~]# sleep 1000&
[1] 159350
[root@localhost ~]# jobs
[1]+  Running                 sleep 1000 &
[root@localhost ~]# sleep 2000&
[2] 159393
[root@localhost ~]# sleep 30000&
[3] 159418
[root@localhost ~]# jobs
[1]   Running                 sleep 1000 &
[2]-  Running                 sleep 2000 &
[3]+  Running                 sleep 30000 &

```
sleep으로 백그라운드 작업 만들어 둔다.


```
[root@localhost ~]# fg %3
sleep 30000
```

콘솔이 안떨어지는 상태로 유지된다.

```
[root@localhost ~]# fg
sleep 30000
```
인자 없이 `fg`만 입력하면 + 붙은 것 먼저 실행된다.(어차피 +는 한 개 밖에 없음)


### =Foreground 작업을 Background로 전환하기=

#### 1. Ctrl + z 후 bg
```
[root@localhost ~]# fg %3
sleep 30000
^Z
[3]+  Stopped                 sleep 30000
[root@localhost ~]# jobs
[1]   Running                 sleep 1000 &
[2]-  Running                 sleep 2000 &
[3]+  Stopped                 sleep 30000
[root@localhost ~]# bg
[3]+ sleep 30000 &
[root@localhost ~]# jobs
[1]   Running                 sleep 1000 &
[2]-  Running                 sleep 2000 &
[3]+  Running                 sleep 30000 &


```

Ctrl + Z  하면 Stopped 된다
`bg` 로 백그라운드에서 Running 된다.

> Ctrl + C 하면 그냥 종료된다.


#### =+, -의 역할=

**+: 기본작업**
기본 백그라운드 작업

**-: 대체 기본 작업**
그 다음 우선 순위의 작업

 오직 하나의 작업만 기본 작업으로 지정될 것, 다른 작업은 - 혹은 아무 표시 없음.

### disown
터미널을 닫아도 프로세스를 유지할 수 있다.
터미널 종료 시 SIGHUP 신호를 보내지 않도록 "보호"하는 기능

* `-h` : `jobs`에 그대로 남음
그냥 `disown` 하면 jobs에 안보임

```
[root@localhost ~]# sleep 1234 &
[1] 172639
[root@localhost ~]# jobs
[1]+  Running                 sleep 1234 &
[root@localhost ~]# disown -h %1
[root@localhost ~]# jobps
-bash: jobps: command not found
[root@localhost ~]# jobs
[1]+  Running                 sleep 1234 &

```

터미널 껐다 다시 켠다

```
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Wed Mar 12 18:59:36 2025 from 192.168.38.1
[root@localhost ~]# ps aux | grep 1234
root      172639  0.0  0.1 217156   824 ?        S    19:00   0:00 sleep 1234

```

sleep 1234 가 보인다.
& 는 연산자라서 명령줄에서 안나타남.




-> `kill -CONT PID` 
SIGCONT 신호를 보내 stopped된 프로세스 계속 실행 가능하다.
그렇게 해도 `disown` 해서 쉘의 job 목록에서 제거되어 있다면 `fg`해도 포어그라운드로 가져올 수 없다.


### nohup
터미널을 닫아도 프로세스를 유지할 수 있다.
```
[root@localhost ~]# nohup sleep 3333 &
[1] 176617
[root@localhost ~]# nohup: ignoring input and appending output to 'nohup.out'
^C

```

로그값이 계속 저장되기 때문에 콘솔이 안 떨어진다.

```
[root@localhost ~]# nohup sleep 23223 > /dev/null 2>&1 &
[2] 176780
[root@localhost ~]# jobs
[1]-  Running                 nohup sleep 3333 &
[2]+  Running                 nohup sleep 23223 > /dev/null 2>&1 &

```

`> /dev/null 2>&1 &` 로 로그 무시 가능

### ==명령어의 위치==

#### 리눅스 명령어
이진 실행 파일 형태
`/usr/bin` 
`/bin` 
`/sbin` 
`/usr/sbin`

 과 같은 디렉토리에 위치한다


`/bin` 
```
[root@localhost bin]# ll | grep 'ls'
-rwxr-xr-x. 1 root root      34160 Jan 18  2023 false
-rwxr-xr-x. 1 root root     268888 May 25  2024 grub2-menulst2cfg
-rwxr-xr-x. 1 root root      12376 Oct  1  2022 idiag-socket-details
-rwxr-xr-x. 1 root root     143296 Jan 18  2023 ls
```

`/sbin` 
```
[root@localhost sbin]# ll | grep ifconfig
-rwxr-xr-x. 1 root root   82728 Apr  7  2021 ifconfig

```

`/usr/sbin`
```
[root@localhost sbin]# ls
accessdb              fsfreeze                     lvrename                   setpci
accton                fstrim                       lvresize                   setquota
adcli                 fuse2fs                      lvs                        setsebool
addgnupghome          fuser                        lvscan                     sfdisk
addpart               g13-syshelp                  makedumpfile               shutdown
adduser               genhomedircon                matchpathcon               slattach
agetty                genhostid                    mcelog                     smartctl
alternatives          genl                         mdadm                      smartd
anacron               getcap                       mdmon                      sos
applygnupgdefaults    getenforce                   mii-diag                   sos-collector
arp                   getpcaps                     mii-tool                   sosrepo
...(생략)

[root@localhost sbin]# ll | grep NetworkManager
-rwxr-xr-x. 1 root root 3579408 Apr  3  2024 NetworkManager

```
`/usr/bin`
```
[root@localhost bin]# cd /usr/bin
[root@localhost bin]# ls
'['                                   nl-qdisc-list
 ac                                   nl-route-add
 addr2line                            nl-route-delete
 alias                                nl-route-get
 apropos                              nl-route-list
 ar                                   nl-rule-list
 arch                                 nl-tctree-list
 as                                   nl-util-addr
 at                                   nm
 atq                                  nmcli
 atrm                                 nm-online
 attr                                 nmtui
 aulast                               nmtui-connect
 aulastlog                            nmtui-edit
 ausyscall                            nmtui-hostname

```


### ==UUID 저장 장소==

#### UUID
Universally Unique Identifier
전 세계적으로 중복되지 않는 유일한 식별자
16바이트 크기의 값이며, 32자리 16진수 형태로 표현한다.
* 파일 시스템이나 파티션, LVM 볼륨 등 시스템 리소스 식별에 사용한다.
* 분산환경(ex. 마이크로서비스, 데이터베이스 PK)에서 고유 키 값으로 활용한다.
* 프로그램, 세션 등의 유니크 식별자

	*GPT(GUID Partition Table) 방식의 파티션은, 디스크 파티션 테이블 자체에 'Partition GUID 라는 것을 저장하며, 이는 파일시스템 UUID가 아닌, 파티션 자체에 부여되는 GUID 값이다.*

#### 파일시스템 UUID

==`blkid` , `lsblk`로 찾기==
```
[root@localhost ~]# blkid
/dev/nvme0n1: PTUUID="4d81d0d7" PTTYPE="dos"
/dev/nvme0n1p1: UUID="48863184-18e5-426f-8dce-25ca10bf17a4" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="4d81d0d7-01"
/dev/nvme0n1p2: UUID="Gbjpzt-wZIv-7FdF-Rkyf-ebDM-rb4f-xjplKi" TYPE="LVM2_member" PARTUUID="4d81d0d7-02"
/dev/nvme0n3: UUID="eCLc1F-8dPn-9S1N-Tlc8-IEEw-T0Xa-G2QojM" TYPE="LVM2_member"
/dev/sr0: BLOCK_SIZE="2048" UUID="2024-05-27-14-12-55-00" LABEL="Rocky-8-10-x86_64-dvd" TYPE="iso9660" PTUUID="add8671b" PTTYPE="dos"
/dev/mapper/rl-root: UUID="15550a72-0202-4b95-8445-637d4d45c054" BLOCK_SIZE="512" TYPE="xfs"
/dev/mapper/rl-swap: UUID="6ffaca09-d26a-4ad5-a6d1-fc1913ac4b57" TYPE="swap"
/dev/mapper/my_vg-my_lv: UUID="2c4cf7f9-f358-4803-b4aa-77f32a404035" BLOCK_SIZE="512" TYPE="xfs"
```
* PTUUID : Partition Table UUID로, 디스크의 파티션 자체에 할당된 UUID
* PTTYPE : 디스크 전체의 파티션 테이블의 유형.  <-> TYPE : 해당 파티션의 파일 시스템 포맷 유형
	* dos - MBR(Master Boot Record) 파티셔닝을 사용하는 경우
	* gpt - GPT(GUID Partition Table)를 사용하는 장치

```

#특정 디스크의 UUID 찾기
[root@localhost ~]# blkid /dev/nvme0n1p1
/dev/nvme0n1p1: UUID="48863184-18e5-426f-8dce-25ca10bf17a4" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="4d81d0d7-01"

#lsblk 사용하기
[root@localhost ~]# lsblk -o NAME,MOUNTPOINT,UUID,FSTYPE
NAME          MOUNTPOINT   UUID                                   FSTYPE
sr0                        2024-05-27-14-12-55-00                 iso9660
nvme0n1
├─nvme0n1p1   /boot        48863184-18e5-426f-8dce-25ca10bf17a4   xfs
└─nvme0n1p2                Gbjpzt-wZIv-7FdF-Rkyf-ebDM-rb4f-xjplKi LVM2_member
  ├─rl-root   /            15550a72-0202-4b95-8445-637d4d45c054   xfs
  └─rl-swap   [SWAP]       6ffaca09-d26a-4ad5-a6d1-fc1913ac4b57   swap
nvme0n2
nvme0n3                    eCLc1F-8dPn-9S1N-Tlc8-IEEw-T0Xa-G2QojM LVM2_member
└─my_vg-my_lv /mnt/my_data 2c4cf7f9-f358-4803-b4aa-77f32a404035   xfs
```


**ext4의 UUID 찾기**
`tune2fs -l /dev/sda1`


**xfs의 UUID 찾기**
```
[root@localhost etc]# xfs_admin -u /dev/mapper/my_vg-my_lv
UUID = 2c4cf7f9-f358-4803-b4aa-77f32a404035

```
#### LVM UUID
```
[root@localhost etc]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/nvme0n3
  VG Name               my_vg
  PV Size               5.00 GiB / not usable 4.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              1279
  Free PE               255
  Allocated PE          1024
  PV UUID               eCLc1F-8dPn-9S1N-Tlc8-IEEw-T0Xa-G2QojM

  --- Physical volume ---
  PV Name               /dev/nvme0n1p2
  VG Name               rl
  PV Size               <19.00 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              4863
  Free PE               0
  Allocated PE          4863
  PV UUID               Gbjpzt-wZIv-7FdF-Rkyf-ebDM-rb4f-xjplKi

[root@localhost etc]# vgdisplay
  --- Volume group ---
  VG Name               my_vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <5.00 GiB
  PE Size               4.00 MiB
  Total PE              1279
  Alloc PE / Size       1024 / 4.00 GiB
  Free  PE / Size       255 / 1020.00 MiB
  VG UUID               151qlT-vQ8O-rGk1-Sppf-WoTL-YcAD-nlylTH

  --- Volume group ---
  VG Name               rl
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <19.00 GiB
  PE Size               4.00 MiB
  Total PE              4863
  Alloc PE / Size       4863 / <19.00 GiB
  Free  PE / Size       0 / 0
  VG UUID               WjHBJy-YOJ5-d5ls-tl08-i6KS-0feU-FzeXBx

[root@localhost etc]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/my_vg/my_lv
  LV Name                my_lv
  VG Name                my_vg
  LV UUID                M3ErJk-Nbp4-Htfo-NCJz-VieM-BdB8-7RkYia
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2025-03-10 16:27:29 +0900
  LV Status              available
  # open                 1
  LV Size                4.00 GiB
  Current LE             1024
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2

  --- Logical volume ---
  LV Path                /dev/rl/swap
  LV Name                swap
  VG Name                rl
  LV UUID                ePzDTf-AiIC-bAps-CFyb-chTM-xMIK-XlyzNU
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2025-03-10 14:23:04 +0900
  LV Status              available
  # open                 2
  LV Size                2.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1

  --- Logical volume ---
  LV Path                /dev/rl/root
  LV Name                root
  VG Name                rl
  LV UUID                WANId7-SKFu-bnRf-qCNy-eHev-bV1l-N4ozhm
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2025-03-10 14:23:04 +0900
  LV Status              available
  # open                 1
  LV Size                <17.00 GiB
  Current LE             4351
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
```


> **/dev/disk/by-uuid**
>  /dev/disk/by-uuid 안에 심볼릭 링크를 만들어 두어, UUID를 이용해 해당 디스크/파티션/파일시스템에 접근할 수 있게 한다.
> 이 디렉터리에 심볼릭 링크는 UUID -> /dev/sdXn 처럼 매핑해둔 것이다.

```
[root@localhost etc]# ls -l /dev/disk/by-uuid/
total 0
lrwxrwxrwx. 1 root root 10 Mar 24 09:55 15550a72-0202-4b95-8445-637d4d45c054 -> ../../dm-0
lrwxrwxrwx. 1 root root  9 Mar 24 09:55 2024-05-27-14-12-55-00 -> ../../sr0
lrwxrwxrwx. 1 root root 10 Mar 24 09:55 2c4cf7f9-f358-4803-b4aa-77f32a404035 -> ../../dm-2
lrwxrwxrwx. 1 root root 15 Mar 24 13:38 48863184-18e5-426f-8dce-25ca10bf17a4 -> ../../nvme0n1p1
lrwxrwxrwx. 1 root root 10 Mar 24 09:55 6ffaca09-d26a-4ad5-a6d1-fc1913ac4b57 -> ../../dm-1

```

하드웨어나 디스크 순서가 바뀌면 /dev/sda가 /dev/sdb 등으로 바뀔 수 있기 때문에 UUID로 지정하여 신뢰성과 이식성을 높인다.
또한 대규모 서버나 스토리지를 다룰 때 sda, sdb 같은 이름으로는 혼동이 쉽기 때문에 UUID로 관리한다.




#### 스크립트, 구성 파일
시스템 설정, 네트워크 설정 등 자동으로 실행되거나 부팅 시 적용되는 명령어(수동 입력 필요 x)
설정 파일, 스크립트에 기록됨

**네트워크 인터페이스 설정**
`/etc/sysconfig/network-scripts/ifcfg-XXX`
```
[root@localhost ~]# cd /etc/sysconfig/network-scripts/
[root@localhost network-scripts]# ls
ifcfg-bond0  ifcfg-ens160  ifcfg-ens224

```

**부팅 관련 설정**
`/etc/fstab`
파일시스템 마운트 정보를 저장하는 파일
부팅 시점에 어떤 디스크(파티션), 네트워크 드라이브(NFS), 스왑 파티션 등을 어떤 디렉터리에 마운트할지 정의한다.
```
[root@localhost etc]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Mon Mar 10 05:23:08 2025
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rl-root     /                       xfs     defaults        0 0
UUID=48863184-18e5-426f-8dce-25ca10bf17a4 /boot                   xfs     defaults        0 0
/dev/mapper/rl-swap     none                    swap    defaults        0 0
/dev/my_vg/my_lv /mnt/my_data xfs defaults 0 0

```



`/etc/sudoers`
슈퍼유저 권한 위임 설정을 담고 있다.
```
[root@localhost etc]# cat /etc/sudoers
## Sudoers allows particular users to run various commands as
## the root user, without needing the root password.
##
## Examples are provided at the bottom of the file for collections
## of related commands, which can then be delegated out to particular
## users or groups.
##
## This file must be edited with the 'visudo' command.

## Host Aliases
## Groups of machines. You may prefer to use hostnames (perhaps using
## wildcards for entire domains) or IP addresses instead.
# Host_Alias     FILESERVERS = fs1, fs2
# Host_Alias     MAILSERVERS = smtp, smtp2

## User Aliases
## These aren't often necessary, as you can use regular groups
## (ie, from files, LDAP, NIS, etc) in this file - just use %groupname
## rather than USERALIAS
# User_Alias ADMINS = jsmith, mikem


## Command Aliases
## These are groups of related commands...

## Networking
# Cmnd_Alias NETWORKING = /sbin/route, /sbin/ifconfig, /bin/ping, /sbin/dhclient, /usr/bin/net, /sbin/iptables, /usr/bin/rfcomm, /usr/bin/wvdial, /sbin/iwconfig, /sbin/mii-tool

## Installation and management of software
# Cmnd_Alias SOFTWARE = /bin/rpm, /usr/bin/up2date, /usr/bin/yum

## Services
# Cmnd_Alias SERVICES = /sbin/service, /sbin/chkconfig, /usr/bin/systemctl start, /usr/bin/systemctl stop, /usr/bin/systemctl reload, /usr/bin/systemctl restart, /usr/bin/systemctl status, /usr/bin/systemctl enable, /usr/bin/systemctl disable

## Updating the locate database
# Cmnd_Alias LOCATE = /usr/bin/updatedb

## Storage
# Cmnd_Alias STORAGE = /sbin/fdisk, /sbin/sfdisk, /sbin/parted, /sbin/partprobe, /bin/mount, /bin/umount

## Delegating permissions
# Cmnd_Alias DELEGATING = /usr/sbin/visudo, /bin/chown, /bin/chmod, /bin/chgrp

## Processes
# Cmnd_Alias PROCESSES = /bin/nice, /bin/kill, /usr/bin/kill, /usr/bin/killall

## Drivers
# Cmnd_Alias DRIVERS = /sbin/modprobe

# Defaults specification

#
# Refuse to run if unable to disable echo on the tty.
#
Defaults   !visiblepw

#
# Preserving HOME has security implications since many programs
# use it when searching for configuration files. Note that HOME
# is already set when the the env_reset option is enabled, so
# this option is only effective for configurations where either
# env_reset is disabled or HOME is present in the env_keep list.
#
Defaults    always_set_home
Defaults    match_group_by_gid

# Prior to version 1.8.15, groups listed in sudoers that were not
# found in the system group database were passed to the group
# plugin, if any. Starting with 1.8.15, only groups of the form
# %:group are resolved via the group plugin by default.
# We enable always_query_group_plugin to restore old behavior.
# Disable this option for new behavior.
Defaults    always_query_group_plugin

Defaults    env_reset
Defaults    env_keep =  "COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS"
Defaults    env_keep += "MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE"
Defaults    env_keep += "LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES"
Defaults    env_keep += "LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE"
Defaults    env_keep += "LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY"

#
# Adding HOME to env_keep may enable a user to run unrestricted
# commands via sudo.
#
# Defaults   env_keep += "HOME"

Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin

## Next comes the main part: which users can run what software on
## which machines (the sudoers file can be shared between multiple
## systems).
## Syntax:
##
##      user    MACHINE=COMMANDS
##
## The COMMANDS section may have other options added to it.
##
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL

## Allows members of the 'sys' group to run networking, software,
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL

## Allows members of the users group to mount and unmount the
## cdrom as root
# %users  ALL=/sbin/mount /mnt/cdrom, /sbin/umount /mnt/cdrom

## Allows members of the users group to shutdown this system
# %users  localhost=/sbin/shutdown -h now

## Read drop-in files from /etc/sudoers.d (the # here does not mean a comment)
#includedir /etc/sudoers.d

```


#### 셸 초기화 파일
사용자가 자주 사용하는 명령어, 별칭(alias)
**~/.bashrc**, **~/.bash_profile**, **~/.zshrc** <= 셸 초기화 파일

#### 셸 초기화 파일
명령어 기록
`~/.bash_history`
세션이 **종료될 때**(logout 등) `.bash_history` 파일에 기록을 덮어씁니다
```
[root@localhost etc]# cat ~/.bash_history
ip addr
init 0

...(생략)

faillock
which faillock

```