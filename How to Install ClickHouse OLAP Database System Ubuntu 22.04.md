# How to Install ClickHouse OLAP Database System Ubuntu 22.04



ClickHouse is an open-source columnar database management system designed for online analytical processing (OLAP). It was developed by Yandex, a Russian multinational corporation specializing in Internet-related products and services. ClickHouse is known for its high performance in handling large volumes of data and its ability to execute complex queries in real-time, making it an ideal solution for data analytics and big data applications.

This tutorial will guide you through the process of installing ClickHouse on an Ubuntu 22.04 system, configuring it, and performing basic operations.

## Step 1 – Install Required Dependencies

ClickHouse requires certain dependencies to be installed on your system. Install these dependencies using the following command:

```bash
apt install apt-transport-https ca-certificates dirmngr
```

## Step 2 – Add ClickHouse Repository and GPG Key

Next, we need to add the ClickHouse repository to our system’s package sources list and import the GPG key to ensure the packages’ authenticity.

```bash
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754
echo "deb https://packages.clickhouse.com/deb stable main" | tee /etc/apt/sources.list.d/clickhouse.list
```

After adding the ClickHouse repository, update your package list to include the new repository:

```bash
apt update
```

## Step 3 – Install ClickHouse

Now, install the ClickHouse server and client using the following command:

```bash
apt install clickhouse-server clickhouse-client
```

You will be asked to set a password for the default user.

```bash
Enter password for the default user: 
Password for the default user is saved in file /etc/clickhouse-server/users.d/default-password.xml.
```

You can also access the **/etc/clickhouse-server/users.d/default-password.xml** file to retrieve or change the password as needed.

## Step 4 – Start and Enable ClickHouse Service

To start the ClickHouse server and enable it to start on boot, use the following commands:

```bash
systemctl start clickhouse-server 
systemctl enable clickhouse-server
```

Verify that the ClickHouse server is running:

```bash
systemctl status clickhouse-server
```

You should see output similar to the following, indicating that the service is active and running:

```bash
● clickhouse-server.service - ClickHouse Server (analytic DBMS for big data)
     Loaded: loaded (/lib/systemd/system/clickhouse-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-06-26 11:27:31 UTC; 3s ago
   Main PID: 43570 (clickhouse-serv)
      Tasks: 666 (limit: 4579)
     Memory: 133.7M
        CPU: 1.065s
     CGroup: /system.slice/clickhouse-server.service
             ├─43569 clickhouse-watchdog "" "" "" "" "" "" "" --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-server.pid
             └─43570 /usr/bin/clickhouse-server --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-server.pid

Jun 26 11:27:30 ubuntu systemd[1]: Starting ClickHouse Server (analytic DBMS for big data)...
Jun 26 11:27:30 ubuntu clickhouse-server[43569]: Processing configuration file '/etc/clickhouse-server/config.xml'.
Jun 26 11:27:30 ubuntu clickhouse-server[43569]: Logging trace to /var/log/clickhouse-server/clickhouse-server.log
Jun 26 11:27:30 ubuntu clickhouse-server[43569]: Logging errors to /var/log/clickhouse-server/clickhouse-server.err.log
```

## Step 5 – Access ClickHouse Client

To interact with the ClickHouse server, use the ClickHouse client. Open the client and log in using the default user:

```bash
clickhouse-client --user default --password

Enter the password when prompted. You should see a prompt similar to this:

ClickHouse client version 24.5.3.5 (official build).
Password for user (default): 
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 24.5.3.

Warnings:
 * Delay accounting is not enabled, OSIOWaitMicroseconds will not be gathered. You can enable it using `echo 1 > /proc/sys/kernel/task_delayacct` or by using sysctl.
 * Maximum number of threads is lower than 30000. There could be problems with handling a lot of simultaneous queries.

ubuntu :) 
```

## Step 6 – Basic Operations in ClickHouse

Now that ClickHouse is installed and running, let’s perform some basic operations to familiarize ourselves with the database system.

Create a new database named db1:

```bash
CREATE DATABASE db1;
USE db1;
```

Create a table named users with some sample columns:

```bash
CREATE TABLE users (id UInt64, name String, jobs String, last_login DateTime) ENGINE=MergeTree() PRIMARY KEY id ORDER BY id;
```

Insert some sample data into the user’s table:

```bash
INSERT INTO users VALUES (1, 'jay', 'admin', '2024-06-10 00:10:10');
INSERT INTO users VALUES (2, 'hit', 'Manager', '2024-06-05 01:19:10');
```

Query the data from the user’s table:

```bash
SELECT * FROM users;
```

The output should be:

```bash
Query id: 9ff7c880-b63b-4762-b90e-f32f0020564f

   ┌─id─┬─name─┬─jobs────┬──────────last_login─┐
1. │  2 │ hit  │ Manager │ 2024-06-05 01:19:10 │
   └────┴──────┴─────────┴─────────────────────┘
   ┌─id─┬─name─┬─jobs──┬──────────last_login─┐
2. │  1 │ jay  │ admin │ 2024-06-10 00:10:10 │
   └────┴──────┴───────┴─────────────────────┘
```

Update a record in the user table:

```bash
ALTER TABLE users UPDATE jobs = 'Administrator' WHERE name = 'hit';
```

Verify the update by querying the data again:

```bash
SELECT * FROM users;
```

The output should show the updated data:

```bash
   ┌─id─┬─name─┬─jobs──────────┬──────────last_login─┐
1. │  2 │ hit  │ Administrator │ 2024-06-05 01:19:10 │
   └────┴──────┴───────────────┴─────────────────────┘
   ┌─id─┬─name─┬─jobs──┬──────────last_login─┐
2. │  1 │ jay  │ admin │ 2024-06-10 00:10:10 │
   └────┴──────┴───────┴─────────────────────┘
```

Finally, drop the table and database:

```bash
DROP TABLE users;
DROP DATABASE db1;
```

List all databases to ensure db1 has been dropped:

```bash
SHOW DATABASES;
```

You should see the following databases:

```bash
   ┌─name───────────────┐
1. │ INFORMATION_SCHEMA │
2. │ default            │
3. │ information_schema │
4. │ system             │
   └────────────────────┘
```

## Conclusion

In this tutorial, we covered the installation of ClickHouse on an Ubuntu 22.04 system, starting and enabling the ClickHouse service, and performing basic database operations. ClickHouse is a robust tool for OLAP applications, offering high performance for real-time analytics. By following these steps, you should have a fully functional ClickHouse setup ready for your data analytics needs. applications. With this setup, you can now start building your own MQTT-based solutions. You can use Mosquitto MQTT as a database on [**dedicated server hosting**](https://www.atlantic.net/dedicated-server-hosting/) from Atlantic.Net!