# Cosbench install

Keyword | Value
----    | ----
VER     | 0.4.2.c3
TARGET  | /usr/local/bin

# Download COSBench

~~~bash
apt-get update
apt-get -y install wget unzip openjdk-7-jre
wget -O ${TARGET}/${VER}.zip https://github.com/intel-cloud/cosbench/releases/download/v${VER}/${VER}.zip
cd ${TARGET}

unzip ${VER}.zip;
ln -s ${TARGET}/${VER} ${TARGET}/cosbench

cd cosbench
chmod +x *.sh
~~~

# Clean source

~~~bash
rm -f ${TARGET}/${VER}.zip
~~~
