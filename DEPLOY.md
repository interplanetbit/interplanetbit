# Demo Deployment

You can deploy ALL exchange components on a single machine with 32G memory, at least 20G HD.

## Create Ubuntu Server

Create EC2 instance with image `Ubuntu Server 18.04 LTS (HVM), SSD Volume Type`, 64bit x86.

Public IP: Must be assigned

Security Group Setting:

- Inbounds:
  - Allow TCP 22: 0.0.0.0/32
  - Allow TCP 80: 0.0.0.0/32
- Outbounds:
  - Allow all tranfic.

User: `ubuntu`, with `sudo` privilege and no password required.

Update and restart Ubuntu:

```
$ sudo apt update
$ sudo apt upgrade
$ sudo reboot
```

#### Install OpenJDK 11

Install OpenJDK 11 by `apt`:

```
$ sudo apt install openjdk-11-jre-headless
```

#### Install Nginx

Install Nginx by `apt`:

```
$ sudo apt install nginx
```

#### Install MySQL 5.7

Install MySQL 5.7 by `apt`:

```
$ sudo apt install mysql-server-5.7
```

Change password of `root` to `password`:

Get password of user `debian-sys-maint` by command:

```
$ sudo more /etc/mysql/debian.cnf
```

Login with `debian-sys-maint` then change `root`'s password to `password`:

```
$ mysql -u debian-sys-maint -p
Enter password: ******** (password of debian-sys-maint can be get from sudo cat /etc/mysql/debian.cnf)
mysql> UPDATE mysql.user SET authentication_string=PASSWORD('password'), plugin='mysql_native_password' where user='root';
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1
mysql> exit
```

Edit MySQL configuration file `/etc/mysql/mysql.conf.d/mysqld.cnf` to set maximum connections to `2000`:

Find `#max_connections = 100`, remove comments and set to `2000`.

Restart MySQL and check:

```
$ sudo service mysql restart
$ mysql -u root -p
Enter password: ********
mysql> show variables like '%max_connection%';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| max_connections    | 2001  |
+--------------------+-------+
```

#### Install Redis 5.x/6.x

Install Redis by `apt` source:

```
$ sudo add-apt-repository ppa:chris-lea/redis-server
$ sudo apt install redis-server
$ redis-cli --version
redis-cli 6.0.x
```

#### Install Kafka

Download and unzip Kafka：

```
$ cd ~
$ wget https://downloads.apache.org/kafka/2.5.0/kafka_2.12-2.5.0.tgz
$ tar zxf kafka_2.12-2.5.0.tgz
$ ln -s kafka_2.12-2.5.0 kafka
```

Create two scripts `nohup-start-zookeeper.sh` and `nohup-start-kafka` under `~` in order to start Kafka quickly:

File `nohup-start-zookeeper.sh`:

```
#!/bin/bash
cd "$(dirname "$0")"
cd kafka
echo "bin/zookeeper-server-start.sh config/zookeeper.properties"
nohup bin/zookeeper-server-start.sh config/zookeeper.properties > nohup-zookeeper.log 2>&1 &
```

File `nohup-start-kafka`:

```
#!/bin/bash
cd "$(dirname "$0")"
cd kafka
echo "bin/kafka-server-start.sh config/server.properties"
nohup bin/kafka-server-start.sh config/server.properties > nohup-kafka.log 2>&1 &
```

Make them executable:

```
$ chmod a+x nohup-start-*.sh
```

## Deploy

Download all from release to local dir.

Create `deploy` dir on server:

```
$ mkdir ~/deploy
```

Upload all files in release to remote server under dir `/home/ubuntu/deploy`:

```
# on your local machine:
$ scp * ubuntu@<ip-address>:/home/ubuntu/deploy
```

Unzip all `.tar.gz` files:

```
$ pwd
/home/ubuntu/deploy
$ sh unzip-all.sh
```

### Config Nginx

Link Nginx config file:

```
$ cd /etc/nginx/sites-enabled
$ sudo rm default
$ sudo ln -s /home/ubuntu/deploy/nginx-proxy/config/default.conf default
$ sudo service nginx reload
```

### Init Database

Unzip `sql.zip` and execute:

```
$ pwd
/home/ubuntu/deploy
$ sh clean-db.sh
```

Login by `root` and check databases:

```
$ mysql -u root -p
Enter password: ********
Welcome to the MySQL monitor...

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| crypto_contracts   |
| crypto_hd          |
| crypto_manage      |
| crypto_options     |
| crypto_quotation   |
| crypto_shared      |
| crypto_spots       |
| crypto_ui          |
| crypto_wallet      |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
13 rows in set (0.00 sec)
```

