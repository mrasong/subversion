## linux 一键安装subversion
@2012-05-25


### 安装方法 ###
```
chmod +x subversion-1.6.18.sh && ./subversion-1.6.18.sh
```

> 下载：[subversion-1.6.18.sh][1]

### 使用方法 ###
```
/etc/init.d/subversion {start|stop|restart|reload}
```



### 修改配置 ###

> /home/data/svn/main/conf/svnserve.conf
```
    [general]
    ### 匿名访问
    anon-access = none
    # auth-access = write

    ### 用户文件
    password-db = passwd

    ### 权限文件
    authz-db = authz

    ### svn 名称
    realm = MY SVN

    [sasl]
    ### 这些可以默认
    # use-sasl = true
    # min-encryption = 0
    # max-encryption = 256
```
### 添加用户 ###

> /home/data/svn/conf/passwd 

```
[users]
### 用户名 = 密码
zhangsan = password
lisi = password2
### .....
```

### 设置权限 ###

>  /home/data/svn/conf/authz

```
[aliases]
# joe = /C=XZ/ST=Dessert/L=Snake City/O=Snake Oil, Ltd./OU=Research Institute/CN=Joe Average

[groups]
# harry_and_sally = harry,sally
# harry_sally_and_joe = harry,sally,&joe

[/]
### 根目录下 zhangsan 可读写， lisi 只读
zhangsan = rw
lisi = r

# [/foo/bar]
# harry = rw

### 别名用户 joe
# &joe = r

### 所有用户
# * =

# [repository:/baz/fuz]
### 用户组
# @harry_and_sally = rw
# * = r
```


### 源码 ###
```
#!/bin/bash
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

SVN_PATH=/usr/local/subversion
SVN_DATA=/home/data/svn
SVNSERVE=/etc/init.d/subversion
# Check if user is root
if [ $(id -u) != "0" ]; then
    echo "Error: You must be root to run this script!"
    exit 1
fi

clear
printf "=======================================================================\n"
printf "Install Subversion-1.6.18 Written by MrAsong \n"
printf "=======================================================================\n"
printf "Author:   MrAsong \n"
printf "Email:    i@mrsaong.com \n"
printf "Home:     http://mrasong.com/ \n"
printf "Version:  2012-05-25 \n"
printf "\n"
printf "=======================================================================\n"
cur_dir=$(pwd)

    get_char()
    {
    SAVEDSTTY=`stty -g`
    stty -echo
    stty cbreak
    dd if=/dev/tty bs=1 count=1 2> /dev/null
    stty -raw
    stty echo
    stty $SAVEDSTTY
    }
    echo ""
    echo "Press any key to start install Subversion..."
    char=`get_char`

printf "=========================== install Subversion ======================\n"

wget -c http://subversion.tigris.org/downloads/subversion-1.6.18.tar.gz
wget -c http://subversion.tigris.org/downloads/subversion-deps-1.6.18.tar.gz

tar zxvf subversion-1.6.18.tar.gz
tar zxvf subversion-deps-1.6.18.tar.gz
cd subversion-1.6.18

./configure --prefix=$SVN_PATH  --without-berkeley-db 
make && make install

rm -f $SVNSERVE

cat >$SVNSERVE<<eof
#! /bin/bash
# chkconfig: 2345 99 99
# Subversion:    Subversion Daemon 
# Description: Startup script for subversion on Debian. Place in /etc/init.d and
# run 'update-rc.d -f subversion defaults', or use the appropriate command on your
# distro. For CentOS/Redhat run: 'chkconfig --add subversion'

SVNPATH=$SVN_DATA
SVN=$SVN_PATH/bin/svnserve

RETVAL=0
prog="Subversion By MRASONG"

start () {
    echo -n \$"Starting \$prog: "
    \$SVN -d -r \$SVNPATH
    RETVAL=\$?
    echo
    [ \$RETVAL -eq 0 ]
}
stop () {
    echo -n \$"Stopping \$prog: "
    killall \$SVN
    RETVAL=\$?
    echo
    [ \$RETVAL -eq 0 ]
}

restart () {
    stop
    start
}


# See how we were called.
case "\$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    restart|reload)
        restart
        ;;
    *)
    echo \$"Usage: \$0 {start|stop|restart|reload}"
    exit 1
esac

exit \$?

eof
chmod +x $SVNSERVE

mkdir -p $SVN_DATA
$SVN_PATH/bin/svnadmin create $SVN_DATA/main #create main project


if [ -s /etc/debian_version ]; then
update-rc.d -f subversion defaults
elif [ -s /etc/redhat-release ]; then
chkconfig --level 345 subversion on
fi

#Starting Subversion...
$SVNSERVE start

printf "===================== install Subversion completed =====================\n"
printf "Install Subversion completed,enjoy it!\n"
printf "For more information please visit http://mrasong.com/a/subversion \n"
printf "=======================================================================\n"
```
