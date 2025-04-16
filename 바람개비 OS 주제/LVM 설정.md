
- OS 설치 영역 볼륨 외 2개 이상의 볼륨을 할당하고 1개 볼륨 LVM 설정 후 증설
- LVM 관련된 설정 위치와 해당 내용


#### LVM (Logical Volume Manager)

Linux의 저장 공간을 효율적이고 유연하게 관리하기 위한 Kernel의 한 종류
Storage 확장/변경에 유연하고, 크기를 변경할 때, 기존 Data의 이전이 필요 없다.

파일 시스템이 **블록** 장치에 직접 접근해서 읽고 쓰는 것이 아니라, LVM이 만든 **가상의 블록 장치**
(==볼륨)==에 읽기 쓰기를 진행한다.

> [!NOTE]
>  **Disk Partitioning과의 차이점** 
>  기존 방식의 경우 HDD를 Partitioning한 다음에 OS 영역에 Mount하여 R/W 해주는 방식인데, LVM은 Partitioning 대신 Volume이라는 단위로 저장 장치를 다루는 방식 
이 경우 저장 공간의 크기가 고정되어서 ==증설/축소가 어렵다==.

![[Pasted image 20250226112047.png]]

![[Pasted image 20250226103159.png]]
![[Pasted image 20250226103300.png]]
**물리적 볼륨 / PV (Physical Volume)**  
- 실제 디스크 장치를 분할한 파티션된 상태를의미한다.  
- PV는 일정한 크기의 PE들로 구성된다.  

**물리적 확장 / PE (Physical Extent)**  
- PV를 구성하는 일정한 크기의 Block.  
- 보통 1PE는 4MB에 해당한다.  
- PE와 LE는 1:1로 대응한다.  

**볼륨 그룹 / VG (Volume Group)**  
- PV들이 모여서 생성되는 단위이다. (모든걸 합친 거대한 지점토 덩어리의 느낌이다)  
- 사용자는 VG를 원하는대로 쪼개서 LV로 만들게 된다.  

**논리적 볼륨 / LV (Logical Volume)**  
- 사용자가 최종적으로 사용하는 단위로, VG에서 필요한 크기로 할당 받아 LV를 생성한다.

### =디스크 추가하기=

![[Pasted image 20250306140443.png]]
![[Pasted image 20250310150850.png]]

![[Pasted image 20250306140517.png]]

![[Pasted image 20250310150911.png]]

한 개는 10G, 나머지 한 개는 5G로 설정


![[Pasted image 20250306140543.png]]

##### 1. 가상 디스크 생성

```
[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0          11:0    1  2.5G  0 rom
nvme0n1     259:0    0   20G  0 disk
├─nvme0n1p1 259:1    0    1G  0 part /boot
└─nvme0n1p2 259:2    0   19G  0 part
  ├─rl-root 253:0    0   17G  0 lvm  /
  └─rl-swap 253:1    0    2G  0 lvm  [SWAP]
nvme0n2     259:3    0   10G  0 disk
nvme0n3     259:4    0    5G  0 disk


[root@localhost ~]# fdisk -l
Disk /dev/nvme0n1: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x4d81d0d7

Device         Boot   Start      End  Sectors Size Id Type
/dev/nvme0n1p1 *       2048  2099199  2097152   1G 83 Linux
/dev/nvme0n1p2      2099200 41943039 39843840  19G 8e Linux LVM

Disk /dev/nvme0n2: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/nvme0n3: 5 GiB, 5368709120 bytes, 10485760 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/rl-root: 17 GiB, 18249416704 bytes, 35643392 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/rl-swap: 2 GiB, 2147483648 bytes, 4194304 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

```

