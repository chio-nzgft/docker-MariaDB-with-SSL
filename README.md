# docker-MariaDB-with-SSL

#Create certs
```
mkdir /etc/newcerts
cd /etc/newcerts
# CA key
openssl genrsa 2048 > ca-key.pem
openssl req -new -x509 -nodes -days 3600 -key ca-key.pem -out ca-cert.pem
# server key
openssl req -newkey rsa:2048 -days 3600 -nodes -keyout server-key.pem -out server-req.pem
openssl rsa -in server-key.pem -out server-key.pem
openssl x509 -req -in server-req.pem -days 3600 -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem
# client key
openssl req -newkey rsa:2048 -days 3600 -nodes -keyout client-key.pem -out client-req.pem
openssl rsa -in client-key.pem -out client-key.pem
openssl x509 -req -in client-req.pem -days 3600 -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem
# check key ok
openssl verify -CAfile ca-cert.pem server-cert.pem client-cert.pem
# ls key
ls /etc/newcerts
ca-cert.pem  ca-key.pem  client-cert.pem  client-key.pem  client-req.pem  server-cert.pem  server-key.pem  server-req.pem

```

#Run mysql docker
```
docker run -it --name mariadb -p 3306:3306 -v /var/lib/mysql:/var/lib/mysql -v /etc/newcerts:/etc/newcerts -e MYSQL_DATABASE=DB -e MYSQL_USER=user -e MYSQL_PASSWORD=userpass -e MYSQL_ROOT_PASSWORD=admin echochio/alpine-mariadb
```

#use env Run mysql docker
```
SERVER_KEY=$(cat -E /etc/newcerts/server-key.pem | xargs)
SERVER_CERT=$(cat -E /etc/newcerts/server-cert.pem | xargs)
CA_CERT=$(cat -E /etc/newcerts/ca-cert.pem | xargs)
docker run -it --name mariadb -p 3306:3306 -v /var/lib/mysql:/var/lib/mysql -e MYSQL_DATABASE=DB -e MYSQL_USER=user -e MYSQL_PASSWORD=userpass -e MYSQL_ROOT_PASSWORD=admin -e SERVER_KEY=$SERVER_KEY -e SERVER_CERT=$SERVER_CERT -e CA_CERT=$CA_CERT echochio/alpine-mariadb
```

#Create user for SSL
```
#  mysql --host=127.0.0.1 -u root -padmin
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 17
Server version: 10.1.22-MariaDB MariaDB Server

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> grant all on *.* to 'cross'@'192.168.0.17' identified by '123456'  require ssl;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> Bye
```

#Check SSL user
```
#  mysql --host=127.0.0.1 -u cross -p123456 --ssl-ca=/etc/newcerts/ca-cert.pem --ssl-cert=/etc/newcerts/client-cert.pem --ssl-key=/etc/newcerts/client-key.pem -e 'status'
--------------
mysql  Ver 15.1 Distrib 5.5.52-MariaDB, for Linux (x86_64) using readline 5.1

Connection id:          18
Current database:
Current user:           cross@172.17.0.1
SSL:                    Cipher in use is DHE-RSA-AES256-GCM-SHA384
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server:                 MariaDB
Server version:         10.1.22-MariaDB MariaDB Server
Protocol version:       10
Connection:             127.0.0.1 via TCP/IP
Server characterset:    utf8
Db     characterset:    utf8
Client characterset:    utf8
Conn.  characterset:    utf8
TCP port:               3306
Uptime:                 20 min 8 sec

Threads: 1  Questions: 34  Slow queries: 0  Opens: 18  Flush tables: 1  Open tables: 11  Queries per second avg: 0.028
--------------
```
