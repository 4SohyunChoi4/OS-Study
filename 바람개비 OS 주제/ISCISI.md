
< 목표 >
장비 1대는 iscsi 역할을 하는 장비로 구성하고 다른 2대의 장비에서는 해당 장비의 iscsi 볼륨 연결하자

## ISCSI란
Internet Small Computer System Interface의 약자
TCP/IP 네트워크를 통해 SCSI 명령어를 전달하는 프로토콜
즉, 네트워크를 통해 ==원격의 스토리지 장치를 마치 로컬 디스크처럼 사용==할 수 있게 해준다
IP 기반의 스토리지 네트워킹 표준

### ISCSI의 특징

* 블록 수준 스토리지 접근 : SCSI 명령어를 사용하여 스토리지 장치에 접근하므로, 서버에서는 원격 저장소를 마치 로컬 디스크처럼 인식하여 사용할 수 있다. 
	* 블록 디바이스 (/dev/sdb, /dev/sdc ..) -> 물리적 위치가 내부적으로 추상화되어 OS가 구분되지 않음 <-> 파일, 디렉터리 단위 x
* ==표준 IP 네트워크 사용== : 별도의 전용 스토리지 네트워크 없이도 기존의 Ethernet 기반 네트워크를 통해 스토리지 자원을 공유할 수 있다.
* SAN(Storage Area Network) 구현 : 여러 서버가 중앙 집중식 저장소에 접근하는 SAN 환경 구축. 가상화 환경이나 백업 솔루션 등 다양한 용도로 활용된다.
* 비용 효율성 : 전용 스토리지 네트워크 장비에 비해 일반적인 네트워크 인프라를 사용하기 때문에 비용 절감 효과가 있다.


### 참고. NAS VS SAN

![[Pasted image 20250324142123.png]]

NAS 는 네트워크에 직접 연결하는 반면, SAN은 별도의 네트워크 필요
* 파이버 채널 스위치(FC switch)는 ==컴퓨터 데이터 저장소를 서버에 연결하는 네트워킹 장치==로, 스토리지 네트워크(SAN)에서 사용된다.

