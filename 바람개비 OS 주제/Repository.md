
- 관련 파일 위치, 설정값 의미

### Repository란?

보통 윈도우 환경에서는 실행파일을 다운로드 받거나 배포된 CD나 DVD 포멧을 이용한다.

하지만 리눅스에서는 크리에이터가 소프트웨어를 **.deb(Ubuntu)나 .rpm(Red Hat)으로 배포**하며, 여기에 패키지와 다른 설치에 필요한 요소들이 포함되어 있다.

우리는 이러한 파일들을 package repository를 통해 접근할 수 있다.

리눅스는 대부분의 소프트웨어와 프로그램을 패키지로 배포한다. 이러한 패키지는 시스템이 잘 설치 및 실행되기 위한 모든 필수 파일과 메타데이터를 포함한다.

따라서 package repoistory는 미리 컴파일된 소프트웨어 패키지 중앙 저장소이다. 수동으로 소스코드로부터 소프트웨어를 컴파일링하는 것보다 더욱 효율적이다.


## mirrorlist vs baseurl

```
mirrorlist=https://mirrors.rockylinux.org/mirrorlist?arch=$basearch&repo=AppStream-$releasever
```
==실제 링크가 아니라 mirror 주소들을 동적으로 제공하는 특수한 리스트 파일을 내려주는 서버의 주소 => 미러 리스트 서버==
==이 목록 중 가장 빠르거나 가용성이 좋은 미러를 고른다.==

```
#baseurl=http://dl.rockylinux.org/$contentdir/$releasever/BaseOS/$basearch/os/
```

==고정된 repository 주소, 단일 서버 주소==

mirrorlist=> dynamic / baseurl => static
우선순위 문제, 둘 중 하나만 사용하도록 권장

#### /etc 내 파일
`/etc/yum.conf`
yum의 전반적인 동작을 제어한다.

`/etc/yum.repos.d/`
각 리포지토리의 설정 파일이 존재한다

이 두개도 자동 관리 도구에 의해 덮어씌어질 수 있다.

## AppStream Repository
```
[root@localhost yum.repos.d]# cat /etc/yum.repos.d/Rocky-AppStream.repo
# Rocky-AppStream.repo
#
# The mirrorlist system uses the connecting IP address of the client and the
# update status of each mirror to pick current mirrors that are geographically
# close to the client.  You should use this for Rocky updates unless you are
# manually picking other mirrors.
#
# If the mirrorlist does not work for you, you can try the commented out
# baseurl line instead.

[appstream]
name=Rocky Linux $releasever - AppStream
mirrorlist=https://mirrors.rockylinux.org/mirrorlist?arch=$basearch&repo=AppStream-$releasever
#baseurl=http://dl.rockylinux.org/$contentdir/$releasever/AppStream/$basearch/os/
gpgcheck=1
enabled=1
countme=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-rockyofficial


```
## BaseOs repository

```
[root@localhost yum.repos.d]# cat /etc/yum.repos.d/Rocky-BaseOS.repo
# Rocky-BaseOS.repo
#
# The mirrorlist system uses the connecting IP address of the client and the
# update status of each mirror to pick current mirrors that are geographically
# close to the client.  You should use this for Rocky updates unless you are
# manually picking other mirrors.
#
# If the mirrorlist does not work for you, you can try the commented out
# baseurl line instead.

[baseos]
name=Rocky Linux $releasever - BaseOS
mirrorlist=https://mirrors.rockylinux.org/mirrorlist?arch=$basearch&repo=BaseOS-$releasever
#baseurl=http://dl.rockylinux.org/$contentdir/$releasever/BaseOS/$basearch/os/
gpgcheck=1
enabled=1
countme=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-rockyofficial

```

2개의 저장소가 있다.

#### **저장소를 BaseOS와 AppStream으로 분리한 이유  

RockyLinux뿐 아니라 CentOS 8과 RHEL 8 부터는 "BaseOS", "AppStream" 2개의 저장소를 사용한다.  
저장소를 이 두가지로 분리한 이유는 자주 업데이트되는 AppStream패키지들로 인하여 기반이되는 OS플랫폼관련 패키지들에 영향을 받지않는 안정성과 유연성 때문.

```null
- BaseOS 저장소 : 운영체제의 기반이 되는 기본 기능의 코어세트 제공
- AppStream : 운영서비스의 Application, 런타임언어 및 데이터베이스
```



repository 설정을 확인할 수 있다.

![[Pasted image 20250226013855.png]]
한국에서는 3개의 mirror 사이트 운영중이다.


#### local repository
외부와의 통신이 엄격히 제한적인 상황된 내부망에서 패키지 설치를 위하여 폐쇠망에서는 local repository를 운영하게 된다.

repo 파일의 형식
```
[저장소ID]

name=저장소이름

baseurl=패키지와repodata가 존재하는 위치

gpgcheck=0(GPG서명사용안함) 또는 1(GPG서명사용함)

enabled=0(Repo비활성화)또는 1(Repo활성화)
```



[참고]
https://www.linux.co.kr/bbs/board.php?bo_table=lecture&wr_id=4494
