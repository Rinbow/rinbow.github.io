---
title: Rsync 同步文件 client&server 配置
tags: [linux, rsync]
---

#### Server 端配置

编辑 /etc/rsyncd.conf 文件

```
######### 全局配置参数 ##########
# 指定 rsync 端口。默认 873
port = 888
# rsync 服务的运行用户，默认是 nobody，文件传输成功后属主将是这个 uid
uid = root
# rsync 服务的运行组，默认是 nobody，文件传输成功后属组将是这个 gid
gid = root
# rsync daemon 在传输前是否切换到指定的 path 目录下，并将其监禁在内
use chroot = no
# 指定最大连接数量，0 表示没有限制
max connections = 200
# 确保 rsync 服务器不会永远等待一个崩溃的客户端，0 表示永远等待
timeout = 300
# 客户端连接过来显示的消息
motd file = /var/rsyncd/rsync.motd
# 指定 rsync daemon 的 pid 文件
pid file = /var/run/rsyncd.pid
# 指定锁文件
lock file = /var/run/rsync.lock
# 指定 rsync 的日志文件，而不把日志发送给 syslog
log file = /var/log/rsyncd.log
# 指定哪些文件不用进行压缩传输
dont compress = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2

###########下面指定模块，并设定模块配置参数，可以创建多个模块###########
[testmodule]
# 指定该模块的路径，该参数必须指定，启动 rsync 服务前该目录必须存在，rsync 请求访问模块本质就是访问该路径
path = /data/src-data/
# 忽略某些 IO 错误信息
ignore errors
# 指定该模块是否可读写，即能否上传文件，false 表示可读写，true 表示可读不可写。所有模块默认不可上传
read only = yes
# 指定该模式是否支持下载，设置为 true 表示客户端不能下载。所有模块默认可下载
write only = false
# 客户端请求显示模块列表时，该模块是否显示出来，设置为 false 则该模块为隐藏模块。默认 true
list = true
# 指定允许连接到该模块的机器，多个 ip 用空格隔开或者设置区间
hosts allow = 10.0.0.0/24
# 指定不允许连接到该模块的机器
hosts deny = 0.0.0.0/32
# 指定连接到该模块的用户列表，只有列表里的用户才能连接到模块，用户名和对应密码保存在 secrts file 中，这里使用的不是系统用户，而是虚拟用户。不设置时，默认所有用户都能连接，但使用的是匿名连接
auth users = testuser
保存 auth users 用户列表的用户名和密码，每行包含一个 username:passwd。由于"strict modes"默认为true，所以此文件要求非 rsync daemon 用户不可读写。只有启用了 auth users 该选项才有效
secrets file = /etc/rsyncd.passwd
```

执行`rsync --daemon`或`rsync --daemon --config=/etc/rsyncd.conf`启动 rsync daemon。

注意：

- 客户端推到服务端时，文件的属主和属组是配置文件中指定的 uid 和 gid。但是客户端从服务端拉的时候，文件的属主和属组是客户端正在操作 rsync 的用户身份，因为执行 rsync 程序的用户为当前用户。

- auth users 和 secrets file 这两行不是一定需要的，省略它们时将默认使用匿名连接。但是如果使用了它们，则 secrets file 的权限必须是 600。客户端的密码文件也必须是 600。

- 关于 secrets file 的权限，实际上并非一定是 600，只要满足除了运行 rsync daemon 的用户可读即可。是否检查权限的设定是通过选项 strict mode 设置的，如果设置为 false，则无需关注文件的权限。但默认是 yes，即需要设置权限。

#### Client 端配置

为了实现自动化同步文件，需要编写 rsync_data.sh 脚本文件以及 crontab 配置。

##### rsync_data.sh