![[Pasted image 20250402094543.png]]
##### NAS(Network Attached Storage)
==L4 계층==(Transfer 계층, TCP/IP)
==파일 단위==로 데이터 접근 -> 서버에 직접 연결되는 게 아니라, ==네트워크를 통해 접근==
이더넷(IP 을 통해 파일을 제공하는 단일 스토리지 장치
스토리지가 다른 호스트 없이 직접 네트워크에 연결되는 방식
공유 스토리지를 네트워크 마운트 볼륨으로 제공
[[NFS, SMB]]와 같은 파일 시스템 프로토콜로 접근, 혹은 ==NAS를 **파일 서버**처럼 마운트해서 사용==



ex. FTP
<-> DAS(Direct Attached Storage)

**장점**
* 여러 다른 장치들의 데이터 저장/읽기에 용이하다
* 전용 OS를 사용하여 DAS 방식 대비 I/O 속도가 더 높다
* 스케일 아웃 방식의 NAS는 클러스터 구성이 가능하다

**단점**
* 네트워크를 사용해야 하므로 대역폭(전송 속도)에 제한이 있다
* 네트워크 병목 현상에 취약
* 보안 이슈가 있어서 DB에 사용하지 않음.
 
##### SAN(Storage Area Network)
L2 계층 (Data link 계층, Fibre Channel) or L3(Network 계층)
파일이 아닌 ==블록 단위==의 I/O 입출력을 기본으로 한다. 각 블록들은 저장된 위치(특정 스토리지 시스템의 특정 디스크)에 대한 주소를 가지고 있어 호스트들의 요청에 따라 블록을 재구성해 하나의 데이터로 조합하여 호스트에 전달된다.
여러 스토리지들을 하나의 네트워크에 연결시킨 다음 이 네트워크를 스토리지 전용 네트워크로 구성하는 방식
스토리지에 접근하기 위해서는 각 호스트들은 모두 SAN 전용 네트워크를 거쳐서 접근해야 한다.
SAN은 별도의 **SAN 전용 스위치**를 필요로 한다. 별도의 데이터 전달 통로를 통해 스토리지 시스템에 액세스 하기 때문에 일반 네트워크 소통량에 영향을 받지 않고 신속한 데이터 액세스
연결된 디스크는 **로컬 드라이브로 보인다.**
공유는 못하지만 SAN에 있는 Storage 공간을 N명에게 각각 '할당' 할 수 있다.

**장점**
* 높은 성능, 낮은 지연 시간
* 중앙 집중식 스토리지 관리
* 유연한 스토리지 할당
* 고가용성 및 데이터 보호

**단점**
* 초기 투자 비용
* 복잡한 관리 및 구성
* 전용 네트워크 필요
* 확장성에 따른 관리 부담


### 초기 구축

![[Pasted image 20250324165855.png]]
![[Pasted image 20250324165908.png]]

### iSCSI Target 구성

#### `targetcli` 설치

LIO(iSCSI target) 관리를 위한 `targetcli` 설치
```
[root@localhost ~]# dnf install targetcli
Last metadata expiration check: 2:33:50 ago on Mon 24 Mar 2025 02:33:41 PM KST.
Dependencies resolved.
....(생략)
Complete!
```

디렉터리 생성 후 `targetcli` 실행
```
[root@localhost ~]# mkdir -p /var/iscsi_disk

[root@localhost ~]# targetcli
targetcli shell version 2.1.53
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.
```

**targetcli saveconfig**
`targetcli` 명령어에서 설정을 저장한다.

**targetcli saveconfig -autosave**
자동으로 설정을 저장할 수 있도록 설정하는 옵션
시스템 재부팅, targetcli 변경사항 다 반영됨.
targetcli 8버전부터 생겼다.




#### ==파일 기반 backing store 생성==
10GB 크기의 이미지 파일을 /var/iscsi_disks 폴더에 생성
```
/> /backstores/fileio create name=mydisk file_or_dev=/var/iscsi_disks/mydisk.img size=10G
Created fileio mydisk with size 10737418240
```

- **Backstores** : iSCSI 서비스로 제공되는 스토리지 리소스. 스토리지 종류 설정 (iSCSI target의 저장 영역을 어떤 것으로 구성할지?)
    - block : 실제 블록 디바이스를 저장 영역으로
        - 생성 : `backstores/block create name=[이름] dev=[디스크 경로]`
    - fileio : 파일 시스템 중 일부 공간  
        - 생성 : `backstores/fileio create name=[이름] file_or_dev=[생성할 파일 경로] size=[할당할 사이즈]`
        - 예시 : `backstores/fileio create name=file1 file_or_dev=/iscsi size=100M`
    - pscsi : 실제 SCSI 장치 (물리적인 SCSI 장치)
    - ramdisk : RAM 공간의 일부

=> 테스트할 때는 주로 block, fileio 위주로 사용한다.

* name : 이름
* file_or_dev / dev : 파일/디스크 경로
* write_io= ISCSI Target의 IO 방식
	* direct : direct I/O 모드 - 직접 디스크에 쓰기
	* wb : Write-Back 모드 - OS 캐시를 사용, 나중에 디스크에 저장(데이터 유실 위험)
	* wt : Write-Through - 캐시 + 디스크에 데이터 저장 동시에


#### iSCSI target 생성
target 이름은 IQN 형식으로 지정한다.
IQN은 주로 “년-월.도메인:식별자” 형식으로 지정한다.

> IQN : iSCSI Qualified Name
> iSCSI에서 사용하는 식별자
> 도메인 이름과 소유 시점을 기반으로 한다.
> -> **가독성**을 위해 사용된다.


```
/> /iscsi create iqn.2025-03.com.baramgebi:target1
Created target iqn.2025-03.com.baramgebi:target1.
Created TPG 1.
Global pref auto_add_default_portal=true
Created default portal listening on all IPs (0.0.0.0), port 3260.
```


#### LUN 생성
백킹 스토어를 target에 LUN으로 연결한다.

> **LUN(Logical Unit Number)**
> 스토리지에서 논리적인 디스크 장치를 식별하기 위한 고유 번호
> >>대용량 스토리지를 논리적으로 잘라내서 구별해서 사용하기 위해 사용한다.
> 서버가 LUN을 하나의 디스크처럼 인식하고, 필요한 파일 시스템을 올리거나 데이터를 저장·읽을 수 있다

```
/> /iscsi/iqn.2025-03.com.baramgebi:target1/tpg1/luns create /backstores/fileio/mydisk
Created LUN 0.
```

#### ACL(접근 제어) 설정
initiator에서 사용할 IQN을 허용한다.

> **ACL(Access Control List)**
>  iSCSI Target에 대한 접근 권한을 제어하기 위한 목록
>  어떤 IQN(혹은 IP)에서 오는 연결만 허용할지’를 미리 정의해두면, ACL을 통과한 Initiator만이 블록 디바이스를 정상적으로 마운트하고 사용할 수 있다. 
>  =>무단 접근 방지
>  => 보안 강화
```
/> /iscsi/iqn.2025-03.com.baramgebi:target1/tpg1/acls create iqn.2025-03.com.baramgebi:initiator1
Created Node ACL for iqn.2025-03.com.baramgebi:initiator1
Created mapped LUN 0.
/> exit

Global pref auto_save_on_exit=true
Configuration saved to /etc/target/saveconfig.json
```

```
/iscsi/iqn.20...et1/tpg1/acls> create iqn.2025-03.com.baramgebi:initiator2
Created Node ACL for iqn.2025-03.com.baramgebi:initiator2
Created mapped LUN 0.
```

iSCSI 포트 3260이 열려 있는지 확인하고 
```
[root@localhost ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: bond0 ens160 ens224
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[root@localhost ~]# firewall-cmd --permanent --zone=public --add-port=3260/tcp
success
[root@localhost ~]# firewall-cmd --reload
success

[root@localhost ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: bond0 ens160 ens224
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 3260/tcp
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

target 서비스 활성화, 부팅시 자동 시작 되도록 설정
```
[root@localhost ~]# systemctl enable target
Created symlink /etc/systemd/system/multi-user.target.wants/target.service → /usr/lib/systemd/system/target.service.
[root@localhost ~]# systemctl start target
```


```
/iscsi/iqn.20...bi:initiator1> ls
o- iqn.2025-03.com.baramgebi:initiator1  [Mapped LUNs: 1]
  o- mapped_lun0 .............. [lun0 fileio/mydisk (rw)]
/iscsi/iqn.20...bi:initiator1> cd /
/> ls
o- / .............................................. [...]
  o- backstores ................................... [...]
  | o- block ....................... [Storage Objects: 0]
  | o- fileio ...................... [Storage Objects: 1]
  | | o- mydisk  [/var/iscsi_disks/mydisk.img (10.0GiB) write-back activated]
  | |   o- alua ........................ [ALUA Groups: 1]
  | |     o- default_tg_pt_gp  [ALUA state: Active/optimized]
  | o- pscsi ....................... [Storage Objects: 0]
  | o- ramdisk ..................... [Storage Objects: 0]
  o- iscsi ................................. [Targets: 1]
  | o- iqn.2025-03.com.baramgebi:target1 ...... [TPGs: 1]
  |   o- tpg1 .................... [no-gen-acls, no-auth]
  |     o- acls ............................... [ACLs: 2]
  |     | o- iqn.2025-03.com.baramgebi:initiator1  [Mapped LUNs: 1]
  |     | | o- mapped_lun0 .... [lun0 fileio/mydisk (rw)]
  |     | o- iqn.2025-03.com.baramgebi:initiator2  [Mapped LUNs: 1]
  |     |   o- mapped_lun0 .... [lun0 fileio/mydisk (rw)]
  |     o- luns ............................... [LUNs: 1]
  |     | o- lun0  [fileio/mydisk (/var/iscsi_disks/mydisk.img) (default_tg_pt_gp)]
  |     o- portals ......................... [Portals: 1]
  |       o- 0.0.0.0:3260 .......................... [OK]
  o- loopback .............................. [Targets: 0]
```

- **iqn.2025-03.com.baramgebi:initiator1**
- **iqn.2025-03.com.baramgebi:initiator2**
두 개의 acl에 대해 LUN이 매핑되어 있음.

### iSCSI initiator(클라이언트) 구성

#### `open-iscsi` 설치

```
[root@localhost ~]# dnf install iscsi-initiator-utils
Rocky Linux 8 - App 1.6 MB/s |  17 MB     00:10
Rocky Linux 8 - Bas 1.8 MB/s |  19 MB     00:10
Rocky Linux 8 - Ext  16 kB/s |  15 kB     00:00
Dependencies resolved.
(생략)
  iscsi-initiator-utils-6.2.1.4-8.git095f59c.el8_8.x86_64
  iscsi-initiator-utils-iscsiuio-6.2.1.4-8.git095f59c.el8_8.x86_64
  isns-utils-libs-0.99-1.el8.x86_64

Complete!
```

#### initiator 이름 설정
클라이언트 1, 2에 각각 이름 설정
*CHAP 인증은 잘 사용하지 않는다.

```
[root@localhost ~]# vi /etc/iscsi/initiatorname.iscsi

InitiatorName=iqn.2025-03.com.baramgebi:initiator1 # 1번 node

InitiatorName=iqn.2025-03.com.baramgebi:initiator2 # 2번 node
```

#### target 검색(discovery)

target 서버에서 제공하는 iSCSI target 목록을 보여준다.
```
[root@localhost ~]# iscsiadm -m discovery -t sendtargets -p 192.168.38.128
192.168.38.128:3260,1 iqn.2025-03.com.baramgebi:target1
```

#### initiator2에서 target 로그인
```
[root@localhost ~]# sudo iscsiadm -m node -T iqn.2025-03.com.baramgebi:target1 -p 192.168.38.128 --login
Logging in to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260]
Login to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260] successful.
```

`-l` 로 자동으로 다 로그인 가능한데, 테스트에서만 하고 실제로는 필요 없는 데까지 다 로그인 되니까 권장하진 않는다.

####  initiator2에서 세션 확인

```
[root@localhost ~]# iscsiadm -m session
tcp: [9] 192.168.38.128:3260,1 iqn.2025-03.com.baramgebi:target1 (non-flash)
```

#### initiator1에서 target 로그인
오류가 발생했다.
처음에 이름에 오타가 있어서 `/etc/iscsi/initiatorname.iscsi`에서 이름을 정확히 변경하였다.

```
[root@localhost ~]# vi /etc/iscsi/initiatorname.iscsi
[root@localhost ~]# cat /etc/iscsi/initiatorname.iscsi   InitiatorName=iqn.2025-03.com.baramgebi:initiator1

