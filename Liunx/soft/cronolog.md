## 一、Cronolog安装

	# wget https://files.cnblogs.com/files/crazyzero/cronolog-1.6.2.tar.gz
	# tar zxvf cronolog-1.6.2.tar.gz
	# cd cronolog-1.6.2
	# ./configure
	# make
	# make install

> 查看cronolog安装后所在目录（验证安装是否成功）

	# which cronolog	
	  /usr/local/sbin/cronolog


## 二、Tomcat7以前的版本：# vim catalina.sh

> 1、注释掉（#）

	# touch “$CATALINA_BASE”/logs/catalina.out

> 2、修改tomcat bin目录下的catalina.sh文件中的两处

	org.apache.catalina.startup.Bootstrap “$@” start  \

	>> “$CATALINA_BASE”/logs/catalina.out 2>&1 &

改为:

	org.apache.catalina.startup.Bootstrap "$@" start  2>&1 \

	| /usr/local/sbin/cronolog "$CATALINA_BASE"/logs/catalina.%Y-%m-%d.out >> /dev/null &

> 3、重起Tomcat

	
## 三、Tomcat7以后的版本：  # vim catalina.sh

> 1、第一步 206

	if [ -z "$CATALINA_OUT" ] ; then

	CATALINA_OUT="$CATALINA_BASE"/logs/catalina.out

	fi

改为:

	if [ -z "$CATALINA_OUT" ] ; then

	CATALINA_OUT="$CATALINA_BASE"/logs/catalina.%Y-%m-%d.out

	fi

> 2、第二步 417

	touch "$CATALINA_OUT"

改为:

	# touch "$CATALINA_OUT"

> 3、第三步<两处> 430

	org.apache.catalina.startup.Bootstrap "$@" start \

	>> "$CATALINA_OUT"  2>&1 &

改为:

	org.apache.catalina.startup.Bootstrap "$@" start 2>&1 \

	| /usr/local/sbin/cronolog "$CATALINA_OUT" >> /dev/null &


> 完整的修改如下：

	# touch "$CATALINA_OUT"
	if [ "$1" = "-security" ] ; then
		........
	else
	eval "\"$_RUNJAVA\"" "\"$LOGGING_CONFIG\"" $LOGGING_MANAGER $JAVA_OPTS $CATALINA_OPTS \
	-Djava.endorsed.dirs="\"$JAVA_ENDORSED_DIRS\"" -classpath "\"$CLASSPATH\"" \
	-Dcatalina.base="\"$CATALINA_BASE\"" \
	-Dcatalina.home="\"$CATALINA_HOME\"" 
	-Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
	org.apache.catalina.startup.Bootstrap "$@" start 2>&1 \
	| /usr/local/sbin/cronolog "$CATALINA_BASE"/logs/catalina.%Y-%m-%d.out >>/dev/null &
	fi
