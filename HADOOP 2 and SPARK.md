# Hadoop 2

### Update system
```sh
$ sudo apt update
$ sudo apt full-upgrade
```

### Install java

```sh
$ sudo apt-get install openjdk*
```

### Install openssh server and client
```sh
$ sudo apt-get install openssh-server openssh-client
```

### Create user for hadoop (you can name it as you like)
```sh
$ sudo adduser hadoop
```
Enter user's password

Login new user
```sh
$ su - hadoop
```
Enter user's password

### Generate SSH's key
```sh
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
```

### Test connect with SSH
```sh
$ ssh localhost
```
If the questions are answered YES
After connecting, the server will introduce itself and give some recommendations
```sh
exit
```

### Download Hadoop
Visit the website [Hadoop releases](https://hadoop.apache.org/releases.html) and copy the link to the latest version of Hadoop 2, and then paste it into the command below
```sh
wget https://archive.apache.org/dist/hadoop/common/hadoop-2.10.0/hadoop-2.10.0.tar.gz
```
Unpacking
```sh
tar -xzvf hadoop-2.10.0.tar.gz
mv hadoop-2.10.0 hadoop
```

### Configure the environment
If you use a different environment, such as zsh, then you need to use the changes in this environment
This document describes bash
```sh
vim ~/.bashrc
```

Instead of Vim, you can use any console editor, such as nano.
This instruction will describe an example for VIM

The file opens, click I (switch VIM to edit mode) and write to the end of the file.
```sh
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre/
export HADOOP_HOME=/home/hadoop/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
```
press esc (switch VIM to command mode) and type command Write and Exit:
```sh
:wq
```
Later in the document, in order not to describe all VIM commands, I will use the word Edit

Restart the environment
```sh
source ~/.bashrc
```

Go to the unpacking directory
cd hadoop/etc/hadoop
Edit hadoop-env.sh
```sh
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre/
export HADOOP_CONF_DIR=${HADOOP_CONF_DIR:-"/home/hadoop/hadoop/etc/hadoop"}
```

Edit core-site.xml  
```sh
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>var/app/hadoop/tmp</value>
    </property>
</configuration>
```

Edit hdfs-site.xml
```sh
<configuration>
<property>
 <name>dfs.replication</name>
 <value>1</value>
</property>

<property>
  <name>dfs.name.dir</name>
    <value>file:///home/hadoop/hadoopdata/hdfs/namenode</value>
</property>

<property>
  <name>dfs.data.dir</name>
    <value>file:///home/hadoop/hadoopdata/hdfs/datanode</value>
</property>
</configuration>
```

Using the mapred-site template, we create a configuration file
```sh
cp mapred-site.xml.template mapred-site.xml
```

Edit mapred-site.xml
```sh
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

Edit yarn-site.xml
```sh
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>localhost:8025</value> </property>
        <property> <name>yarn.resourcemanager.scheduler.address</name>
        <value>localhost:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>localhost:8050</value>
    </property>
</configuration>

# запускаем это дерьмо
cd $HADOOP_HOME/sbin/
hdfs namenode -format
./start-all.sh   (если предупреждение пишем yes)
```

### Testing the configuration

Checking the installation of Hadoop modules
```sh
jps
```
You should see a list of modules
```sh
20035 SecondaryNameNode (It doesn't work because there is no second node)
19782 DataNode
21671 Jps
20343 NodeManager
19625 NameNode
20187 ResourceManager
```

# We check the performance
```sh
hadoop version
```
You should see a verion Hadoop

Hadoop modules must be available in the browser by IP or by local address:
- http://localhost:50070/
- http://localhost:8088/
- http://localhost:8042/

Make the HDFS directoire
```sh
$HADOOP_HOME/bin/hdfs dfs -mkdir /user
$HADOOP_HOME/bin/hdfs dfs -mkdir /user/hadoop
```

Hadoop command memo: 
- https://riptutorial.com/ru/hadoop/example/20034/%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%8B-hadoop-v2


# Install Apache Spark

Download spark
```sh
curl -O https://downloads.apache.org/spark/spark-2.4.5/spark-2.4.5-bin-hadoop2.7.tgz
```

Unpacking and installing

```sh
tar xvf spark-2.4.5-bin-hadoop2.7.tgz

sudo mv spark-2.4.5-bin-hadoop2.7/ /opt/spark
```


### Configure the environment

Edit  ~/.bashrc
```sh
export SPARK_HOME=/opt/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
```

Launching Spark
```sh
SPARK_HOME/sbin/start-master.sh
```

Checking the launch 
```sh
ss -tunelp | grep 8080
```
Or you can go to the browser by following the link
```sh
spark://ubuntu:7077.
```

Launching slave:
```sh
SPARK_HOME/sbin/start-slave.sh spark://ubuntu:7077
```


### Starting the Spark Shell
```sh
SPARK_HOME/bin/spark-shell
```

If you plan to work through Python you can open the built in handler:
```sh
SPARK_HOME/bin/pyspark
```

For a stop:
```sh
SPARK_HOME/sbin/stop-slave.sh
SPARK_HOME/sbin/stop-master.sh
```
