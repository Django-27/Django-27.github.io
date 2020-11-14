# 1 Hadoop Setup In The Pseudo-distributed Mode (Mac OS)
## A. brew search hadoop
## B. Go to hadoop base directory, /usr/local/Cellar/hadoop/3.1.0_1/libexec/etc/hadoop and under this folder,it requires to modify these files:
- i. hadoop-env.sh
Change from
export HADOOP_OPTS="$HADOOP_OPTS -Djava.net.preferIPv4Stack=true"
export JAVA_HOME="$(/usr/libexec/java_home)"
to
export HADOOP_OPTS="$HADOOP_OPTS -Djava.net.preferIPv4Stack=true -Djava.security.krb5.realm= -Djava.security.krb5.kdc="
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home"
- ii. core-site.xml
Then configure HDFS address and port number, open core-site.xml, input following content in configuration tag
```
<!-- Put site-specific property overrides in this file. -->
 <configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/local/Cellar/hadoop/hdfs/tmp</value>
        <description>A base for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:8020</value>
    </property>
</configuration>
```
- iii. mapred-site.xml
Configure mapreduce to use YARN, first copy mapred-site.xml.template to mapred-site.xml, and open mapred-site.xml, add
```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=/Users/Masser/hadoop</value>
    </property>
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=/Users/Masser/hadoop</value>
    </property>
    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=/Users/Masser/hadoop</value>
    </property>
</configuration>
```
- iv. hdfs-site.xml
Set HDFS default backup, the default value is 3, we should change to 1, open hdfs-site.xml, add
```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration> 
```
- v. yarn-site.xml
To walk through the org.apache.hadoop.yarn.exceptions.InvalidAuxServiceException: The auxService:mapreduce_shuffle does not exist, we need to modify the file,
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
</configuration>
```
## C. Setup a pass-phraseless SSH and Authorize the generate SSH keys:

- cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys # Finally, Enable Remote Login in (System Preference->Sharing), Just click “remote login”.
- ssh localhost # Test ssh at localhost: It should not prompt for a password

Last login: Fri Aug 17 12:08:08 2018
## D. Format the distributed file system with the below command before starting the hadoop daemons. So that we can put our data sources into the hdfs file system while performing the map-reduce job
- $ hdfs namenode -format
## E. We need to provide aliases to start and stop Hadoop Daemons. For the purpose, we edit ~/.bash_profile and add
```
alias hstart="/usr/local/Cellar/hadoop/3.1.0_1/sbin/start-all.sh"
alias hstop="/usr/local/Cellar/hadoop/3.1.0_1/sbin/stop-all.sh"

$ source ~/.zshrc
```
- i. Start hadoop using alias: $ hstart
- ii. Hadoop NameNode started on port 9870 default. Access your server on port 9870: http://localhost:9870/
- iii. Access port 8042 for getting the information about the cluster and all applications: http://localhost:8042/
- iv. Access port 9864 to get details about your Hadoop node: http://localhost:9864/datanode.html
- v. Stop hadoop using alias: $ hstop