[root@localhost ~]# iscsiadm -m discovery -t sendtargets -p 192.168.38.128
192.168.38.128:3260,1 iqn.2025-03.com.baramgebi:target1
[root@localhost ~]# iscsiadm -m node -T iqn.2025-03.com.baramgebi:target1 -p 192.168.38.128 --login
Logging in to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260]
iscsiadm: Could not login to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260].
iscsiadm: initiator reported error (24 - iSCSI login failed due to authorization failure)
iscsiadm: Could not log into all portals
```

로그인 실패 , 로그아웃 시도
```
[root@localhost ~]#  iscsiadm -m node --logout
iscsiadm: No matching sessions found
```

iscsid 서비스 재시작

```
[root@localhost ~]# sudo systemctl restart iscsid
```

로그인 다시 시도
```
[root@localhost ~]# iscsiadm -m discovery -t sendtargets -p 192.168.38.128
192.168.38.128:3260,1 iqn.2025-03.com.baramgebi:target1
[root@localhost ~]# iscsiadm -m node -T iqn.2025-03.com.baramgebi:target1 -p 192.168.38.128 --login
Logging in to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260]
Login to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260] successful.
```

성공하였다.

```
[root@localhost ~]# iscsiadm -m discovery -t sendtargets -p 192.168.38.128
192.168.38.128:3260,1 iqn.2025-03.com.baramgebi:target1
[root@localhost ~]# iscsiadm -m node --login
Logging in to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260]
iscsiadm: Could not login to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260].
iscsiadm: initiator reported error (24 - iSCSI login failed due to authorization failure)
iscsiadm: Could not log into all portals

