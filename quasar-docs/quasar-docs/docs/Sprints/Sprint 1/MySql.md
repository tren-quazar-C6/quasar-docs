# MySQL Installation

Remote server connection from Termius. Run this command to identify the server's OS:

```bash
cat /etc/os-release
```

The server OS is Ubuntu, so from now on the commands to install MySQL are meant for Ubuntu only.

1. Update the package list
```bash
sudo apt update
```

2. Install MySQL Server
```bash
sudo apt install -y mysql-server
```

3. Start MySQL
```bash
sudo systemctl start mysql
```

4. Enable MySQL on boot
```bash
sudo systemctl enable mysql
```

5. Run the security script
```bash
sudo mysql_secure_installation
```
The script will ask you to set a root password, remove anonymous users, and disable remote root login. Answer `Y` to all.

6. Verify MySQL version
```bash
mysql --version
```

7. Connect to MySQL
```bash
sudo mysql
```

8. Verify the installation — inside the MySQL shell run:
```sql
SHOW DATABASES;
```
If you see `information_schema`, `mysql`, `performance_schema`, and `sys`, the installation is all done!