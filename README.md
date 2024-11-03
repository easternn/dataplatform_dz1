# Настройка HDFS

## Предварительные требования
- Доступ ко всем нодам с пользователем team через пароль и знание пароля для пользователя team на всех нодах.

## Шаг 1: Настройка названий нод, чтобы не вводить каждый раз ip адрес. 
В файл /etc/hosts на team@team-5-jn, team@team-5-nn, team@team-5-dn-00, team@team-5-dn-01 вставить следующие строчки в конец файла:
```
192.168.1.22 team-5-jn
192.168.1.23 team-5-nn
192.168.1.24 team-5-dn-0 
192.168.1.25 team-5-dn-1
```

## Шаг 2: Создание пользователя hadoop
Цель: на 3-х внутренних нодах есть пользователь hadoop со сложным паролем без sudo прав

На хостах team@team-5-nn, team@team-5-dn-00, team@team-5-dn-01 выполнить слeдующую команду:
sudo adduser hadoop
В процессе нужно задать пароль для нового пользователя, нужно задать сложный пароль.
Для переключения из пользователя team на пользователя hadoop на любой из 3-х внутренниз нод нужно выполнить команды
su hadoop
-- Ввести пароль для пользователя hadoop
cd ~
Для того, чтобы переключиться обратно на пользователя team нужно выполнить аналогичные шаги

## Шаг 3. Создание и распространение ssh ключей.
На нодах hadoop@team-5-nn, hadoop@team-5-dn-00, hadoop@team-5-dn-01 выполнить слeдующие команды:
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
ssh-copy-id hadoop@team-5-nn
ssh-copy-id hadoop@team-5-dn-0
ssh-copy-id hadoop@team-5-dn-1

В процессе нужно будет ввести пароли для пользователя hadoop на хостах team-5-nn, team-5-dn-00, team-5-dn-01 и несколько раз ввести "yes".

После выполнения данных команд подключение через ssh между данными трема хостами не должно требовать введения пароля.


## Проверка
После выполнения Шагов 1, 2, 3. Должны быть:
1. 3 пользователя hadoop на нодах team-5-nn, team-5-dn-00, team-5-dn-01 без sudo прав
2. Доступ по ssh без пароля между нодами team-5-nn, team-5-dn-00, team-5-dn-01 на пользователе hadoop

## Шаг 4: Установка Hadoop

На узлах hadoop@team-5-nn, hadoop@team-5-dn-00, hadoop@team-5-dn-01 (устанавливаем в ~ то есть в домашней директории):

```bash
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz
tar -xzvf hadoop-3.4.0.tar.gz
```

## Шаг 5: Настройка переменных окружения

На всех узлах добавьте в `~/.bashrc`:

```bash
export JAVA_HOME=$(readlink -f /usr/bin/java | sed 's:bin/java::')
export HADOOP_HOME=/home/hadoop/hadoop-3.4.0
export HADOOP_INSTALL=/home/hadoop/hadoop-3.4.0
export HADOOP_COMMON_HOME=/home/hadoop/hadoop-3.4.0
export HADOOP_HDFS_HOME=/home/hadoop/hadoop-3.4.0
export HADOOP_CONF_DIR=/home/hadoop/hadoop-3.4.0/etc/hadoop
export PATH=$PATH:/home/hadoop/hadoop-3.4.0/sbin:/home/hadoop/hadoop-3.4.0/bin

source ~/.bashrc
```

## Шаг 6: Настройка Hadoop

### Все конфиг файлы лежат в `/home/hadoop/hadoop-3.4.0/etc/hadoop`

1. Установите JAVA_HOME в `hadoop-env.sh`:

   ```bash
   echo "export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64" >> $HADOOP_CONF_DIR/hadoop-env.sh
   ```

2. Настройте `core-site.xml` на всех узлах:

   ```xml
   <configuration>
     <property>
       <name>fs.defaultFS</name>
       <value>hdfs://192.168.1.23:9000</value>
     </property>
   </configuration>
   ```

3. Настройте `hdfs-site.xml` на NameNode:

   ```xml
   <configuration>
     <property>
       <name>dfs.replication</name>
       <value>3</value>
     </property>
     <property>
       <name>dfs.namenode.name.dir</name>
       <value>file:///home/hadoop/hadoop-3.4.0/hdfs/namenode</value>
     </property>
     <property>
       <name>dfs.datanode.data.dir</name>
       <value>file:///home/hadoop/hadoop-3.4.0/hdfs/datanode</value>
     </property>
   </configuration>
   ```

4. Настройте `hdfs-site.xml` на DataNodes:

   ```xml
   <configuration>
     <property>
       <name>dfs.replication</name>
       <value>3</value>
     </property>
     <property>
       <name>dfs.datanode.data.dir</name>
       <value>file:///home/hadoop/hadoop-3.4.0/hdfs/datanode</value>
     </property>
   </configuration>
   ```

5. Настройте файл `workers` на всех узлах (/home/hadoop/hadoop-3.4.0/etc/hadoop/workers), нужно добавить остальные датаноды:

   ```
   team-5-nn
   team-5-dn-0
   team-5-dn-1
   ```

## Шаг 7: Создание директорий для HDFS

Только в NameNode:

```bash
mkdir -p /home/hadoop/hadoop-3.4.0/hdfs/namenode
```

Во всех нодах (NameNode и DataNodes):

```bash
mkdir -p /home/hadoop/hadoop-3.4.0/hdfs/datanode
```

## Шаг 8: Форматирование NameNode

На NameNode:

```bash
hdfs namenode -format
```

## Шаг 9: Запуск сервисов Hadoop

На NameNode:

```bash
start-dfs.sh
```

## Шаг 10: Проверка установки

Проверка запущенных сервисов:

```bash
jps
```

Для Namenode должны увидеть
- 21922 Jps
- 21603 NameNode
- 21787 SecondaryNameNode
- 19728 DataNode

Для остальных Datanode
- 19728 DataNode
- 19819 Jps


Проверка версии Hadoop:

```bash
hadoop version
```

## Шаг 11: Настройка WEB интерфейсов.

