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

#use env set up
```
SERVER_KEY=$(cat -E /etc/newcerts/server-key.pem | xargs)
SERVER_CERT=$(cat -E /etc/newcerts/server-cert.pem | xargs)
CA_CERT=$(cat -E /etc/newcerts/ca-cert.pem | xargs)
docker run -it --name mariadb -p 3306:3306 -v /var/lib/mysql:/var/lib/mysql -e MYSQL_DATABASE=DB -e MYSQL_USER=user -e MYSQL_PASSWORD=userpass -e MYSQL_ROOT_PASSWORD=admin -e SERVER_KEY=$SERVER_KEY -e SERVER_CERT=$SERVER_CERT -e CA_CERT=$CA_CERT echochio/alpine-mariadb
```
