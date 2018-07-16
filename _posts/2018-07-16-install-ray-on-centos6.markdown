---
layout: post
title: "Install Ray on CentOS6"
date: 2018-07-16 20:40:00 -0700
categories: daily linux
---
I summarize several issues encountered when I am attempting to install Ray on CentOS6.

### How to check CentOS version?
`rpm --query centos-release`

### What's yum equivalence of `apt-get update`?
`yum check-update`

### What's yum equivalence of `apt-get install build-essential`?
`yum groupinstall 'Development Tools'`

### How to install latest Python on CentOS?
`yum` utility relies on Python2.6 on CentOS6 and will be broken if the defulat Python interpreter is updated or replaced. The trick is to install new version at another location (e.g., */usr/local/*) so that they can live side-by-side with the system version.

Pre-requisites:
```
# Start by making sure your system is up-to-date:
yum update
# Compilers and related tools:
yum groupinstall -y "development tools"
# Libraries needed during compilation to enable all features of Python:
yum install -y zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel expat-devel
# If you are on a clean "minimal" install of CentOS you also need the wget tool:
yum install -y wget
```
Download and installation:
```
# Python 2.7.14:
wget http://python.org/ftp/python/2.7.14/Python-2.7.14.tar.xz
tar xf Python-2.7.14.tar.xz
cd Python-2.7.14
./configure --prefix=/usr/local --enable-unicode=ucs4 --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"
make && make altinstall

# Python 3.6.3:
wget http://python.org/ftp/python/3.6.3/Python-3.6.3.tar.xz
tar xf Python-3.6.3.tar.xz
cd Python-3.6.3
./configure --prefix=/usr/local --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"
make && make altinstall
```
strip symbols from the shared library to reduce the memory footprint:
```
# Strip the Python 2.7 binary:
strip /usr/local/lib/libpython2.7.so.1.0
# Strip the Python 3.6 binary:
strip /usr/local/lib/libpython3.6m.so.1.0
```
install *pip* related:
```
# First get the script:
wget https://bootstrap.pypa.io/get-pip.py

# Then execute it using Python 2.7 and/or Python 3.6:
python2.7 get-pip.py
python3.6 get-pip.py

# With pip installed you can now do things like this:
pip2.7 install [packagename]
pip2.7 install --upgrade [packagename]
pip2.7 uninstall [packagename]
```

details can be found [here](https://danieleriksson.net/2017/02/08/how-to-install-latest-python-on-centos/)

### How to `pip install` with bad network condition?
`pip install --default-timeout=100 foo`

### How to upgrade gcc and cpp?
CentOS6 contain 4.7 while Ray requires 4.8.

```
cd /etc/yum.repos.d
wget http://people.centos.org/tru/devtools-2/devtools-2.repo 
yum --enablerepo=testing-devtools-2-centos-6 install devtoolset-2-gcc devtoolset-2-gcc-c++
export CC=/opt/rh/devtoolset-2/root/usr/bin/gcc
export CPP=/opt/rh/devtoolset-2/root/usr/bin/cpp
export CXX=/opt/rh/devtoolset-2/root/usr/bin/c++
```

details can be found [here](https://superuser.com/questions/381160/how-to-install-gcc-4-7-x-4-8-x-on-centos)

In addition, on CentOS6, we have to `yum install devtoolset-2-binutils-devel`

### How to download large third-party project for Ray?
When the network condition is bad, `pip2.7 install -e . --verbose` always hang with RPC error.
A trick is to download the required third-party project from github seperately and unfold them to corresponding directory according to the log of Ray, e.g., `cp -r /home/admin/jones.wz/catapult /home/admin/jones.wz/ray/thirdparty/scripts/..//pkg/catapult`.
Another problem is that some Ray installation scripts hard coded the `python2` command which refers to the default Python of CentOS. We can find such scripts from the log of Ray and replace `python2` by `python2.7`, e.g., `vi /home/admin/jones.wz/ray/thirdparty/scripts/build_ui.sh`