> [!NOTE]
> 
> > **lvcreate ,lvextend -L l 차이?**
> > 
> > **-L : 고정 크기로 지정**
> > ex) `lvcreate -L 5G -n my_lv my_vg
> > 정확히 5G의 논리 볼륨 할당
> > 
> > 
> > **-l : 특정 비율만큼 할당**
> > PE(Physical Extents) 단위를 기준으로 크기 설정
> > 볼륨 그룹의 전체 크기나 남은 공간의 비율로 크기를 설정
> > 유연하게 할당
> >
>    ex) `lvextend -l +100%FREE /dev/my_vg/my_lv`
> > VG 남은 용량 100% 할당
> > ex) `lvextend -l +50%VG /dev/my_vg/my_lv`
> > 볼륨 그룹의 전체 크기의 50%를 논리 볼륨에 할당
>  l- 하고 nn% 다음에 VG, LV, PVS, 등등..

#### ~~가상디스크 생성해서 진행~~

![[Pasted image 20250226015043.png]]
![[Pasted image 20250226112639.png]]
sdb가 없어 loopback device(가상 디바이스)로 설정하였다.

![[Pasted image 20250226112716.png]]
10G 가상 디스크 생성 및 loop1을 가상 디스크로 설정



loop는 급하게 필요할 때 아니면 실제로 잘 안쓴다.

#### 2.  LVM 초기화 
```
[root@localhost ~]# pvcreate /dev/nvme0n3
  Physical volume "/dev/nvme0n3" successfully created.

```
 Physical Volume 생성


```
[root@localhost ~]# vgcreate my_vg /dev/nvme0n3
  Volume group "my_vg" successfully created

```
 Volume Group 생성

```
[root@localhost ~]# lvcreate -L 3G -n my_lv my_vg
  Logical volume "my_lv" created.

```

Logical Volume 생성
```
[root@localhost ~]# mkfs.xfs /dev/my_vg/my_lv
meta-data=/dev/my_vg/my_lv       isize=512    agcount=4, agsize=196608 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=786432, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

```

XFS 방식의 파일 시스템 생성 -> mxfs.xfs
EXT4 사용하려면 -> mkfs.ext4

>`mkfs.xfs`
>`mkfs -t xfs`(구버전)
>
>두 가지 다 사용할 수 있다.


> [!NOTE] XFS 파일 시스템
> 고성능 저널링 파일 시스템
> **대용량 데이터 처리**와 **빠른 파일 시스템 확장**이 필요한 환경에서 사용된다.
> sudo mkfs.xfs /dev/my_vg/my_lv



```
[root@localhost ~]# mkdir /mnt/my_data

[root@localhost ~]# mount /dev/my_vg/my_lv /mnt/my_data

[root@localhost ~]# df -h | grep my_data
/dev/mapper/my_vg-my_lv  3.0G   54M  3.0G   2% /mnt/my_data


```

마운트 확인

```

[root@localhost ~]# echo "/dev/my_vg/my_lv /mnt/my_data xfs defaults 0 0" | sudo tee -a /etc/fstab
/dev/my_vg/my_lv /mnt/extra_data xfs defaults 0 0

```
자동 마운트 설정
/etc/fstab에 설정하면 자동 마운트 가능

-> echo로 지정하는 것보다는 `vi`를 이용해서 직접 수정하는 게 낫다.



```
[root@localhost ~]# mount -a
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
[root@localhost ~]# df -h | grep my_data
/dev/mapper/my_vg-my_lv  3.0G   54M  3.0G   2% /mnt/my_data

```
`mount -a`는 /etc/fstab에 있는 마운트 항목을 적용하는 명령어


```
[root@localhost ~]# umount /mnt/my_data
[root@localhost ~]# df -h | grep my_data

```

마운트 테스트는 `umount`로

-> 이것보다는 `mount -a`해서 적용되는지 여부를 확인하는 게 빠르다.

#### mount -t
-t : type
파일시스템 유형을 지정할 때 사용한다.
특정 파일 시스템 타입을 명시적으로 지정 가능
대부분의 경우 리눅스가 자동으로 감지 but 특정 파일 시스템을 강제 지정할 때 유용


#### /etc/fstab의 내용
```
[root@localhost ~]# cat /etc/fstab

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
/dev/mapper/rl-root     /                       xfs     defaults        0 0                                              <---1번
UUID=48863184-18e5-426f-8dce-25ca10bf17a4 /boot                   xfs     defaults        0 0                            <-- 2번
/dev/mapper/rl-swap     none                    swap    defaults        0 0                                              <-- 3번
/dev/my_vg/my_lv /mnt/my_data xfs defaults 0 0    <-- 4번

