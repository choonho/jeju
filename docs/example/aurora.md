
~~~bash
apt-get update
apt-get -y install unzip
~~~

~~~bash
git clone http://git-wip-us.apache.org/repos/asf/aurora.git
cd aurora
./gradlew distZip
~~~

# Installing Aurora

~~~bash
cd aurora
unzip dist/distributions/aurora-scheduler-*.zip -d /usr/local
ln -nfs "$(ls -dt /usr/local/aurora-scheduler-* | head -1)" /usr/local/aurora-scheduler
~~~

