# 🆆 Web Solution With WordPress
## 🌐 WordPress Web Solutions with DevOps & Cloud Integration


> **`WordPress` is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).**

> **In this project you will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using `WordPress`.**

---

### **This project consists of two parts.**
- Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.

- Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.

### Three-Tier Architecture
- Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.

- Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.

> **Presentation Layer (PL)**: This is the user interface such as the client server or browser on your laptop.

> **Business Layer (BL)**: This is the backend program that implements business logic. Application or Webserver.

> **Data Access or Management Layer (DAL)**: This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server.

### Three-Tier Setup

- A Laptop or PC to serve as a client
- An EC2 Linux Server as a web server (This is where you will install WordPress)
- An EC2 Linux server as a database (DB) server

> Use RedHat OS for this project

> **Note**: for Ubuntu server, when connecting to it via SSH/Putty or any other tool, we used ubuntu user, but for RedHat you will need to use ec2-user user. Connection string will look like ec2-user@<Public-IP>

---

## 🎯**Step 1: Setting Up the Web Server for WordPress Deployment**

- Launch an EC2 instance that will serve as `Web Server`. Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

<img width="1309" height="595" alt="Screenshot From 2025-10-07 23-00-34" src="https://github.com/user-attachments/assets/52b88dce-7d08-44de-af24-4e2f71cb410a" />

<img width="1307" height="179" alt="Screenshot From 2025-10-07 23-02-37" src="https://github.com/user-attachments/assets/872be3c2-2d58-4fbd-b9a2-67cd0252eb75" />


- Attach all three volumes one by one to your Web Server EC2 instance.

<img width="1097" height="306" alt="Screenshot From 2025-10-07 23-20-16" src="https://github.com/user-attachments/assets/ad0b2457-a15c-42b3-bae8-179eb6387d42" />

<img width="1321" height="541" alt="Screenshot From 2025-10-07 23-21-33" src="https://github.com/user-attachments/assets/5f084748-9550-4418-936c-57381c5af029" />

- Open up the Linux terminal to begin configuration

- Use the `lsblk` command to list all block devices currently attached to the server. Identify the names of the newly added devices. In Linux, all block devices are represented as files under the `/dev/` directory. Verify the presence of the three newly created devices—typically named `xvdf`, `xvdg`, and `xvdh`—by running `ls /dev/` and confirming they appear in the listing.

<img width="1319" height="729" alt="Screenshot From 2025-10-07 23-48-28" src="https://github.com/user-attachments/assets/efe0cffb-0a1d-45b6-9058-feae9e594fcd" />

- Use `df -h` command to see all mounts and free space on your server.
- Use `fdisk` utility to create a single partition on each of the 3 disks.

```
sudo fdisk /dev/nvme1n1
sudo fdisk /dev/nvme2n1
sudo fdisk /dev/nvme3n1
```

<img width="1295" height="552" alt="Screenshot From 2025-10-07 23-54-59" src="https://github.com/user-attachments/assets/3cd2ee72-7af0-4bdb-b5cf-0558f2debed0" />

- Use the `lsblk` utility to verify the newly created partitions on each of the three attached disks.

<img width="1319" height="309" alt="Screenshot From 2025-10-08 00-09-41" src="https://github.com/user-attachments/assets/c47451f1-f30f-45b7-9372-8d998a6f1832" />

- Install `lvm2` package using
```
sudo yum install lvm2
```
- Run `sudo lvmdiskscan` command to check for available partitions.

<img width="1315" height="275" alt="image" src="https://github.com/user-attachments/assets/75ad0337-6113-42f6-875b-5d1cd31894e2" />


> Note: Previously, in Ubuntu we used apt command to install packages, in RedHat/CentOS a different package manager is used, so we shall use yum command instead.

- Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.

```
sudo pvcreate /dev/nvme1n1p1
sudo pvcreate /dev/nvme2n1p1
sudo pvcreate /dev/nvme3n1p1
```

- Verify that your Physical volume has been created successfully by running:
```
sudo pvs
```

<img width="1319" height="306" alt="image" src="https://github.com/user-attachments/assets/c5d944ab-097b-41a4-8ec6-d5a6aa9e8ec2" />

- Use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

```
sudo vgcreate webdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1
```

- Verify that your VG has been created successfully by running :
```
sudo vgs
```

<img width="1308" height="162" alt="image" src="https://github.com/user-attachments/assets/d1290c27-7c2d-4f46-a4eb-3e3867b45638" />

- Use `lvcreate` utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. 

> NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```

- Verify that your Logical Volume has been created successfully by running:
```
sudo lvs
```

<img width="1303" height="209" alt="image" src="https://github.com/user-attachments/assets/622d2360-6b8b-4aed-8e98-369aa351c3a0" />

- Verify the entire setup
```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk
```

<img width="1304" height="479" alt="image" src="https://github.com/user-attachments/assets/94b21f2d-baad-4871-bca8-2cb6e8c1b447" />

<img width="1307" height="405" alt="image" src="https://github.com/user-attachments/assets/4c94de3b-c435-44ae-ae5d-384136822b9a" />


> Use `mkfs.ext4` to format the logical volumes with ext4 filesystem.

```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

<img width="1301" height="557" alt="image" src="https://github.com/user-attachments/assets/c8e29024-a6a9-4fa2-aac5-0bc1952b314c" />

- Create /var/www/html directory to store website files
```
sudo mkdir -p /var/www/html
```

- Create /home/recovery/logs to store backup of log data 
```
sudo mkdir -p /home/recovery/logs
```

- Mount /var/www/html on apps-lv logical volume
```
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```

- Use `rsync` utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
```
sudo rsync -av /var/log/ /home/recovery/logs/
```

- Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important)
```
sudo mount /dev/webdata-vg/logs-lv /var/log
```

- Restore log files back into /var/log directory
```
sudo rsync -av /home/recovery/logs/ /var/log
```

- Update `/etc/fstab` file so that the mount configuration will persist after restart of the server.
> The UUID of the device will be used to update the /etc/fstab file;
```
sudo blkid
```

<img width="1323" height="299" alt="image" src="https://github.com/user-attachments/assets/7babc5bf-0ef0-49e9-8122-4905e2410d1c" />

```
sudo vi /etc/fstab
```

<img width="1128" height="157" alt="image" src="https://github.com/user-attachments/assets/62689cd2-51a6-494e-abe5-8ec90b899b75" />

> Update `/etc/fstab` in this format using your own UUID and rememeber to remove the leading and ending quotes.

- Test the configuration and reload the daemon
```
sudo mount -a
sudo systemctl daemon-reload
```

- Verify your setup by running `df -h`, output must look like this:

<img width="1214" height="369" alt="image" src="https://github.com/user-attachments/assets/39ca032e-12f9-4326-a3f7-5e9b2a1fe3dd" />

---

## 🎯**Step 2: Database Server Preparation**

- Deploy a second RedHat EC2 instance designated as the `DB Server`. Repeat the steps used for the Web Server setup, but create a volume named `db-lv` instead of `apps-lv`, and mount it to the `/db` directory rather than `/var/www/html/`.

<img width="1107" height="200" alt="image" src="https://github.com/user-attachments/assets/022af968-53e2-448d-93bb-fac83521fe41" />

<img width="1105" height="263" alt="image" src="https://github.com/user-attachments/assets/2281cad8-36e0-4860-a9ed-d7460b7f3f38" />

```
sudo fdisk /dev/nvme1n1
sudo fdisk /dev/nvme2n1
sudo fdisk /dev/nvme3n1
```

<img width="1242" height="549" alt="image" src="https://github.com/user-attachments/assets/45b9bf88-47da-4a0a-96f5-071bd3528aca" />

- Verify the new partitions:

```
lsblk
```

<img width="1227" height="306" alt="image" src="https://github.com/user-attachments/assets/fdad76e0-b109-466b-bbcd-ec5eee4e92a7" />


- Install `lvm2` package using

```
sudo yum install lvm2
```
- Run `sudo lvmdiskscan` command to check for available partitions.

<img width="1227" height="306" alt="image" src="https://github.com/user-attachments/assets/189cbb0b-e6d2-42fb-b00a-43c8adb98e47" />


- Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.

```
sudo pvcreate /dev/nvme1n1p1
sudo pvcreate /dev/nvme2n1p1
sudo pvcreate /dev/nvme3n1p1
```

- Verify that your Physical volume has been created successfully by running:
```
sudo pvs
```

- Use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

```
sudo vgcreate database-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1
```

- Verify that your VG has been created successfully by running :
```
sudo vgs
```
<img width="1244" height="420" alt="image" src="https://github.com/user-attachments/assets/cba9189c-7b8b-450e-a935-df4b07bf8be4" />


- Use lvcreate to create a single LV for the database:

```
sudo lvcreate -n db-lv -L 14G database-vg
```

- Verify that your Logical Volume has been created successfully by running:
```
sudo lvs
```
<img width="1249" height="135" alt="image" src="https://github.com/user-attachments/assets/90bbfccd-80c8-4daa-aaf1-a248d40d162f" />


- Verify the entire setup
```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk
```