### Create Snapshot Dir

The trading engine stores its status to local file. Create dirs:

```
$ sudo mkdir /data
$ sudo mkdir /data/spots-snapshot
$ sudo mkdir /data/contracts-snapshot
$ sudo chown ubuntu:ubuntu /data/*
```

### Start Exchange

Run under `~/deploy` dir:

```
$ su start-all.sh
```

Use `ps` to check processes:

```
$ ps aux | grep crypto
ubuntu    7342  1.3  1.9 8120384 1243848 ?     Sl   Jun28  58:35 java -server -Xmx2048M -jar /home/ubuntu/deploy/contracts-api/crypto-contracts-api.jar
ubuntu    7356  0.9  1.6 8043468 1063132 ?     Sl   Jun28  40:44 java -server -Xmx2048M -jar /home/ubuntu/deploy/contracts-sequence/crypto-contracts-sequence.jar
ubuntu    7380  1.2  1.6 8053740 1084112 ?     Sl   Jun28  56:11 java -server -Xmx2048M -jar /home/ubuntu/deploy/contracts-store/crypto-contracts-store.jar
ubuntu    7427  0.7  1.7 8048616 1122956 ?     Sl   Jun28  32:50 java -server -Xmx2048M -jar /home/ubuntu/deploy/manage/crypto-manage.jar
ubuntu    7457  0.8  1.5 8040380 1015400 ?     Sl   Jun28  36:52 java -server -Xmx2048M -jar /home/ubuntu/deploy/options-sequence/crypto-options-sequence.jar
ubuntu    7478  0.6  1.6 7131876 1055480 ?     Sl   Jun28  28:58 java -server -Xmx2048M -jar /home/ubuntu/deploy/push/crypto-push.jar
ubuntu    7508  1.5  1.6 8068964 1079488 ?     Sl   Jun28  68:11 java -server -Xmx2048M -jar /home/ubuntu/deploy/quotation/crypto-quotation.jar
ubuntu    7545  1.1  1.9 8106068 1274768 ?     Sl   Jun28  48:46 java -server -Xmx2048M -jar /home/ubuntu/deploy/shared-api/crypto-shared-api.jar
ubuntu    7574  1.2  1.8 8096640 1229996 ?     Sl   Jun28  56:41 java -server -Xmx2048M -jar /home/ubuntu/deploy/spots-api/crypto-spots-api.jar
ubuntu    7603  0.9  1.6 8045908 1094648 ?     Sl   Jun28  41:05 java -server -Xmx2048M -jar /home/ubuntu/deploy/spots-sequence/crypto-spots-sequence.jar
ubuntu    7636  1.2  1.6 8051548 1076872 ?     Sl   Jun28  53:41 java -server -Xmx2048M -jar /home/ubuntu/deploy/spots-store/crypto-spots-store.jar
ubuntu    7684  0.7  1.8 8045500 1188432 ?     Sl   Jun28  31:52 java -server -Xmx2048M -jar /home/ubuntu/deploy/ui/crypto-ui.jar
ubuntu    7710  0.7  1.6 8076228 1082272 ?     Sl   Jun28  33:28 java -server -Xmx2048M -jar /home/ubuntu/deploy/wallet-api/crypto-wallet-api.jar
ubuntu    9376  1.1  1.7 8068348 1152704 ?     Sl   Jun28  48:26 java -server -Xmx2048M -jar /home/ubuntu/deploy/spots-trading/crypto-spots-trading.jar
ubuntu    9436  1.1  1.7 8058896 1124720 ?     Sl   Jun28  51:12 java -server -Xmx2048M -jar /home/ubuntu/deploy/contracts-trading/crypto-contracts-trading.jar
ubuntu   16719  0.0  0.0  14852  1088 pts/0    S+   12:59   0:00 grep --color=auto crypto
```

Logs are stores at `~/deploy/<app>/logs`.

### Mapping Domains

Add domains mapping to local hosts file:

```
<ubuntu-ip-address> www.demo.interplanetbit.com wss.demo.interplanetbit.com api.demo.interplanetbit.com static.demo.interplanetbit.com  manage.demo.interplanetbit.com
```

# Access UI

Traders can sign in from `www.demo.interplanetbit.com`. Initial users are created by SQL script as follow:

```
email            password
-------------------------
bot0@example.com password
bot1@example.com password
bot2@example.com password
bot3@example.com password
bot4@example.com password
bot5@example.com password
bot6@example.com password
bot7@example.com password
bot8@example.com password
bot9@example.com password
```

System administrator can sign in from `manage.demo.interplanetbit.com`.
