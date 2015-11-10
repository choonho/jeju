# Environment

E_IP is your host IP.

Keyword | Value
---- | ----
E_IP | 127.0.0.1


# Install Elastic Search

## download and install elasticsearch

~~~bash
wget https://download.elasticsearch.org/elasticsearch/release/org/elasticsearch/distribution/deb/elasticsearch/2.0.0/elasticsearch-2.0.0.deb

dpkg -i elasticsearch-2.0.0.deb
~~~

## Update node name

edit /etc/elasticsearch/elasticsearch.yml

~~~text
node.name: ${HOSTNAME}
network.bind_host: 0.0.0.0
~~~

## Install module

~~~bash
cd /usr/share/elasticsearch
bin/plugin install mobz/elasticsearch-head
~~~

## Restart elastic search

~~~bash
service elasticsearch restart
~~~

# Install Logstash

## download and install logstash

~~~bash
wget https://download.elastic.co/logstash/logstash/packages/debian/logstash_2.0.0-1_all.deb
dpkg -i logstash_2.0.0-1_all.deb
~~~

# Install Kibana

## download and install Kibana

~~~bash
wget -O /root/kibana-4.2.0-linux-x64.tar.gz https://download.elastic.co/kibana/kibana/kibana-4.2.0-linux-x64.tar.gz
cd /root
tar -zxvf kibana-4.2.0-linux-x64.tar.gz
mv kibana-4.2.0-linux-x64 /opt/
cd /opt
ln -s kibana-4.2.0-linux-x64 kibana
~~~

edit /opt/kibana/config/kibana.yml

~~~text
elasticsearch.url: "http://${E_IP}:9200"
~~~

## update init script

~~~bash
apt-get update
apt-get -y install curl
~~~

edit /etc/init.d/kibana

~~~text
#!/bin/sh
#
# /etc/init.d/kibana -- startup script for kibana4
#
### BEGIN INIT INFO
# Provides:          kibana
# Required-Start:    $network $remote_fs $named
# Required-Stop:     $network $remote_fs $named
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Starts kibana
# Description:       Starts kibana using start-stop-daemon
### END INIT INFO

#configure this with wherever you unpacked kibana:
KIBANA_BIN=/opt/kibana/bin

NAME=kibana4
PID_FILE=/var/run/$NAME.pid
PATH=/bin:/usr/bin:/sbin:/usr/sbin:$KIBANA_BIN
DAEMON=$KIBANA_BIN/kibana
DESC="Kibana4"
USER=root
ES_HOST="${E_IP}"

if [ `id -u` -ne 0 ]; then
	echo "You need root privileges to run this script"
	exit 1
fi

. /lib/lsb/init-functions

if [ -r /etc/default/rcS ]; then
	. /etc/default/rcS
fi

case "$1" in
  start)
        log_daemon_msg "Starting $DESC"

        pid=`pidofproc -p $PID_FILE kibana`
        if [ -n "$pid" ] ; then
                log_begin_msg "Already running."
                echo "Already running."
                log_end_msg 0
                exit 0
        fi

        # Check if elasticsearch is up:
        elasticsearchDOWN=true
        timer=1
        # Attempt to start, wait for elasticsearch, if it doesn't complete in 300 seconds (30*10), exit
        while [ "$elasticsearchDOWN" = true ] && [ $timer -lt 10 ]; do
                response=$(curl -w "%{http_code}\\n" "http://$ES_HOST:9200")

                if (echo "$response" | grep -e "\"200"); then
                # Start Daemon
                        elasticsearchDOWN=false
                        start-stop-daemon --start --user "$USER" -c "$USER" --pidfile "$PID_FILE" --make-pidfile --background --exec $DAEMON
                        echo $elasticsearchDOWN
                else
                        log_begin_msg "Elasticsearch not running, waiting..."
                        echo "Elasticsearch not running, waiting..."
                        log_end_msg 0
                        sleep 10
                fi
                timer=$((timer+1))
        done
        log_end_msg $?
        ;;
  stop)
	log_daemon_msg "Stopping $DESC"

	if [ -f "$PID_FILE" ]; then
		start-stop-daemon --stop --user "$USER" -c "$USER" --pidfile "$PID_FILE" \
			--retry=TERM/20/KILL/5 >/dev/null
		if [ $? -eq 1 ]; then
			log_progress_msg "$DESC is not running but pid file exists, cleaning up"
		elif [ $? -eq 3 ]; then
			PID="`cat $PID_FILE`"
			log_failure_msg "Failed to stop $DESC (pid $PID)"
			exit 1
		fi
		rm -f "$PID_FILE"
	else
		log_progress_msg "(not running)"
	fi
	log_end_msg 0
	;;
  status)
	status_of_proc -p $PID_FILE kibana kibana && exit 0 || exit $?
    ;;
  restart|force-reload)
	if [ -f "$PID_FILE" ]; then
		$0 stop
		sleep 1
	fi
	$0 start
	;;
  *)
	log_success_msg "Usage: $0 {start|stop|restart|force-reload|status}"
	exit 1
	;;
esac

exit 0
~~~

update permission

~~~bash
chmod 755 /etc/init.d/kibana
~~~
