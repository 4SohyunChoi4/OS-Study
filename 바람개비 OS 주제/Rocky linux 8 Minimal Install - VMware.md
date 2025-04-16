
## rocky linux 8.10 minimal iso 설치
[https://rockylinux.org/download](https://rockylinux.org/download)

[https://gonghakjoa.tistory.com/91](https://gonghakjoa.tistory.com/91)


~~virtual box에 설치~~
## VMWare에 설치

https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion
회원가입

VMware Workstation 162.2 Pro 설치

시리얼 넘버
`FC3D0-FTFE5-H81WQ-RNWZX-PV894`


![[Pasted image 20250304093805.png]]

![[Pasted image 20250304093814.png]]

![[Pasted image 20250304151643.png]]

![[Pasted image 20250304095356.png]]
user 생성

![[Pasted image 20250304095429.png]]
이때(Bridge로 함.) 안돼서 다시 설치하고 NAT로 변경 후 192.168 ... ip 뜬 것을 확인했다.


![[Pasted image 20250304095457.png]]


설치 완료했다.


`[root@localhost ~]# ip addr`
`1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000`
    `link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00`
    `inet 127.0.0.1/8 scope host lo`
       `valid_lft forever preferred_lft forever`
    `inet6 ::1/128 scope host`
       `valid_lft forever preferred_lft forever`
`2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000`
    `link/ether 00:0c:29:ff:b3:09 brd ff:ff:ff:ff:ff:ff`
    `altname enp3s0`
    `inet 192.168.38.128/24 brd 192.168.38.255 scope global dynamic noprefixroute ens160`
       `valid_lft 1551sec preferred_lft 1551sec`
    `inet6 fe80::20c:29ff:feff:b309/64 scope link noprefixroute`
       `valid_lft forever preferred_lft forever`



