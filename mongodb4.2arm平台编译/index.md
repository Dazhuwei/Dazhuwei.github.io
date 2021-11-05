# mongodb4.2 ARM平台编译



# mongodb4.2 ARM平台编

## 1.Gcc和依赖软件下载

```shell
wget https://gcc.gnu.org/pub/gcc/infrastructure/gmp-6.1.0.tar.bz2

wget https://gcc.gnu.org/pub/gcc/infrastructure/mpfr-3.1.4.tar.bz2

wget https://gcc.gnu.org/pub/gcc/infrastructure/mpc-1.0.3.tar.gz

wget https://gcc.gnu.org/pub/gcc/infrastructure/isl-0.18.tar.bz2

wget https://gcc.gnu.org/pub/gcc/releases/gcc-9.2.0/gcc-9.2.0.tar.gz
```

## 2.下载mongodb源码包

```shell
wget https://fastdl.mongodb.org/src/mongodb-src-r4.2.8.tar.gz
```

 将1、2的软件包上传到/opt目录下

## 3.安装gcc和依赖

### 1）检查gcc版本

```shell
gcc -version
```

### 2）安装依赖

```shell
yum install -y bzip2
```



### 3）上传gcc-9.2.0.tar.gz到/opt目录解压，将依赖包放入解压出来的gcc目录

```shell
cd /opt

tar xf gcc-9.2.0.tar.gz

cp isl-0.18.tar.bz2 mpc-1.0.3.tar.gz gmp-6.1.0.tar.bz2 mpfr-3.1.4.tar.bz2 gcc-9.2.0

cd gcc-9.2.0

./contrib/download_prerequisites
```



### 4）执行编译安装

```shell
mkdir gcc-build-9.2.0

cd gcc-build-9.2.0

../configure --enable-checking=release --enable-language=c,c++ --disable-multilib --prefix=/usr

make -j`cat /proc/cpuinfo| grep "processor"| wc -l` //8核大概需要40分钟

make install
```



### 5）查看gcc版本

```shell
gcc -version
```

## 4.安装系统依赖包

```shell
yum install libcurl-devel libyaml libyaml-devel python-setuptools zlib-devel libffi-devel openssl openssl-devel
```



## 5.安装python

### 1)升级Python至3.7版本。

Python3.7安装需要花费较长时间，请耐心等待。

```shell
yum install wget -y

yum install -y zlib* openssl*

cd /usr/local/src

wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tgz

tar -zxvf Python-3.7.0.tgz

cd Python-3.7.0

mkdir /usr/local/python37

./configure --prefix=/usr/local/python37 --enable-optimizations

make -j8 && make install
```

ps：如果出现上面的错误，再装一下依赖

```shell
yum install libffi-devel -y
```

### 2)设置Python3.7环境变量。

```shell
cp /usr/local/python37/bin/python3.7 /usr/bin

mv /usr/bin/python3.7 /usr/bin/python3

mkdir /usr/local/lib/python3.7

cp /usr/local/python37/lib/* /usr/local/lib/python3.7/ -rf

ldconfig
```

### 3)升级setuptools到新版本。

```shell
cd /usr/local/src/

wget --no-check-certificate https://pypi.python.org/packages/source/s/setuptools/setuptools-19.6.tar.gz

tar -zxvf setuptools-19.6.tar.gz

cd setuptools-19.6

python3 setup.py build

python3 setup.py install
```



### 4)安装python模块

```shell
/usr/local/python37/bin/pip3 install psutil pyyaml cheetah3
```



## 6.安装mongodb

```shell
tar xf mongodb-src-r4.2.3.tar.gz -C /opt

cd mongodb-src-r4.2.3

mkdir /usr/local/mongodb

//指定安装路径--prefix=/usr/local/mongodb

python3 buildscripts/scons.py --prefix=/usr/local/mongodb install MONGO_VERSION=4.2.3 core CFLAGS="-march=armv8-a+crc -mtune=generic" -j8 --disable-warnings-as-errors

//8C核大概40分钟

//删除调试信息，最后也可以将/home/mongodb/mongodb-src-r4.2.3 目录删除

cd /usr/local/mongodb

strip mongos

strip mongod

strip mongo
```



## 7.测试

```shell
cd /home/mongodb/mongodb/

mkdir -p data/db

nohup /home/mongodb/mongodb/bin/mongod --dbpath /home/mongodb/mongodb/data/db &

/home/mongodb/mongodb/bin/mongo

show db
```

**参考文档**

https://bbs.huaweicloud.com/blogs/155007


