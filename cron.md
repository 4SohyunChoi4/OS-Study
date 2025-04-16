### cron
유닉스 계열의 잡 스케줄러

| 필드명         | 값의 허용 범위                         | 허용된 특수문자      |
| ----------- | -------------------------------- | ------------- |
| 초 (Seconds) | 0 ~ 59                           | , - * /       |
| 분 (Minutes) | 0 ~ 59                           | , - * /       |
| 시 (Hours)   | 0 ~ 23                           | , - * /       |
| 일 (Day)     | 1 ~ 31                           | , - * / ? L W |
| 월 (Month)   | 1 ~ 12 or JAN ~ DEC              | , - * /       |
| 요일 (Week)   | 0 ~ 6 or SUN ~ SAT  <br>(7도 일요일) | , - * / ? L # |
| 연도 (Year)   | empty or 1970 ~ 2099             | , - * /       |

#### Cron 표현식 - 특수문자
* * : 모든 값
* ? : 특정한 값이 없음
	* 일 or 요일 필드에만 사용 가능
	* 조건이 없다, 관계 없다
	* 예를 들어 '일'은 설정해야 하는데 '요일'은 무관할 때 한쪽은 요일에 ? 사용
* - : 범위 (ex. MON-WED)
* , : 특별한 값일 때만 (ex. MON, WED)
* / : 시작 시간 / 단위 (ex. 0분부터 매 5분이면 0/5)
* L : 일에서 사용하면 마지막 일, 요일에는 마지막 요일(토요일)
* W : 가장 가까운 평일 (ex. 15W -> 15일에서 가장 가까운 평일 (월~ 금)을 찾음
* `#` : 몇째주의 무슨 요일 (ex. 3#2 : 2번째주 수요일) 


*예시*
ex. 0 0 12 * * ? 
0분 0초 12시 매일

ex. 0 0/5 * * * ?
5분마다

## ==cron의 설정 위치==

### /etc/cron.d/sysstat
```
[root@localhost cron]# cat /etc/cron.d/sysstat # rocky linux 8
*/10 * * * * root /usr/lib64/sa/sa1 1 1 # 매 10분마다 데이터 수집
53 23 * * * root /usr/lib64/sa/sa2 -A # 23:53 분에 하루 요약 보고서 수집
```

```
[root@localhost cron]# journalctl -u crond
-- Logs begin at Tue 2025-04-08 22:16:26 KST, end at Tue 2025-04-08 23:59:01 KST. --
Apr 08 22:16:34 localhost.localdomain systemd[1]: Started Command Scheduler.
Apr 08 22:16:34 localhost.localdomain crond[854]: (CRON) STARTUP (1.5.2)
Apr 08 22:16:34 localhost.localdomain crond[854]: (CRON) INFO (Syslog will be used instead of sendmail.)
Apr 08 22:16:34 localhost.localdomain crond[854]: (CRON) INFO (RANDOM_DELAY will be scaled with factor 70% if used.)
Apr 08 22:16:34 localhost.localdomain crond[854]: (CRON) INFO (running with inotify support)
Apr 08 22:20:01 localhost.localdomain CROND[2751]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Apr 08 22:30:01 localhost.localdomain CROND[6299]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Apr 08 22:40:01 localhost.localdomain CROND[9846]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Apr 08 22:50:01 localhost.localdomain CROND[13364]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Apr 08 23:00:01 localhost.localdomain CROND[16868]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Apr 08 23:01:02 localhost.localdomain CROND[17225]: (root) CMD (run-parts /etc/cron.hourly)
Apr 08 23:10:02 localhost.localdomain CROND[20463]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Apr 08 23:20:01 localhost.localdomain CROND[24120]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Apr 08 23:30:01 localhost.localdomain CROND[27712]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Apr 08 23:40:01 localhost.localdomain CROND[31375]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Apr 08 23:45:01 localhost.localdomain CROND[33196]: (root) CMD (/usr/local/bin/my_script.sh)
Apr 08 23:46:02 localhost.localdomain CROND[33552]: (root) CMD (/usr/local/bin/my_script.sh)
Apr 08 23:47:01 localhost.localdomain CROND[33908]: (root) CMD (/usr/local/bin/my_script.sh)
Apr 08 23:48:01 localhost.localdomain CROND[34264]: (root) CMD (/usr/local/bin/my_script.sh)
Apr 08 23:49:01 localhost.localdomain CROND[34627]: (root) CMD (/usr/local/bin/my_script.sh)
Apr 08 23:50:01 localhost.localdomain CROND[34993]: (root) CMD (/usr/local/bin/my_script.sh)
Apr 08 23:50:01 localhost.localdomain CROND[34995]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Apr 08 23:53:01 localhost.localdomain CROND[36076]: (root) CMD (/usr/lib64/sa/sa2 -A)
Apr 08 23:54:01 localhost.localdomain crond[854]: (*system*) RELOAD (/etc/crontab)

```
`journalctl`로 확인해봤다. 
중간에 내가 한 script 파일도 확인됐다.


```
[root@localhost ~]# vi /etc/crontab

HELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

```
시스템 전역에서 사용하는 cron 테이블

```
[root@localhost ~]# cd /etc/cron.d
[root@localhost cron.d]# ls
0hourly  raid-check  sysstat

```
개별 cron 설정 파일들을 여기에 두면 crontab처럼 실행된다.

```
[root@localhost cron.d]# /etc/cron.
cron.d/       cron.daily/   cron.hourly/  cron.monthly/ cron.weekly/

```
이 안에 스크립트 넣으면 주기별로 실행된다.

```
[root@localhost cron.d]# cd /var/spool/cron
[root@localhost cron]# ls
root
[root@localhost cron]# cat root
* * * * * echo "Cron test executed at $(date)" >> /tmp/cron_test.log

```
사용자 별로 `crontab -e` 한 것 저장된다.
직접 수정x, `crontab -e` 로 수정한다.




## ==cron 테스트하기==
```
[root@localhost ~]# vi /etc/cron.d/cron_test
* * * * * root /usr/local/bin/my_script.sh
```
매 초마다` my_script.sh` 실행하자


```
[root@localhost ~]# echo "* * * * * root /usr/local/bin/my_script.sh" > /etc/cron.d/cron_test
```
크론 설정 파일 생성한다.
바로 된다.


```
[root@localhost ~]# cat /tmp/cron_test.log
Cron test executed at Wed Apr  2 00:19:01 KST 2025
Cron test executed at Wed Apr  2 00:20:01 KST 2025
Cron test executed at Wed Apr  2 00:21:01 KST 2025
Cron test executed at Wed Apr  2 00:22:01 KST 2025
Cron test executed at Wed Apr  2 00:23:01 KST 2025
Cron test executed at Wed Apr  2 00:24:01 KST 2025
Cron test executed at Wed Apr  2 00:25:01 KST 2025
Cron test executed at Wed Apr  2 00:26:01 KST 2025
Cron test executed at Wed Apr  2 00:27:02 KST 2025
Cron test executed at Wed Apr  2 00:28:01 KST 2025
Cron test executed at Wed Apr  2 00:29:01 KST 2025
Cron test executed at Wed Apr  2 00:30:01 KST 2025
Cron test executed at Wed Apr  2 00:31:01 KST 2025
Cron test executed at Wed Apr  2 00:32:01 KST 2025
Cron test executed at Wed Apr  2 00:33:01 KST 2025
Cron test executed at Wed Apr  2 00:34:01 KST 2025

```

```
[root@localhost ~]# rm /etc/cron.d/cron_test

```
끝난 뒤에는 cron 삭제한다.
### crontab
`cron`이 실행할 작업 목록을 저장하는 파일
<-> `cron` : 주기적으로 작업을 실행하는 데몬(백그라운드 서비스)

| 필드명         | 값의 허용 범위                         | 허용된 특수문자      |
| ----------- | -------------------------------- | ------------- |
| 분 (Minutes) | 0 ~ 59                           | , - * /       |
| 시 (Hours)   | 0 ~ 23                           | , - * /       |
| 일 (Day)     | 1 ~ 31                           | , - * / ? L W |
| 월 (Month)   | 1 ~ 12 or JAN ~ DEC              | , - * /       |
| 요일 (Week)   | 0 ~ 6 or SUN ~ SAT  <br>(7도 일요일) | , - * / ? L # |
-> cron에서 초, 년도가 빠진 버전

**crontab 옵션**
* -  u : user 정의
* - e(edit) : 유저의 크론탭 편집 -> 각 유저마다 개인이 가진 크론탭이 있음
* - l (list) : 유저의 크론탭 리스트 본다
* -r (remove) : 유저의 크론탭 지운다



```
[root@localhost ~]# crontab -e

* * * * * echo "Cron test executed at $(date)" >> /tmp/cron_test.log
	

## ~~ 몇분 뒤 ~~
[root@localhost cron.d]# cat /tmp/cron_test.log
Cron test executed at Wed Apr  2 00:19:01 KST 2025
Cron test executed at Wed Apr  2 00:20:01 KST 2025
Cron test executed at Wed Apr  2 00:21:01 KST 2025
Cron test executed at Wed Apr  2 00:22:01 KST 2025
Cron test executed at Wed Apr  2 00:23:01 KST 2025
Cron test executed at Wed Apr  2 00:24:01 KST 2025
Cron test executed at Wed Apr  2 00:25:01 KST 2025
Cron test executed at Wed Apr  2 00:26:01 KST 2025
Cron test executed at Wed Apr  2 00:27:02 KST 2025
Cron test executed at Wed Apr  2 00:28:01 KST 2025
Cron test executed at Wed Apr  2 00:29:01 KST 2025
Cron test executed at Wed Apr  2 00:30:01 KST 2025

```


## cron 대신 일회성 작업?
### at
지정한 시간에 한 번만 명령을 실행

```
[root@localhost ~]# vi my_script.sh
#!/bin/bash
echo "동작 굿"
date
whoami


[root@localhost ~]# echo "sh my_script.sh > /tmp/at_test.log 2>&1" | at now
warning: commands will be executed using /bin/sh
job 11 at Tue Apr  8 23:17:00 2025

[root@localhost ~]# cat /tmp/at_test.log
동작 굿
Tue Apr  8 23:17:34 KST 2025
root
```

```
at 12:34
at now + 10 minutes
at 3pm
at midnight
```

여러가지 스타일로 가능

```
[root@localhost ~]# atq
10      Tue Apr  8 23:17:00 2025 a root

```
`at` 목록 확인

`atrm` - `at` 작업 삭제

### systemd-run + timer

CentOS 7 이후에서 사용 가능
일회성, 주기 모두 가능
```
[root@localhost ~]# systemd-run --on-calendar="2025-04-08 23:31" echo "hello world"
Running timer as unit: run-r03f752dfe74945848778cf2c19de0174.timer
Will run service as unit: run-r03f752dfe74945848778cf2c19de0174.service

```
### sleep + &

```
[root@localhost ~]# (sleep 6; sh my_script.sh) &
[2] 28059
[root@localhost ~]# 
동작 굿
Tue Apr  8 23:31:04 KST 2025
root
```