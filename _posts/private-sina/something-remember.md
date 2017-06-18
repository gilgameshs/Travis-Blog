mysql -u 用户名 -p密码 -h 服务器IP地址 -P 服务器端MySQL端口号 -D 数据库名

注意：

    (1)服务器端口标志-P一定要大些以区别于用户-p,如果直接连接数据库标志-D也要大写；

    (2)如果要直接输入密码-p后面不能留有空格如-pmypassword;

    (3)命令结束段没有';'分号。

线上地址登录命令：mysql -uea_report -hm6472i.mars.grid.sina.com.cn -P6472 -Dea_report_service -p9eec06d6aa9c5ec

tomcat调试端口设置：
	CATALINA_OPTS="-server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=6662"
	JAVA_OPTS='-Xms1024m -Xmx4096m -XX:MaxNewSize=1024m -XX:MaxPermSize=1024m'



if [ -d "gina-admanager" ]; then
    rm -rf gina-admanager
    rm -rf gina-admanager.war
fi

chown -hR guiqiang gina-admanager.war
chgrp -hR gina gina-admanager.war

CATALINA_OPTS="-server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=6662"
JAVA_OPTS=-Xms1024m -Xmx4096m -XX:MaxNewSize=1024m -XX:MaxPermSize=1024m




<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="10\.236\.66\.116"/>


nexus验证：
<server>
         <id>Releases</id>
         <username>deployment</username>
         <password>sinawuqiang1</password>
</server>


Jenkins slave.jar  需要从 http://yourserver:port/jnlpJars/slave.jar 下载，49线上的jenkins是在tomcat下的，所以地址有变化，为 http://yourserver:port/jenkins/jnlpJars/slave.jar