```
6주차 또 2번 클라이언트에서 로그인 안되는 문제가 발생하였음.

```
[root@localhost cron.d]# targetcli ls
o- / ............................................................................................................ [...]
  o- backstores ................................................................................................. [...]
  | o- block ..................................................................................... [Storage Objects: 0]
  | o- fileio .................................................................................... [Storage Objects: 1]
  | | o- mydisk .......................................... [/var/iscsi_disks/mydisk.img (10.0GiB) write-back activated]
  | |   o- alua ...................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp .......................................................... [ALUA state: Active/optimized]
  | o- pscsi ..................................................................................... [Storage Objects: 0]
  | o- ramdisk ................................................................................... [Storage Objects: 0]
  o- iscsi ............................................................................................... [Targets: 1]
  | o- iqn.2025-03.com.baramgebi:target1 .................................................................... [TPGs: 1]
  |   o- tpg1 .................................................................................. [no-gen-acls, no-auth]
  |     o- acls ............................................................................................. [ACLs: 1]
  |     | o- iqn.2025-03.com.baramgebi:initiator1 .................................................... [Mapped LUNs: 1]
  |     |   o- mapped_lun0 .................................................................. [lun0 fileio/mydisk (rw)]
  |     o- luns ............................................................................................. [LUNs: 1]
  |     | o- lun0 .................................... [fileio/mydisk (/var/iscsi_disks/mydisk.img) (default_tg_pt_gp)]
  |     o- portals ....................................................................................... [Portals: 1]
  |       o- 0.0.0.0:3260 ........................................................................................ [OK]
  o- loopback ............................................................................................ [Targets: 0]
