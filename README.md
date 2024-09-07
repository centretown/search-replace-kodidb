## Correcting MariaDB Kodi database to a different network.
I currently have about 30 devices on my rooming house network. Previously my problem was solved by adding the server ip address and name to each client's ***hosts*** file. Back then there were fewer clients and no Android boxes. I didn't have access the ***hosts*** on these boxes.  So I regenerated the database with hard coded ip addresses. 

#### Take a dump and see what's there.
```bash
sudo mariadb-dump --all-databases >kodi.mariadb.dump
grep 'ftp://' kodi.mariadb.dump | more
INSERT INTO `path` VALUES (1,'ftp://192.168.0.161:21/Public/
```
#### Log in to Maria.
```bash
mariadb -u kodi -p
```
```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 42
Server version: 10.6.18-MariaDB-0ubuntu0.22.04.1 Ubuntu 22.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
#### See what ya got.
```sql
show databases;
```
```
+--------------------+
| Database           |
+--------------------+
| MyMusic83          |
| MyVideos131        |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
6 rows in set (0.001 sec)
```
#### Nervously drop the old.
```sql
MariaDB [(none)]> DROP DATABASE MyMusic83;
Query OK, 24 rows affected (2.302 sec)

MariaDB [(none)]> DROP DATABASE MyVideos131;
Query OK, 40 rows affected (2.658 sec)

MariaDB [(none)]> show databases;
```
#### Inspect the damage.
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.001 sec)
```
#### Make a copy of your dump.
```bash
cp kodi.mariadb.dump kodi.mariadb.dump.backup
```
#### Replace the old with the new.
```bash
sed -i 's/192\.168\.0\.161/192\.168\.10\.161/g' kodi.mariadb.dump
```
#### Review the changes.
```bash
grep 'ftp://' kodi.mariadb.dump | more
```
```
INSERT INTO `path` VALUES (1,'ftp://192.168.10.161:21/Public
```
#### Replace the old with the new.
```bash
sudo mariadb < /mnt/molly/maria/kodi.mariadb.sql
```
## Conclusion
After modifying the advancedsettings.xml in Kodi's userdata folder. Things ran beautifully. It's likely that in this version of kodi a simple update query may have achieved the same result. It hasn't always been the case and I didn't want to take the time to find out. This approach has many more use cases.

#### Final Note
On the Android devices I still had to reinstall kodi to clear the previous install's advancedsettings.xml