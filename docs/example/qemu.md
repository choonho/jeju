# About QEMU

# Environment

Keyword  | Value
-----    | -----
QEMU_GIT | git://git.qemu.org/qemu.git
QEMU_DIR | qemu

# Installation

## Download source

~~~bash
apt-get -y install git
git clone ${QEMU_GIT}
apt-get -y build-dep qemu
cd ${QEMU_DIR}
./configure --enable-debug --target-list="x86_64-softmmu" --enable-kvm
make -j 8
make install
~~~

# Usage

