---
layout: post
title: The experience of using linux shell
comments: true
author: "Yin Haomin"
tags:
    - Experience
    - linux
    - shell
---

#### 1. 怎么使用命令在FTP中跳过输入name, password.

以下为sample

```
#!/bin/bash
#$1 is the host name of the ftp server.
#$2 is the port of the ftp server.
#$3 is the user name of the ftp server.
#$4 is the password of the ftp server.
#$5 is the local folder path
#$6 is the remote folder path
#usage:this_script <host name> <port> <user name> <password> <local folder path> <remote folder path> 
HOST="$1"
PORT="$2"
USER="$3"
PASSWD="$4"
LOCALPATH="$5"
REMOTEPATH="$6"
ftp -n $HOST $PORT <<END_SCRIPT
quote USER $USER
quote PASS $PASSWD
cd $REMOTEPATH
mput $LOCALPATH/*
quit
END_SCRIPT
exit 0 
```

#### 2. 使用命令检查FTP上文件是否存在

```
#!/bin/bash
#$1 is the host name of the ftp server.
#$2 is the port of the ftp server.
#$3 is the user name of the ftp server.
#$4 is the password of the ftp server.
#$5 is the file path
#usage:this_script <host name> <port> <user name> <password> <file path>
HOST="$1"
PORT="$2"
USER="$3"
PASSWD="$4"
FILEPATH="$5"
errlog=./err.log
ftpCheckFile(){
ftp -n $HOST $PORT <<EOF
quote USER $USER
quote PASS $PASSWD
ls $FILEPATH
bye
EOF
}
rm -f $errlog
ftpCheckFile $1 >/dev/null 2>$errlog
bytes=`wc -c $errlog | awk '{print $1}'`
if [ $bytes -eq 0 ]; then
      echo "$1 - Exist!"
else
      echo "$1 - Not exist!"
fi

```
