---
title: Migration
description: 
published: true
date: 2026-06-08T08:14:21.896Z
tags: 
editor: markdown
dateCreated: 2026-06-08T08:14:21.896Z
---

# Raspberry Pi microSD → NVMe 이전 정리

## 목적

기존 Raspberry Pi는 microSD 카드에서 부팅하고 있었다.

microSD는 여러모로 손해이기에 남는 NVMe가 있어 거기로 옮기려고 한다. 

그래서 기존 microSD에 있던 OS, 설정, 서비스, 파일을 NVMe로 복제하고, 이후부터는 microSD 없이 NVMe에서 부팅하고 저장하도록 변경했다.

<br>

## 기존 상태

이전 디스크 상태는 아래와 같았다.

    lsblk

    NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
    loop0         7:0    0     2G  0 loop 
    mmcblk0     179:0    0  29.7G  0 disk 
    ├─mmcblk0p1 179:1    0   512M  0 part /boot/firmware
    └─mmcblk0p2 179:2    0  29.2G  0 part /
    zram0       254:0    0     2G  0 disk [SWAP]
    nvme0n1     259:0    0 232.9G  0 disk 
    └─nvme0n1p1 259:1    0 232.9G  0 part /mnt/nvme

이 상태에서 원본과 대상은 아래와 같다.

    원본 디스크: /dev/mmcblk0
    대상 디스크: /dev/nvme0n1

`mmcblk0`은 기존 microSD 카드이다.

`nvme0n1`은 새로 사용할 NVMe 디스크이다.

<br>

## rpi-clone 설치

microSD 전체를 NVMe로 복제하기 위해 `rpi-clone`을 사용했다.

필요 패키지를 먼저 설치한다.

    sudo apt update
    sudo apt install -y git rsync parted e2fsprogs dosfstools

그다음 `rpi-clone`을 설치한다.

    curl https://raw.githubusercontent.com/geerlingguy/rpi-clone/master/install | sudo bash

설치 확인은 아래 명령어로 한다.

    which rpi-clone
    rpi-clone

정상 설치되면 대략 아래 경로가 나온다.

    /usr/local/sbin/rpi-clone

<br>

## NVMe 마운트 해제

처음 NVMe는 `/mnt/nvme`에 마운트되어 있었다.

복제 대상 디스크는 마운트된 상태로 두면 안 되기 때문에 먼저 언마운트했다.

    cd ~
    sudo umount /mnt/nvme

다시 확인한다.

    lsblk

이때 `nvme0n1p1` 옆에 `/mnt/nvme`가 보이지 않아야 한다.

<br>

## rpi-clone 실행

복제 명령어는 아래와 같이 실행했다.

    sudo rpi-clone nvme0n1

중요한 점은 파티션 이름이 아니라 디스크 이름을 넣는 것이다.

    # 맞는 명령어
    sudo rpi-clone nvme0n1

    # 틀린 명령어
    sudo rpi-clone nvme0n1p1

실행하면 아래처럼 원본 디스크와 대상 디스크가 표시된다.

    Booted disk: mmcblk0 31.9GB
    Destination disk: nvme0n1 250.1GB

이후 대상 디스크의 기존 데이터가 지워진다는 경고가 나온다.

    Initialize and clone to the destination disk nvme0n1?  (yes/no):

여기서 `yes`를 입력했다.

<br>

## 파일시스템 라벨 입력

진행 중 아래와 같은 질문이 나왔다.

    Optional destination ext type file system label (16 chars max):

이 항목은 NVMe의 root 파티션 라벨을 정하는 부분이다.

특별히 지정할 필요가 없어서 아무것도 입력하지 않고 `Enter`를 눌렀다.

<br>

## 복제 진행

이후 `rpi-clone`이 NVMe를 초기화하고, microSD의 파티션 구조를 NVMe에 맞게 생성했다.

진행 로그는 대략 아래와 같았다.

    Initializing
      Imaging past partition 1 start.
      => dd if=/dev/mmcblk0 of=/dev/nvme0n1 bs=1M count=12 ...
      Resizing destination disk last partition ...
        Resize success.
      Changing destination Disk ID ...
      => mkfs -t vfat -F 32 -n bootfs /dev/nvme0n1p1 ...
      => mkfs -t ext4 -L rootfs /dev/nvme0n1p2 ...

    Syncing file systems
      Mounting /dev/nvme0n1p2 on /mnt/clone
      => rsync // /mnt/clone with-root-excludes ...

`rsync` 단계는 microSD의 root 파일시스템을 NVMe로 복사하는 단계이다.

시간이 오래 걸릴 수 있지만 정상 진행 상태이다.

중간에 아래 메시지가 나올 수 있다.

    mount: (hint) your fstab has been modified, but systemd still uses
           the old version; use 'systemctl daemon-reload' to reload.

이 메시지는 치명적인 오류가 아니다.

`rpi-clone`이 복제 대상의 `fstab`을 수정했지만, 현재 실행 중인 systemd가 아직 예전 정보를 들고 있다는 안내이다.

복제 완료 후 재부팅하거나 `systemctl daemon-reload`를 실행하면 된다.

<br>

## microSD 제거 후 NVMe 부팅

복제가 끝난 뒤 라즈베리파이를 종료했다.

    sudo poweroff

전원이 완전히 꺼진 뒤 microSD 카드를 제거했다.

그다음 NVMe만 연결된 상태로 전원을 다시 넣었다.

이렇게 하면 Raspberry Pi가 NVMe에서 부팅을 시도한다.

<br>

## NVMe 부팅 확인

부팅 후 `lsblk`로 확인했다.

    lsblk

결과는 아래와 같았다.

    NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
    loop0         7:0    0     2G  0 loop 
    zram0       254:0    0     2G  0 disk [SWAP]
    nvme0n1     259:0    0 232.9G  0 disk 
    ├─nvme0n1p1 259:1    0   512M  0 part /boot/firmware
    └─nvme0n1p2 259:2    0 232.4G  0 part 

이 상태에서 `mmcblk0`이 보이지 않는 것은 정상이다.

microSD 카드를 제거했기 때문이다.

`nvme0n1p1`이 `/boot/firmware`로 잡혀 있고, `nvme0n1p2`가 root 파일시스템으로 사용되는 구조이다.

추가 확인 명령어는 아래와 같다.

    findmnt /
    findmnt /boot/firmware
    df -h

정상이라면 아래처럼 보여야 한다.

    /              /dev/nvme0n1p2
    /boot/firmware /dev/nvme0n1p1

<br>

## systemd reload

복제 중 `fstab` 관련 메시지가 나왔기 때문에 부팅 후 systemd 설정을 다시 읽도록 했다.

    sudo systemctl daemon-reload

<br>


## 확인하기

기존에는 microSD 카드의 `/dev/mmcblk0`에서 부팅했다.

이제는 NVMe 디스크인 `/dev/nvme0n1`에서 부팅한다.

최종 구조는 아래와 같다.

    /dev/nvme0n1p1 → /boot/firmware
    /dev/nvme0n1p2 → /

microSD 카드는 더 이상 사용하지 않는다.

부팅, OS, 서비스 파일, Wiki.js 데이터, Maskly 정적 파일, Docker 관련 파일이 모두 NVMe에서 동작하는 구조가 되었다.

