# Setting up MySQL
[[toc]]


## Creating a database for Pterodactyl
MySQL is a core component of Pterodactyl Panel but it can be confusing to setup and use if you've never done so before.
This is a very basic tutorial that skims just enough of the surface to set MySQL up and running with the panel.
If you're interested in learning more, there are some great tutorials available on the Internet.

### Logging In
The first step in this process is to login to the MySQL command line where we will be executing some statements to get
things setup. To do so, simply run the command below and provide the Root MySQL account's password that you setup when
installing MySQL. If you do not remember doing this, chances are you can just hit enter as no password is set.

``` bash
mysql -u root -p
```

### Creating a user
For security sake, and due to changes in MySQL 5.7, you'll need to create a new user for the panel. To do so, we want
to first tell MySQL to use the mysql database, which stores such information.

Next, we will create a user called `pterodactyl` and allow logins from localhost which prevents any external connections
to our database. You can also use `%` as a wildcard or enter a numeric IP. We will also set the account password
to `somePassword`.

``` sql
USE mysql;

# Remember to change 'somePassword' below to be a unique password specific to this account.
CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED WITH mysql_native_password BY 'somePassword';
```

### Create a database
Next, we need to create a database for the panel. In this tutorial we will be naming the database `panel`, but you can
substitute that for whatever name you wish.

``` sql
CREATE DATABASE panel;
```

### Assigning permissions
Finally, we need to tell MySQL that our pterodactyl user should have access to the panel database. To do this, simply
run the command below. If you plan on also using this MySQL instance as a database host on the Panel you'll want to
include the `WITH GRANT OPTION` (which we are doing here). If you won't be using this user as part of the host setup
you can remove that.

``` sql
GRANT ALL PRIVILEGES ON panel.* TO 'pterodactyl'@'127.0.0.1' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

## Creating a Database Host for Nodes
:::tip
This section covers creating a MySQL user that has permission to create and modify users. This allows the Panel to create per-server databases on the given host.
:::

### Creating a user
If your database is on a different host than the one where your Panel or Daemon is installed make sure to use the IP address of the machine the Panel is running on. If you use `127.0.0.1` and try to connect externally, you will receive a connection refused error.

```sql
USE mysql;

# You should change the username and password below to something unique.
CREATE USER 'pterodactyluser'@'127.0.0.1' IDENTIFIED BY 'somepassword';
```

### Assigning permissions
The command below will give your newly created user the ability to create additional users, as well as create and destroy databases. As above, ensure `127.0.0.1` matches the IP address you used in the previous command.

```sql
GRANT ALL PRIVILEGES ON *.* TO 'pterodactyluser'@'127.0.0.1' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### Allowing external database access
Chances are you'll need to allow external access to this MySQL instance in order to allow servers to connect to it. To do this, open `my.cnf`, which varies in location depending on your OS and how MySQL was installed.

More recent versions of MySQL have moved the default configuration to `mysql.conf.d/mysqld.cnf` or for MariaDB installations the default configuration should be in `50-server.cnf`. *However*, `my.cnf` has been changed to update the default configurations so you don't edit your default configuration files (this is now considered bad practice)!

If you open `my.cnf`, you'll want to add the lines:
```
[mysqld]
bind-address=0.0.0.0
```
This will override the default MySQL configuration, which by default will only accept requests from lo. Updating this will allow connections on all interfaces, and thus, external connections.

If your Node and Daemon are on the same machine, and you won't be needing external access, you can also use the `docker0` interface IP address, rather than `127.0.0.1`. This IP address can be found by running `ip addr | grep docker0`, and it likely looks something like `172.x.x.x`.