```shell
#!/bin/bash

BASE_DIR=/data/rsync-data
SCRIPT_DIR=$BASE_DIR/script
DATA_DIR=$BASE_DIR/data

JOB_NAME="rsync_data"_"$1"

# max run 1 day
MAX_RUN_TIME=86400
if [[ $# > 1 ]];then
    MAX_RUN_TIME=$[ $2 * 60 ]
fi

#function to create file lock to indicate that the job is now currently running
function set_file_lock() {
    JOB_NAME=$1
    echo "lock name: " $JOB_NAME
    if [ -f $SCRIPT_DIR/$JOB_NAME.lock ]; then
        local epgid=`cat $SCRIPT_DIR/$JOB_NAME.lock`
        # Check the epgid empty or not
        # Generate one flag file if empty to alarm and exit current job
        if [ ! -n $epgid ]; then
          echo "[$(date)] [CHECK]PID_EMPTY_ERROR. Exit $JOB_NAME and remove lock file"
          remove_file_lock $JOB_NAME
          exit 3
        fi
        if [ "`ps aux | awk '$2=='$epgid' {print "RUNNING"}'`" == "RUNNING" ]; then
            local modify_time=`stat --format "%Z" $SCRIPT_DIR/$JOB_NAME.lock`
            local time_now=`date +"%s"`
            local run_time=$(($time_now - $modify_time))
            if [ $run_time -ge  $MAX_RUN_TIME ];then
                echo "[$(date)] last $JOB_NAME exists too long: $run_time, kill it."
                # kill all processes in the group
                kill -9 -- -$epgid
            else
                echo "[$(date)] $JOB_NAME is now running, will not start a new job."
                exit 2
            fi
        fi
    fi
    # Get process group id and save it.
    # crontab will fork a shell, and run rsync.
    # if we want to control rsync process, we should get rsync pid or pgid.
    # 2019-03-25: Add retry mechanism to avoid empty pid or pgid
    is_success=0
    for ((i=1;i<=10;i++)) do
      pgid=$(ps -p $$ -o pgid | sed -n '2p')
      echo -n $pgid > $SCRIPT_DIR/$JOB_NAME.lock
      local pid=`cat $SCRIPT_DIR/$JOB_NAME.lock`
      if [ -n $pid ]; then
        is_success=1
        break
      else
        sleep 1
        echo "The pid of [$(date)] $JOB_NAME is empty. Retry for $i times"
      fi
    done
    # Fail to generate pid file
    if [ ${is_success} -eq 0 ]; then
      echo "[$(date)] [GENERATE]PID_EMPTY_ERROR. Exit $JOB_NAME and remove lock file"
      remove_file_lock $JOB_NAME
      exit 3
    fi
}

#function to remove file lock to indicate that the job is now restartable
function remove_file_lock() {
    JOB_NAME=$1
    echo "lock name: " $JOB_NAME
    if [ -f $SCRIPT_DIR/$JOB_NAME.lock ]; then
        echo "remove the lock file: $JOB_NAME.lock."
        rm $SCRIPT_DIR/$JOB_NAME.lock
    fi
}

function get_rsync_server_host() {
    server_host=10.128.243.124
    echo ${server_host}
}

set_file_lock $JOB_NAME

echo "==============[Time now is: `date`] $JOB_NAME job started. =============="

# default rsync params
REMOTE_FILE_NAME=""
LOCAL_FILE_NAME=""
RSYNC_PASSWORD_FILE="/etc/rsyncd.passwd"
RSYNC_ACCOUNT=""
RSYNC_PORT="888"
RSYNC_REMOTE_PATH=""
# decide which file to rsync
if [ "$1" == "base.data" ]; then
    RSYNC_REMOTE_PATH="$(get_rsync_server_host)::testmodule"
    RSYNC_ACCOUNT="testuser"
    REMOTE_FILE_NAME="base.data"
    LOCAL_FILE_NAME="base.data"
elif [ "$1" == "base2.data" ]; then
    RSYNC_REMOTE_PATH="$(get_rsync_server_host)::testmodule"
    RSYNC_ACCOUNT="testuser"
    REMOTE_FILE_NAME="base2.data"
    LOCAL_FILE_NAME="base2.data"
else
    echo "wrong args provided: "$1
    remove_file_lock $JOB_NAME
    exit 1
fi

# remove temp file (last downloading), the rsync process may be killed by function set_file_lock.
# file data will not be removed if the rsync process is alive, so it is safe.
rm -vf ${DATA_DIR}/.${LOCAL_FILE_NAME}.*

START_TIME=$(date)
# Almost the whole file changes, so use -W option to disable delta-rsync, just rsync the whole file.
# This will reduce cpu consumption. We also make rsync lower priority of process schedule using nice.
# We may also remove -z option to disable compression for reducing more cpu consumption if needed.
echo "rsync -avzP -W --partial --partial-dir=${DATA_DIR}/rsync_partial/ --password-file=${RSYNC_PASSWORD_FILE} ${RSYNC_ACCOUNT}@${RSYNC_REMOTE_PATH}/${REMOTE_FILE_NAME} ${DATA_DIR}/${LOCAL_FILE_NAME}"
/bin/nice -n19 rsync -avzP -W --port=${RSYNC_PORT} --exclude='.*' --partial --partial-dir=${DATA_DIR}/rsync_partial/ --password-file=${RSYNC_PASSWORD_FILE} ${RSYNC_ACCOUNT}@${RSYNC_REMOTE_PATH}/${REMOTE_FILE_NAME} ${DATA_DIR}/${LOCAL_FILE_NAME}

remove_file_lock $JOB_NAME

echo "==============[Start Time: $START_TIME, End Time: `date`] $JOB_NAME job finished, now quit. ===================="
echo
echo
```

##### crontab 配置

```
#load base.data
* * * * * today=`date +\%Y\%m\%d`; /data/rsync-data/script/rsync_data.sh base.data >> /data/rsync-data/logs/rsync_data_base_$today.prod.log 2>&1

#load base2.data
* * * * * today=`date +\%Y\%m\%d`; /data/rsync-data/script/rsync_data.sh base2.data >> /data/rsync-data/logs/rsync_data_base2_$today.prod.log 2>&1
```

### 参考资料

- [https://www.cnblogs.com/f-ck-need-u/p/7220009.html](https://www.cnblogs.com/f-ck-need-u/p/7220009.html)