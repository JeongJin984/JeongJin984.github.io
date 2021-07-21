---
title: Kubernetes 0
author: Jiny
date: 2021-06-26 14:30:00 +0800
categories: [Infra, Kubernetes]
tags: [k8s, kubernetes, infra]
toc: false
---
 
# Kubernetes 0

___

## 💿 **도커 컨테이너와 가상머신**

> 가상머신: 운영체제 위헤 하드웨어를 에뮬레이션하고 그 위에 운영체제를 올리고 프로세스를 실행

> 컨테이너: 하드웨어 에뮬레이션 없이 리눅스 커널을 공유해서 바로 프로세스를 실행

```Bash
$ docker version
Client: Docker Engine - Community
 Version:           19.03.5
  ...

$ docker run -it ubuntu:latest bash
root@bfccfb4136ae:/# uname -a
Linux f44e33a332f2 4.15.0-70-generic #79-Ubuntu SMP Tue Nov 12 10:36:11 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux

root@bfccfb4136ae:/# cat /etc/*-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.3 LTS"
...
```

```bash
$ docker run -it centos:latest bash
[root@bb0a9b851dbd /]# uname -a
Linux bb0a9b851dbd 4.15.0-70-generic #79-Ubuntu SMP Tue Nov 12 10:36:11 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux

[root@bb0a9b851dbd /]# cat /etc/*-release
CentOS Linux release 8.0.1905 (Core)
NAME="CentOS Linux"
VERSION="8 (Core)"
```

위의 2개의 명령어를 통해

- cat를 통해 파일시스템(배포판)이 다름
- uname -a의 출력 결과가 같음(커널이 같다)
  - 둘다 Utuntu 18.04 Vagrant 가상머신에서 실행

**이를 통해**

- 도커 데스크탑은 리눅스킷 기반의 경량 가상머신 위에서 도커를 실행
- 컨테이너는 호스트 시스템의 커널을 사용한다.
- 컨테이너는 이미지에 따라서 실행되는 환경이 달라진다.

컨테이너가 서로 다른 파일 시스템을 가질 수 있는 이유는 이미지(파일의 집합)를 루트 파일 시스템으로 강제로 인식시켜 프로세스를 실행하기 때문(즉 자신의 루트 /가 아닌 그 아래의 특정 디렉터리를 루트로 인식)

___

## 💿 **도커 컨테이너와 프로세스**

> 도커 컨테이너는 가상머신이 아니고 프로세스다.

```bash
$ echo $$
5673

$ docker run -it ubuntu:latest bash
root@9a09675d42ed:/# echo $$
1
```

일반적으로 1번 프로세스는 pstree에서 확인해 보면 모든 프로세스가 물려있는 프로세스

**컨테이너?**

> 리눅스 커널에 포함된 프로세스 격리 기술들을 사용해서 생성된 특별한 프로세스

```bash
root@47672c55680d:/# kill -9 1
root@47672c55680d:/#
```

_1번 프로세스는 원래 죽일수도 없다._

위와 같은 사실에 비추어 볼때 그러면 어떻게 1번 프로세스가 2개일 수 있냐?(도커는 1번을 죽일수 있고 host의 1번을 죽일 수 없다.)
- 리눅스의 네임스페이스 때문
- PID 네임스페이스가 분리되어 있기 때문에 1번이 됨

___

## 💿 **Namespace**

**1번**

```Bash
$ echo $$
5673

$ mkdir busybox-image
$ docker run --name busybox-image busybox:latest
$ docker export busybox-image > ./busybox-image/image.tar
$ cd busybox-image; tar xf image.tar; cd ..
$ sudo unshare -p -f --mount-proc chroot ./busybox-image /bin/sh
/ # echo $$
1
```

**2번**

```Bash
$ sudo unshare -p /bin/sh
# echo $$
2170
```
- 2번째는 unshare를 통해 네임스페이스를 분리하더라도 호스트의 파일 시스템을 그대로 사용하고 있어서 PID 관련 정보는 /proc 아래의 정보를 사용하기 때문
- 그래서 --mount-proc으로 /proc을 분리하고 독자적인 환경을 위해 이미지(busybox-image)를 사용

### 2개의 PID

1. 프로세스가 바라보는 PID
2. 호스트가 바라보는 PID

2번을 실행해서 프로세스를 보면

```bash
$ ps aux --sort +start_time | tail -n 5
root      2376  0.0  0.1  21472  3700 pts/0    S    03:49   0:00 bash
root      2386  0.0  0.0   7456   732 pts/0    S    03:49   0:00 unshare -p -f --mount-proc chroot ./busybox-image /bin/sh
root      2387  0.0  0.0   1312     4 pts/0    S+   03:49   0:00 /bin/sh
vagrant   2396  0.0  0.1  37516  3480 pts/1    R+   03:50   0:00 ps aux --sort +start_time
vagrant   2397  0.0  0.0   7508   856 pts/1    S+   03:50   0:00 tail -n 5
```

- 이렇게 나오는데 2387을 죽이면 sh unshare로 실행된 sh도 같이 종료됨
- 둘이 같은 프로세스이기 때문
- 즉 프로세스가 보기에는 자신의 PID가 1번이지만 호스트 입장에서는 2387이라는 뜻

___

## 💿 도커 컨테이너와 프로세스

```bash
$ diff \
  <(ls -Al /proc/1/ns | awk '{ print $9 $10 $11 }') \
  <(ls -Al /proc/2899/ns | awk '{ print $9 $10 $11 }')

3,7c3,7
< ipc->ipc:[4026531839]
< mnt->mnt:[4026531840]
< net->net:[4026531993]
< pid->pid:[4026531836]
< pid_for_children->pid:[4026531836]
---
> ipc->ipc:[4026532238]
> mnt->mnt:[4026532236]
> net->net:[4026532298]
> pid->pid:[4026532239]
> pid_for_children->pid:[4026532239]
9c9
< uts->uts:[4026531838]
---
> uts->uts:[4026532237]
```

- ipc, mnt, net, pid, pid_for_children, uts 네임스페이스가 다르다는 것을 확인할 수 있습니다
- 네임스페이스는 다르지만 ps로 일반적인 프로세스라는 것을 한 번 확인 가능

## **컨테이너는 특별한 프로세스다~~~ 이말이야~!**

___


https://www.44bits.io/ko/post/is-docker-container-a-virtual-machine-or-a-process#%EB%8F%84%EC%BB%A4-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EA%B0%80-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EC%BB%A4%EB%84%90%EA%B3%BC-%ED%8C%8C%EC%9D%BC-%EC%8B%9C%EC%8A%A4%ED%85%9C-%ED%99%95%EC%9D%B8%ED%95%98%EA%B8%B0)