```

initiator2에 대한 acl이 누락되어 있었다.

```
/> /iscsi/iqn.2025-03.com.baramgebi:target1/tpg1/acls create iqn.2025-03.com.baramgebi:initiator2
Created Node ACL for iqn.2025-03.com.baramgebi:initiator2
Created mapped LUN 0.

```

```
[root@localhost ~]# iscsiadm -m node --login
Logging in to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260]
Login to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260] successful.

```
다시 로그인 잘 됨

## ==서비스 iscsi, iscsid==

#### iscsi
클라이언트 역할을 하는 iSCSI Initiator를 관리하는 서비스
iSCSI 클라이언트(Initiator)가 iSCSI 서버(Target)과 연결할 때 사용됨
`open-iscsi` 패키지로 설치된다.
내부적으로 `iscsid`(데몬)와 `iscsiadm`(CLI) 도구를 사용
부팅 시 자동으로 iSCSI 세션 복원됨.
```
[root@localhost ~]# systemctl status iscsi
● iscsi.service - Login and scanning of iSCSI devices
   Loaded: loaded (/usr/lib/systemd/system/iscsi.service; enabled; vendor preset: disabled)
   Active: active (exited) since Mon 2025-03-31 21:25:45 EDT; 24h ago
     Docs: man:iscsiadm(8)
           man:iscsid(8)
  Process: 1005 ExecStart=/usr/sbin/iscsiadm -m node --loginall=automatic (code=exited, status=0/SUCCESS)
 Main PID: 1005 (code=exited, status=0/SUCCESS)
    Tasks: 0 (limit: 4477)
   Memory: 0B
   CGroup: /system.slice/iscsi.service

