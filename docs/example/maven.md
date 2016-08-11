# Install Maven from source

Keyword | Value
---- | ----
VER  | 3.3.9

## Download Source

~~~bash
wget http://apache.mirror.cdnetworks.com/maven/maven-3/${VER}/binaries/apache-maven-${VER}-bin.tar.gz

tar -zxvf apache-maven-${VER}-bin.tar.gz -C /usr/local/

cd /usr/local
ln -s apache-maven-${VER} maven
~~~

## Update profile

edit /etc/profile.d/maven.sh

~~~text
export M2_HOME=/usr/local/maven
export M2=$M2_HOME/bin
PATH=$M2:$PATH
~~~

