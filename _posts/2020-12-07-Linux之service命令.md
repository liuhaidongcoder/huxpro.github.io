---
layout: post
title: "Linux之service命令"
subtitle: "Linux之service命令"
author: "刘海东"
header-img: "img/post-bg-universe.jpg"
header-mask: 0.4
tags:
  - Linux
  - Service命令
---

## Linux之Service命令

### 功能用途
service命令主要对服务进行管理,比如启动（start）、停止（stop）、重启（restart）、查看状态（status）等

### 功能原理
service命令本身是一个shell脚本，它在/etc/init.d/目录查找指定的服务脚本，然后调用该服务脚本来完成任务。

#### 命令地址
```bash
[root@localhost ~]# which service
/sbin/service
``` 
#### 脚本内容
```bash
#!/bin/sh

. /etc/init.d/functions

VERSION="$(basename $0) ver. 0.91"
USAGE="Usage: $(basename $0) < option > | --status-all | \
[ service_name [ command | --full-restart ] ]"
SERVICE=
SERVICEDIR="/etc/init.d"
OPTIONS=

if [ $# -eq 0 ]; then
   echo "${USAGE}" >&2
   exit 1
fi

cd /
while [ $# -gt 0 ]; do
  case "${1}" in
    --help | -h | --h* )
       echo "${USAGE}" >&2
       exit 0
       ;;
    --version | -V )
       echo "${VERSION}" >&2
       exit 0
       ;;
    *)
       if [ -z "${SERVICE}" -a $# -eq 1 -a "${1}" = "--status-all" ]; then
          cd ${SERVICEDIR}
          for SERVICE in * ; do
            case "${SERVICE}" in
              functions | halt | killall | single| linuxconf| kudzu)
                  ;;
              *)
                if ! is_ignored_file "${SERVICE}" \
                    && [ -x "${SERVICEDIR}/${SERVICE}" ]; then
                  env -i PATH="$PATH" TERM="$TERM" "${SERVICEDIR}/${SERVICE}" status
                fi
                ;;
            esac
          done
          exit 0
       elif [ $# -eq 2 -a "${2}" = "--full-restart" ]; then
          SERVICE="${1}"
          if [ -x "${SERVICEDIR}/${SERVICE}" ]; then
            env -i PATH="$PATH" TERM="$TERM" "${SERVICEDIR}/${SERVICE}" stop
            env -i PATH="$PATH" TERM="$TERM" "${SERVICEDIR}/${SERVICE}" start
            exit $?
          fi
       elif [ -z "${SERVICE}" ]; then
         SERVICE="${1}"
       else
         OPTIONS="${OPTIONS} ${1}"
       fi
       shift
       ;;
   esac
done

if [ -f "${SERVICEDIR}/${SERVICE}" ]; then
   env -i PATH="$PATH" TERM="$TERM" "${SERVICEDIR}/${SERVICE}" ${OPTIONS}
else
   echo $"${SERVICE}: unrecognized service" >&2
   exit 1
fi
```










