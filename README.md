# Install PostgreSQL Cluster

## Update your Linux packages
```
apt update -y
apt upgrade -y
```

## ติดตั้ง PostgreSQL
```
apt install postgresql
```

## ตั้งค่า Primary Node
1. แก้ไขไฟล์ `postgresql.conf`
```
nano /etc/postgresql/[POSTGRESQL_VERSION]/main/postgresql.conf
```
2. ตั้งค่าตามนี้
```
listen_addresses = '*'
wal_level = replica
max_wal_senders = 3
wal_keep_size = 1GB
```
3. ตั้งค่าไฟล์ `Replicas host` ใยไฟล์ `pg_hba.conf`
```
nano /etc/postgresql/[POSTGRESQL_VERSION]/main/pg_hba.conf
```
4. ตั้งค่าตามนี้
```
host replication replicator [SECOND_SERVER_IP]/32 trust
```

## ทำการ Restart Service ด้วยคำสั่ง
```
systemctl restart postgresql
```

## สร้าง User สำหรับ Replication
1. เปิด PostgreSQL CLI ด้วยคำสั่ง
```
sudo -u postgres psql
```
2. สร้าง User ด้วยคำสั่ง
```
CREATE USER replicator REPLICATION LOGIN CONNECTION LIMIT 3 ENCRYPTED PASSWORD 'your_password';
```
3. ใช้คำสั่ง `\q` เพื่อออกจาก PostgreSQL CLI


## ตั้งค่า Second Node
1. ก่อนอื่ต้องทำการหยุดการทำงานของ PostgrSQL ของเครื่อง Second Node ก่อนด้วยคำสั่ง
```
systemctl stop postgresql
```
2. สร้าง Directory สำหรับ Standby node
```
sudo rsync -av /var/lib/postgresql/[POSTGRESQL_VERSION]/main/ /var/lib/postgresql/[POSTGRESQL_VERSION]/main/
```
3. จากนั้นแก้ไขไฟล์ `postgresql.conf`
```
nano /etc/postgresql/[POSTGRESQL_VERSION]/main/postgresql.conf
```
4. ค้นหาการตั้งค่า และตั้งค่าตามตัวอย่างต่อไปนี้
```
hot_standby = on
primary_conninfo = 'host=[FIRST_SERVER_IP] port=5432 user=replication password=[YOUR_PASSWORD] sslmode=prefer'
primary_slot_name = 'standby_slot'
```
5. จากนั้นทำการ Start Service
```
systemctl start postgresql
```



### Credit
- <a href="https://www.servermania.com/kb/articles/setup-postgresql-cluster">How to Setup a PostgreSQL Database Cluster</a>
- <a href="https://www.cherryservers.com/blog/how-to-configure-ssl-on-postgresql">How to Configure SSL on PostgreSQL</a>