Mar 31 21:25:43 localhost.localdomain systemd[1]: Starting Login and scanning of iSCSI devices...
Mar 31 21:25:45 localhost.localdomain iscsiadm[1005]: Logging in to [iface: default, target: iqn.2025-03.com.baramgebi>
Mar 31 21:25:45 localhost.localdomain iscsiadm[1005]: Login to [iface: default, target: iqn.2025-03.com.baramgebi:targ>
Mar 31 21:25:45 localhost.localdomain systemd[1]: Started Login and scanning of iSCSI devices.

```

#### iscsid
iscsi daemon, 즉 백그라운드 프로세스이다.
`iscsid` 데몬이 실행되어 있어야 iSCSI Initiator가 정상적으로 동작
iSCSI 연결을 유지하고 관리하는 핵심 프로세스
==`iscsi` 서비스가 실행될 때 자동으로 `iscsid`도 실행됨==
```
[root@localhost ~]# systemctl status iscsid
● iscsid.service - Open-iSCSI
   Loaded: loaded (/usr/lib/systemd/system/iscsid.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2025-03-31 21:25:44 EDT; 24h ago
     Docs: man:iscsid(8)
           man:iscsiuio(8)
           man:iscsiadm(8)
 Main PID: 1011 (iscsid)
   Status: "Ready to process requests"
    Tasks: 1 (limit: 4477)
   Memory: 12.7M
   CGroup: /system.slice/iscsid.service
           └─1011 /usr/sbin/iscsid -f

Mar 31 21:25:43 localhost.localdomain systemd[1]: Starting Open-iSCSI...
Mar 31 21:25:44 localhost.localdomain systemd[1]: Started Open-iSCSI.
Mar 31 21:25:45 localhost.localdomain iscsid[1011]: iscsid: Could not set session1 priority. READ/WRITE throughout and>
Mar 31 21:25:45 localhost.localdomain iscsid[1011]: iscsid: Connection1:0 to [target: iqn.2025-03.com.baramgebi:target>
```


## iscsiadm
iSCSI 대상을 검색하고 로그인할 수 있게 해주는 명령줄 도구

주요 옵션(많음)
* -m discovery : discovery 모드를 통해 iSCSI LUN 검색
* -p : IP 주소 지정
	* exe. `iscsiadm -m discovery -t sendtargets -p 192.168.38.128`
* -l : --login : 지정된 레코드에 로그인
* -L : Login All, 모든 세션에 로그인

```
[root@localhost ~]#  iscsiadm -m node --logout
Logging out of session [sid: 1, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260]
Logout of [sid: 1, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260] successful.
[root@localhost ~]# iscsiadm -m discovery -t sendtargets -p 192.168.38.128
192.168.38.128:3260,1 iqn.2025-03.com.baramgebi:target1

[root@localhost ~]# iscsiadm -m node -l
Logging in to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260]
Login to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260] successful.

```

## mount 공유 확인해보기

initiator1에서 먼저 진행하자.

```
[root@localhost ~]# iscsiadm -m session
tcp: [2] 192.168.38.128:3260,1 iqn.2025-03.com.baramgebi:target1 (non-flash)
```
세션 확인한다.

```
[root@localhost ~]# iscsiadm -m node -T iqn.2025-03.com.baramgebi:target1 -p 192.168.38.128 --op update -n node.startup -v automatic
```
iSCSI 자동 연결 설정한다. 
재부팅 시 자동 접속된다.


```
[root@localhost ~]# ls -l /dev/disk/by-path/
total 0
lrwxrwxrwx. 1 root root  9 Apr  1 23:18 ip-192.168.38.128:3260-iscsi-iqn.2025-03.com.baramgebi:target1-lun-0 -> ../../sda
lrwxrwxrwx. 1 root root 10 Apr  1 23:18 ip-192.168.38.128:3260-iscsi-iqn.2025-03.com.baramgebi:target1-lun-0-part1 -> ../../sda1
lrwxrwxrwx. 1 root root  9 Mar 31 21:25 pci-0000:00:07.1-ata-2 -> ../../sr0
lrwxrwxrwx. 1 root root 13 Mar 31 21:25 pci-0000:0b:00.0-nvme-1 -> ../../nvme0n1
lrwxrwxrwx. 1 root root 15 Mar 31 21:25 pci-0000:0b:00.0-nvme-1-part1 -> ../../nvme0n1p1
lrwxrwxrwx. 1 root root 15 Mar 31 21:25 pci-0000:0b:00.0-nvme-1-part2 -> ../../nvme0n1p2

```
LUN 을 확인해서 iSCSI LUN으로 인식된 것이 sda인 것을 확인할 수 있다

```
[root@localhost ~]# fdisk -l
Disk /dev/nvme0n1: 5 GiB, 5368709120 bytes, 10485760 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa85e367c

Device         Boot   Start      End Sectors Size Id Type
/dev/nvme0n1p1 *       2048  2099199 2097152   1G 83 Linux
/dev/nvme0n1p2      2099200 10485759 8386560   4G 8e Linux LVM


Disk /dev/mapper/rl-root: 3.5 GiB, 3753902080 bytes, 7331840 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/rl-swap: 512 MiB, 536870912 bytes, 1048576 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sda: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 8388608 bytes

```
`fdisk -l`로 확인

```
[root@localhost ~]# fdisk /dev/sda

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xdd1eb92a.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (16384-20971519, default 16384):
Last sector, +sectors or +size{K,M,G,T,P} (16384-20971519, default 20971519):

Created a new partition 1 of type 'Linux' and of size 10 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

sda의 파티션을 쪼갠다. 
sda1을 사용한다.

```
[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0   10G  0 disk
└─sda1        8:1    0   10G  0 part /mnt/iscsitest
sr0          11:0    1  2.5G  0 rom
nvme0n1     259:0    0    5G  0 disk
├─nvme0n1p1 259:1    0    1G  0 part /boot
└─nvme0n1p2 259:2    0    4G  0 part
  ├─rl-root 253:0    0  3.5G  0 lvm  /
  └─rl-swap 253:1    0  512M  0 lvm  [SWAP]

```


```
[root@localhost ~]# mkfs.ext4 /dev/sda1
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 2619392 4k blocks and 655360 inodes
Filesystem UUID: d81d3881-9f11-4980-aa18-10ab82b2a5a6
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

```
파일 시스템을 ext.4로 생성한다.

```
[root@localhost ~]# mkdir -p /mnt/iscsitest
[root@localhost ~]# mount /dev/sda1 /mnt/iscsitest/
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
[root@localhost ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             350M     0  350M   0% /dev
tmpfs                370M     0  370M   0% /dev/shm
tmpfs                370M  5.3M  364M   2% /run
tmpfs                370M     0  370M   0% /sys/fs/cgroup
/dev/mapper/rl-root  3.5G  2.2G  1.4G  61% /
/dev/nvme0n1p1      1014M  199M  816M  20% /boot
tmpfs                 74M     0   74M   0% /run/user/0
/dev/sda1            9.8G   24K  9.3G   1% /mnt/iscsitest
```

iscsitest 라는 디렉토리를 만들어 준 뒤, 마운트한다.

```
[root@localhost ~]# blkid /dev/sda1
/dev/sda1: UUID="d81d3881-9f11-4980-aa18-10ab82b2a5a6" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="dd1eb92a-01"
[root@localhost ~]# vi /etc/fstab
```

```
#
# /etc/fstab
# Created by anaconda on Mon Mar 24 05:53:43 2025
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rl-root     /                       xfs     defaults        0 0
UUID=e1a54b64-616c-42f3-a273-6f96d804fbcc /boot                   xfs     defaults        0 0
/dev/mapper/rl-swap     none                    swap    defaults        0 0
UUID=d81d3881-9f11-4980-aa18-10ab82b2a5a6 /mnt/newdisk ext4 defaults,_netdev 0 0

```

`/etc/fstab` 에서 추가하여 재부팅시 자동으로 마운트 되도록 한다.

* _ netdev란?
[[LVM 설정]] - `/etc/fsatab`의  '_ netdev 설정' 참고

```
[root@localhost ~]# echo "hello world" >> /mnt/iscsitest/test.txt
[root@localhost ~]# cat /mnt/iscsitest/test.txt
```


initiator2에서 확인해보자.

```
[root@localhost ~]# iscsiadm -m discovery -t sendtargets -p 192.168.38.128
192.168.38.128:3260,1 iqn.2025-03.com.baramgebi:target1
[root@localhost ~]# iscsiadm -m node --login
Logging in to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260]
Login to [iface: default, target: iqn.2025-03.com.baramgebi:target1, portal: 192.168.38.128,3260] successful.
[root@localhost ~]# lsblkfdisk -l
-bash: lsblkfdisk: command not found
[root@localhost ~]# fdisk -l
Disk /dev/nvme0n1: 5 GiB, 5368709120 bytes, 10485760 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa85e367c

Device         Boot   Start      End Sectors Size Id Type
/dev/nvme0n1p1 *       2048  2099199 2097152   1G 83 Linux
/dev/nvme0n1p2      2099200 10485759 8386560   4G 8e Linux LVM


Disk /dev/mapper/rl-root: 3.5 GiB, 3753902080 bytes, 7331840 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/rl-swap: 512 MiB, 536870912 bytes, 1048576 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sda: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 8388608 bytes
Disklabel type: dos
Disk identifier: 0xdd1eb92a

Device     Boot Start      End  Sectors Size Id Type
/dev/sda1       16384 20971519 20955136  10G 83 Linux

```
sda가 sda1로 파티션 되어 있는 것도 확인 가능하다.

```
[root@localhost ~]# blkid /dev/sda1
/dev/sda1: UUID="d81d3881-9f11-4980-aa18-10ab82b2a5a6" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="dd1eb92a-01"

```
파일 시스템 유형이 `ext4`로 표시되며, UUID도 같다.
즉, ==iSCSI가 제대로 마운트 된 것이다.==

```
[root@localhost ~]# mkdir -p /mnt/iscsi_test
[root@localhost ~]# mount /dev/sda1 /mnt/iscsi_test/
[root@localhost ~]# df -hT | grep sda1
/dev/sda1           ext4      9.8G   28K  9.3G   1% /mnt/iscsi_test

```
sda1 마운트

```
[root@localhost ~]# df -hT | grep sda1
/dev/sda1           ext4      9.8G   28K  9.3G   1% /mnt/iscsi_test
[root@localhost ~]# ls /mnt/iscsi_test/
lost+found  test.txt
[root@localhost ~]# cat /mnt/iscsi_test/test.txt
hello world

```

잘보인다