<img width="1268" height="351" alt="image" src="https://github.com/user-attachments/assets/ba9444ec-a935-443e-b608-d77f4e970bc5" />

> Use `mkfs.ext4` to format the logical volumes with ext4 filesystem.

- Create an ext4 filesystem on the LV
```
sudo mkfs.ext4 /dev/database-vg/db-lv
```

- Create the mount point directory
```
sudo mkdir -p /db
```

- Mount the filesystem
```
sudo mount /dev/database-vg/db-lv /db
```

<img width="1268" height="351" alt="image" src="https://github.com/user-attachments/assets/0492ce0c-dd5b-4a7b-a3b8-4bbf5410c6d6" />


- Update `/etc/fstab` file so that the mount configuration will persist after restart of the server.
> The UUID of the device will be used to update the /etc/fstab file;
```
sudo blkid
```

<img width="1263" height="246" alt="image" src="https://github.com/user-attachments/assets/9c073541-1beb-4bf7-a63a-33d93c9e828d" />


```
sudo vi /etc/fstab
```

<img width="1270" height="143" alt="image" src="https://github.com/user-attachments/assets/53c0a00d-8c15-49f8-9a4b-4b5ef98df0d0" />

- Test the configuration and reload the daemon
```
sudo mount -a
sudo systemctl daemon-reload
```

- Verify your setup by running `df -h`, output must look like this:

<img width="1270" height="363" alt="image" src="https://github.com/user-attachments/assets/fef50ec9-fdf2-4041-a7b0-30225f817fce" />

---

## 🎯**Step 3: Configure and Install WordPress on Your Web Server EC2 Instance**

- Update the repository
```
sudo yum -y update
```

- Install wget, Apache and it's dependencies
```
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```

<img width="1317" height="636" alt="image" src="https://github.com/user-attachments/assets/01dc2500-b6ad-422e-b9de-85d15d5456d2" />


- Install PHP and it's dependencies
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

```
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

```
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
```

<img width="1317" height="636" alt="image" src="https://github.com/user-attachments/assets/c001a443-7e05-49c0-938e-e07af13eb161" />

- Verify the PHP version using `php -v`

<img width="999" height="160" alt="image" src="https://github.com/user-attachments/assets/74f65d11-5261-48a6-9e0b-01dbf36102f3" />


- Start and enable PHP-FPM:

```
sudo systemctl start php-fpm
```
```
sudo systemctl enable php-fpm
```
```
setsebool -P httpd_execmem 1
```

<img width="1297" height="293" alt="image" src="https://github.com/user-attachments/assets/42d39899-2643-4a14-9b0d-e7f7c16d2226" />


- Restart Apache

```
sudo systemctl restart httpd
```

- Download wordpress and copy wordpress to `/var/www/html`
```
mkdir wordpress
```
```
cd wordpress
```
```
sudo wget http://wordpress.org/latest.tar.gz
```
```
sudo tar -xzvf latest.tar.gz
```
```
sudo rm -rf latest.tar.gz
```
```
cp wordpress/wp-config-sample.php wordpress/wp-config.php
```
```
cp -R wordpress /var/www/html/
```

- Configure SELinux Policies
```
sudo chown -R apache:apache /var/www/html/wordpress
```
```
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
```
```
sudo setsebool -P httpd_can_network_connect=1
```

---

## 🎯**Step 4: MySQL Installation on the EC2 Database Server**

```
sudo yum update
sudo yum install mysql-server
```

- Check that the MySQL service is active by running
```
sudo systemctl status mysqld
```

- If the service is not running, restart it using
```
sudo systemctl restart mysqld
```

- Enable it to start automatically on boot with
```
sudo systemctl enable mysqld
```

---

## 🎯**Step 5 — Database Configuration for WordPress Deployment**

```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

---

## 🎯**Step 6: Configure WordPress to Connect to the Remote Database**

> `Note`: Open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server's IP address, so in the Inbound Rule configuration specify source as /32


- Install `MySQL` client and test that you can connect from your `Web Server` to your `DB server` by using `mysql-client`

```
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```

- Verify if you can successfully execute `SHOW DATABASES`; command and see a list of existing databases.

- Change permissions and configuration so `Apache` could use `WordPress`:

- Enable TCP port 80 in Inbound Rules configuration for your `Web Server EC2` (enable from everywhere 0.0.0.0/0 or from your workstation's IP)

- Try to access from your browser the link to your WordPress `http://<Web-Server-Public-IP-Address>/wordpress/`

- Fill out your DB credentials:

- If you see this message - it means your WordPress has successfully connected to your remote MySQL database.

---

## **CONGRATULATIONS!**
