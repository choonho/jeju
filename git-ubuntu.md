# Git Server with Ubuntu

Keyword | Value
----    | ----
GIT_DIR | /var/git
SAMPLE  | sample.git

## install git

Install git and gitweb

~~~bash
apt-get update
apt-get -y install git gitweb

cd ${GIT_DIR}
git init --bare --shared ${SAMPLE}
~~~

edit /etc/apache2/sites-available/000-default.conf

~~~text
<VirtualHost *:80>
	DocumentRoot ${GIT_DIR} 
	<Directory "${GIT_DIR}"> 
		Allow from All
		Options +ExecCGI
		AllowOverride All
	</Directory>
	SetEnv GIT_PROJECT_ROOT ${GIT_DIR}
	SetEnv GIT_HTTP_EXPORT_ALL
	ScriptAlias /git/ /usr/lib/git-core/git-http-backend/
</VirtualHost>
~~~

# Configure gitweb

## Update gitweb.conf

edit /etc/gitweb.conf

~~~text
# path to git projects (<project>.git)
$projectroot = "${GIT_DIR}";

# directory to use for temp files
$git_temp = "/tmp";
~~~

## gitweb under Apache - put the following configuration

edit /etc/apache2/conf-available/gitweb.conf

~~~text
Alias /gitweb /usr/share/gitweb
<Directory /usr/share/gitweb>
  Options +FollowSymLinks +ExecCGI
  AddHandler cgi-script .cgi
</Directory>
~~~

## enable gitweb.conf in apache

~~~bash
ln -s /etc/apache2/conf-available/gitweb.conf /etc/apache2/conf-enabled/gitweb.conf
~~~

Enable cgi, so that the gitweb script is run and not displayed/downloaded from web browser.

~~~bash
a2enmod cgi
service apache2 restart
~~~
