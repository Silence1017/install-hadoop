## 简介
这是安装`hadoop`的具体流程。

## 详细流程
1. 配置`java`环境
`oracle`官网下载`jdk-8u221-linux-x64.gz`放到某个目录，此处放在`/usr/lib/java`目录。  
创建`java`的目标路径文件夹：`sudo mkdir /usr/lib/java`  
解压并命名`java`至创建的目录：`sudo tar -zxf jdk-8u221-linux-x64.gz /usr/lib/java`  
配置环境变量：`sudo gedit ~/.bashrc`  
添加以下代码：  
`export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_221`  
`export JRE_HOME=${JAVA_HOME}/jre`  
`export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib`  
`export PATH=${JAVA_HOME}/bin:$PATH`  
修改完成后保存关闭，并输入以下命令使环境变量生效：`source /etc/environment`  
配置所有用户的环境变量：`sudo gedit /etc/profile`  
在文件最后添加：  
`export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_171`  
`export JRE_HOME=${JAVA_HOME}/jre`  
`export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib`  
`export PATH=${JAVA_HOME}/bin:$PATH`  
同样，让环境变量生效：`source /etc/profile`  
查询配置是否成功：`java -version`  
注意：可以用`vim编辑器`配置环境变量：`vim /etc/profile`  
让环境变量生效：`source /etc/profile`  

2. `SSH`安装和配置
安装：`sudo apt-get install ssh`  
`ssh-keygen -t rsa`  
密码输入空，记录在`id_rsa.pub`文件里。  
`cd ~/.ssh`  
`cat id_rsa.pub >> authorized_keys`  
`ssh localhost`  

3. `Hadoop`下载安装
官网下载`hadoop-2.7.7.tar.gz`到`/usr/local/hadoop`目录下。  
同样创建目标文件夹  
解压后命名放入该目录  
验证是否成功安装：`cd /usr/local/hadoop`  
`./bin/hadoop version`  
若安装成功则会显示版本。  
//hadoop单击配置（非分布式）  
在此我们选择运行`grep`例子，将`input`文件夹中的所有文件作为输入，筛选当中符合正则表达式`dfs[a-z.]+`的单词并统计出现的次数，最后输出结果到`output`文件夹中。  
`cd /usr/local/hadoop`  
`mkdir ./input`  
`cp ./etc/hadoop/*.xml ./input`  #将配置文件作为输入文件  
`./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep ./input/ ./output 'dfs[a-z.]+'`  
`cat ./output/*`  
执行成功后如下所示，输出了作业的相关信息，输出的结果是符合正则的单词`dfsadmin`出现了1次  
注意，`Hadoop`默认不会覆盖结果文件，因此再次运行上面实例会提示出错，需要先将`./output`删除。  
`rm -r ./output`（在`/usr/local/hadoop/`目录下）  
//hadoop伪分布式配置  
修改配置文件：`sudo gedit ./etc/hadoop/core-site.xml`  
推荐使用`sudo vi ./etc/hadoop/core-site.xml`（在`/usr/local/hadoop/`目录下）  
`<configuration>`  
`   <property>`  
`<name>hadoop.tmp.dir</name>`  
`<value>file:/usr/local/hadoop/tmp</value>`  
`<description>Abase for other temporary directories.</description>`  
`</property>`  
`<property>`  
`<name>fs.defaultFS</name>`  
`<value>hdfs://localhost:9000</value>`  
`</property>`  
`</configuration>`  
`sudo gedit ./etc/hadoop/hdfs-site.xml`  
推荐使用`sudo vi ./etc/hadoop/hdfs-site.xml`  
`<configuration>`  
`<property>`  
`<name>dfs.replication</name>`  
`<value>1</value>`  
`</property>`  
`<property>`  
`<name>dfs.namenode.name.dir</name>`  
`<value>file:/usr/local/hadoop/tmp/dfs/name</value>`  
`</property>`  
`<property>`  
`<name>dfs.datanode.data.dir</name>`  
`<value>file:/usr/local/hadoop/tmp/dfs/data</value>`  
`</property>`  
`<property>`  
`<name>dfs.http.address</name>`  
`<value>localhost:50070</value>`  
`</property>`  
`</configuration>`  
修改`java`环境变量：`sudo vi ./etc/hadoop/hadoop-env.sh`  
`export JAVA_HOME=/usr/lib/java/jdk1.8.0_221`  
执行`namenode`的格式化：`sudo ./bin/hdfs namenode -format`  
开启`namenode`和`datanode`守护进程：`sudo ./sbin/start-dfs.sh`  
启动完成后可以通过命令`jps`来判断是否成功启动  
关闭命令：`sbin/stop-dfs.sh`  
可以打开`http://localhost:50070/`查看 `NameNode` 和 `Datanode` 信息，还可以在线查看 `HDFS` 中的文件。  
遇到问题：  
`root@localhost's password:localhost:permission denied,please try again`   
开启守护进程时一直出现密码无效的情况  
解决方法：  
1.安装`open ssh`：`sudo apt-get install openssh-server`  
2.修改`root`密码：`sudo passwd root`  
3.修改配置文件，允许`root`用户通过`ssh`登陆：`sudo vi /etc/ssh/sshd_config`  
找到：`#PermitRootLogin prohibit-password`，在其下面添加  
`PermitRootLogin yes`  
4.重启服务：`sudo service ssh restart`  
给`hadoop`配置环境变量：`sudo vim /etc/bash.bashrc`  
在末尾添加：  
`export JAVA_HOME=/usr/lib/java/jdk1.8.0_221`  
`export JRE_HOME=${JAVA_HOME}/jre`  
`export HADOOP_HOME=/usr/local/hadoop`  
`export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib`  
`export PATH=${JAVA_HOME}/bin:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:$PATH`  
然后执行`source`命令：`source /etc/bash.bashrc`  
最后运行`hadoop version`即可查看版本。  
链接：https://blog.csdn.net/weixin_42001089/article/details/81865101  