```
###### 각각 파일 시스템/마운트 위치/파일 시스템 타입/마운트 옵션/dump/fsck


ex) 
* **파일 시스템** : /dev/mapper/rl-root
LVM 기반의 루트 파일 시스템

* **마운트 위치** : /
루트 디렉터리

* **파일 시스템 타입** : xfs
xfs 파일 시스템을 이용한다.

* **마운트 옵션** : defaults
일반적인 기본 마운트 옵션
==** _ netdev :== 
/etc/fstab에 **네트워크 드라이브, 저장소를 자동 mount하게 설정하는 경우** 시스템 재시작 시 마운트를 못해서 대기 발생하는 경우 있다.
이는 시스템이 ==네트워크를 설정하고 연결하기 전에 /etc/fstab의 mount를 시도하기 때문==.
`defaults,_netdev` 로 설정이 필요하다.
ex. `192.168.1.1:/usr/locl /mnt nfs defaults,_netdev 0 0`


- **dump** : 0
dump 명령어로 백업할 지,  0- 안한다/1 - 한다
대부분 현대 리눅스에선 거의 x
`dump -0 -f backup.img /dev/sda1`

- **fsck** : 0
부팅 시 fsck 파일 시스템 체크 우선순위,  0- 안한다/1 - 루트 / 2- 기타
스왑에는 fsck 필요 x




##### fsck
파일 시스템 체크
부팅 시 자동으로 디스크를 검사하고 오류를 복구하는 역할
2번은 루트 파일 시스템 외의 다른 파일 시스템을 검사한다. ex. /home, /boot

1, 2로 하면 우선순위가 높아진다. 무거운 시스템의 우선순위를 높이면 부팅 속도가 느려지게 된다.
중요한 시스템이 있을 땐 먼저 봐야하기 때문에 부팅 순서를 높이기도 한다.

##### 마운트 옵션
mount 명령어의 옵션을 통해 파일 시스템이 어떻게 동작할 지 정의

|           |                                        |             |                                      |
| --------- | -------------------------------------- | ----------- | ------------------------------------ |
| `rw`      | 읽기/쓰기(Read/Write) 허용 /                 | `-o ro`     | 읽기 전용                                |
| `suid`    | `setuid`, `setgid` 비트 사용 허용 (권한 승격 가능) | `-o nosuid` | `setuid`, `setgid` 비트를 무시, 권한 상승 x   |
| `dev`     | 장치 파일(`/dev/sda1` 같은 디바이스 노드) 사용 가능    | `-o nodev`  | 디바이스 파일 무시, 보안 강화                    |
| `-o exec` | 실행 파일 실행 가능 (`chmod +x`)               | `-o noexec` | ./script.sh 류의 파일 실행 x               |
| `auto`    | 부팅 시 자동 마운트 (`mount -a`실행 시 자동 마운트됨)   | `noauto`    | 부팅 시 자동 마운트 x                        |
| `nouser`  | 일반 사용자가 마운트 x , root만                  | `-o user`   | 일반 사용자도 가능, `users`시 다른 사용자도 언마운트 가능 |
| `async`   | 비동기 I/O (쓰기 작업이 즉시 디스크에 반영 x, )        | `-o sync`   | 성능 저하                                |

#### Swap
메모리의 부족을 보조하는 가상 메모리 역할
RAM(- 물리적인 메모리)이 부족할 때 긴급하게 사용됨 -> 프로세스 강종 x, 과도하게 사용 시 시스템이 느려진다.

사용되지 않는 낮은 우선순위의 데이터를 스왑 공간으로 이동
새로운 프로세스가 실행되면 다시 스왑에서 데이터를 불러옴.

#### UUID
Universally Unique Idnetifier 
범용 고유 식별자, 각 네트워크 연결 프로파일이나 시스템 객체에 대해 고유한 값을 부여하여 서로 구분할 수 있게 해준다. ( 약간 MAC 주소 느낌)

NetworkManager 등에서 새 연결을 생성할 때 자동으로 UUID가 생성된다.
nmcli connection show로 확인 가능


##### 3. 볼륨 증설

```
[root@localhost ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0            11:0    1  2.5G  0 rom
nvme0n1       259:0    0   20G  0 disk
├─nvme0n1p1   259:1    0    1G  0 part /boot
└─nvme0n1p2   259:2    0   19G  0 part
  ├─rl-root   253:0    0   17G  0 lvm  /
  └─rl-swap   253:1    0    2G  0 lvm  [SWAP]
nvme0n2       259:3    0   10G  0 disk
nvme0n3       259:4    0    5G  0 disk
└─my_vg-my_lv 253:2    0    3G  0 lvm  /mnt/my_data

```

> [!NOTE] nvme0n1p2의 lvm과 nvme0n3의 lvm의 차이
> nvme0n1p2의 LVM은 **OS 설치 시 자동으로 만들어진** LVM 파티션
> **VS**
> nvme0n3의 lvm은 사용자가 별도로 추가한 디스크
> 

```
[root@localhost ~]# pvs
  PV             VG    Fmt  Attr PSize   PFree
  /dev/nvme0n1p2 rl    lvm2 a--  <19.00g     0
  /dev/nvme0n3   my_vg lvm2 a--   <5.00g <2.00g
[root@localhost ~]# vgs
  VG    #PV #LV #SN Attr   VSize   VFree
  my_vg   1   1   0 wz--n-  <5.00g <2.00g
  rl      1   2   0 wz--n- <19.00g     0
[root@localhost ~]# lvs
  LV    VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  my_lv my_vg -wi-ao----   3.00g
  root  rl    -wi-ao---- <17.00g
  swap  rl    -wi-ao----   2.00g

```
물리 볼륨, 볼륨 그룹, 논리 볼륨 확인
용량이 없을 시 VG에 새로운 PV를 추가해야 한다.(사용하지 않은 nvme2 활용했을 것)

```
[root@localhost ~]# lvextend -L +5G /dev/my_vg/my_lv
  Insufficient free space: 1280 extents needed, but only 511 available
[root@localhost ~]# lvextend -L +1G /dev/my_vg/my_lv
  Size of logical volume my_vg/my_lv changed from 3.00 GiB (768 extents) to 4.00 GiB (1024 extents).
  Logical volume my_vg/my_lv successfully resized.

[root@localhost ~]# lvs
  LV    VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  my_lv my_vg -wi-ao----   4.00g
  root  rl    -wi-ao---- <17.00g
  swap  rl    -wi-ao----   2.00g


```
논리 그룹 1G 증설
PFree가 2G이므로 그 미만으로

```
[root@localhost ~]# xfs_growfs /mnt/my_data
meta-data=/dev/mapper/my_vg-my_lv isize=512    agcount=4, agsize=196608 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=786432, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 786432 to 1048576
[root@localhost ~]# df -h | grep my_data
/dev/mapper/my_vg-my_lv  4.0G   61M  4.0G   2% /mnt/my_data

```
xfs - 파일 시스템 확장

> **xfs** : xfs_growfx /mnt/my_data
> **VS**
> **ext4** : resize2fs /dev/my_vg/my_lv

처음 3G -> 4 G로 증설되었다.



#### vgs VS vsdisplay
##### vgs
volume group scan
요약 정보 제공
```
[root@localhost ~]# vgs
  VG    #PV #LV #SN Attr   VSize   VFree
  my_vg   1   1   0 wz--n-  <5.00g 1020.00m
  rl      1   2   0 wz--n- <19.00g       0
```
 
 * VG : 이름
* '#PV : VG에 할당된 PV 개수
* '#LV : VG에 포함된 LV 개수
* '#SN : VG에 포함된 [^1]스냅샷 개수
* ==Attr : VG의 상태(속성)=='
	* ==`attr` 속성에 따라 `lvextend`가 안 될 수 있음 (`s`(snapshot) 등)==
		* s(snapshot) : 스냅샷 볼륨은 크기 조정이 안된다.
		* t(thin pool) : thin pool의 메타데이터 부족으로 확장 불가할 수도
		* p(partitioned) : 물리적 볼륨이 부족하면 확장 불가

Thin Pool이란? - [[스토리지 할당 방식]]

`man vgs`로 명령어의 상세 정보를 살펴봤다

```
NOTES
       The vg_attr bits are:

       1  Permissions: (w)riteable, (r)ead-only
       # 쓰기 가능? 읽기 전용?

       2  Resi(z)eable
       # 동적으로 크기 조정이 가능한지?

       3  E(x)ported
       # vg이 내보내져 있어 현재 시스템에서 활성화 되어 있지 않는지
       # 그럴 경우 x가 아니라 비어서 나옴

       4  (p)artial:  one  or  more  physical  volumes belonging to the volume
          group are missing from the system
          # 부분적 상태
          # 볼륨 그룹에 속하는 하나 이상의 물리적 볼륨이 시스템에서 누락된 경우
          

       5  Allocation policy: (c)ontiguous, c(l)ing, (n)ormal, (a)nywhere
        # PE 를 어떻게 할당할지
        # Contiguous : 연속 할당 방식
        # [^2]Cling : 특정 방식
        # Normal : 일반적인 할당 방식
        # Anywhere : 임의 할당 방식

       6  (c)lustered, (s)hared
        # c : 클러스터 환경에서 사용될 때
		# s : 여러 호스트에 의해 공유될 때        


```


* Vsize : VG의 물리적 용량. 그 미만(대략)
* Vfree : 할당 x 공간

자세한 건 `man vgs`로 확인 가능하다.(너무 많음)

##### vsdisplay
상세 정보 제공
```
[root@localhost ~]# pvdisplay
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

```
* PE : Physical Extent
크기가 4MB, LVM이 PV의 공간을 이 크기의 단위로 나누어 논리 볼륨에 할당한다.
* Total PE :  총 PE 개수
* Free PE :  남아 있는 PE 개수

##### pvs
```
root[root@localhost ~]# pvs
  PV             VG    Fmt  Attr PSize   PFree
  /dev/nvme0n1p2 rl    lvm2 a--  <19.00g       0
  /dev/nvme0n3   my_vg lvm2 a--   <5.00g 1020.00m

```
* PV : 이름
* VG : 어떤 VG에 속해있는지
* Fmt : PV이 저장된 메타데이터의 포맷
* Attr : 
	* 1. 
		* a: allocatable(혹은 active). 이 PV는 VG에 의해 사용 가능(할당 가능)한 상태임을 나타낸다.
		* A : 할당 불가능하다
	* 2. 
		* 특별한 상태가 없으면 -
		* PV에 문제가 있을 시 m(missing) 등 표시
	* 3. 
		* 특별한 상태가 없으면 -
		* read-only, locked가 없음을 의미
* Psize: 물리적 용량. 그 미만(대략)
* Pfree : 할당 x 공간

자세한 건` man pvs`로 확인 가능하다.

##### ==vgs 주요 옵션==

**기본 vgs**
```
[root@localhost cron.d]# vgs
  VG    #PV #LV #SN Attr   VSize   VFree
  my_vg   1   1   0 wz--n-  <5.00g 1020.00m
  rl      1   2   0 wz--n- <19.00g       0
```


**`vgs -o +attr` - attribute 값 표시**
**`vgs -o +vgsize,vgfree` - VG 총 크기, 남은 값** 
**(이 두개는 원래도 뜸)**
```
[root@localhost cron.d]# vgs -o +attr
  VG    #PV #LV #SN Attr   VSize   VFree    Attr
  my_vg   1   1   0 wz--n-  <5.00g 1020.00m wz--n-
  rl      1   2   0 wz--n- <19.00g       0  wz--n-

[root@localhost cron.d]# vgs -o +vgsize,vgfree
  VG    #PV #LV #SN Attr   VSize   VFree    VSize   VFree
  my_vg   1   1   0 wz--n-  <5.00g 1020.00m  <5.00g 1020.00m
  rl      1   2   0 wz--n- <19.00g       0  <19.00g       0
```

`vgs -o +pvcount,lvcount `- PV, LV 개수
```
[root@localhost cron.d]# vgs -o +pvcount,lvcount
  VG    #PV #LV #SN Attr   VSize   VFree    #PV #LV
  my_vg   1   1   0 wz--n-  <5.00g 1020.00m   1   1
  rl      1   2   0 wz--n- <19.00g       0    1   2
```

`vgs -o +vguuid` - UUID 값 출력
```
[root@localhost cron.d]# vgs -o +vguuid
  VG    #PV #LV #SN Attr   VSize   VFree    VG UUID
  my_vg   1   1   0 wz--n-  <5.00g 1020.00m 151qlT-vQ8O-rGk1-Sppf-WoTL-YcAD-nlylTH
  rl      1   2   0 wz--n- <19.00g       0  WjHBJy-YOJ5-d5ls-tl08-i6KS-0feU-FzeXBx
```

`vgs --units m` - MB 단위로 출력한다. (g이면 GB)
```
[root@localhost cron.d]# vgs --units g
  VG    #PV #LV #SN Attr   VSize  VFree
  my_vg   1   1   0 wz--n-  5.00g 1.00g
  rl      1   2   0 wz--n- 19.00g    0g
[root@localhost cron.d]# vgs --units m
  VG    #PV #LV #SN Attr   VSize     VFree
  my_vg   1   1   0 wz--n-  5116.00m 1020.00m
  rl      1   2   0 wz--n- 19452.00m       0m
```

`vgs --noheadings` - 헤더빼고
`vgs --separator ,` 
```
[root@localhost cron.d]# vgs --noheadings
  my_vg   1   1   0 wz--n-  <5.00g 1020.00m
  rl      1   2   0 wz--n- <19.00g       0
[root@localhost cron.d]# vgs --separator ,
  VG,#PV,#LV,#SN,Attr,VSize,VFree
  my_vg,1,1,0,wz--n-,<5.00g,1020.00m
  rl,1,2,0,wz--n-,<19.00g,0
```


`lvs -o +lv_attr`  
`vgs -o +vg_attr`
==LV나 VG가 비활성인 것을 확인할 수 있다.==

`lvchange -a y` # LV 활성화
`lvchange -a n` # LV 비활성화

`vgchange -a y` # VG 활성화
`vgchange -a n` # VG 비활성화

-> LVM 실습 때문에 in use로 떠서 확인이 어려웠다.


<참고>
[^1]: LVM 스냅샷 : 특정 시점의 논리 볼륨 상태를 복제하여 백업이나 롤백 등에 사용한다.
	`lvcreate` 명령어에 `-s` 옵션(스냅샷)을 사용
	ex. `lvcreate -L 1G -s -n my_lv_snapshot /de/my_vg/my_lv`
	스냅샷에 할당할 공간의 크기는 1G, my_lv_snapshot이라는 것 생성한다.

[^2]: 어떤 논리 볼륨이 한 개의 물리 볼륨에 대부분 할당되어 있다면, LV를 확장할 때 "cling" 방식은 새 익스텐트를 그와 동일한 PV에 배치한다. 데이터가 한 곳에 집중된다.



#### ~~볼륨 1개 할당 더 하기(총 2개 더 할당)~~

~~`fallocate -l 1G /mnt/disk3.img  `~~
~~`losetup /dev/loop3 /mnt/disk3.img~~ 

~~![[Pasted image 20250226134857.png]]~~

~~![[Pasted image 20250226134933.png]]~~
* ~~파일시스템 생성~~
* ~~볼륨 mount~~
* ~~자동 마운트 설정~~

