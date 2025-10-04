# 🇼 Web Solution With WordPress
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
> **Business Layer (BL)**: This is the backend program that implements business logic. Application or Webserver
> **Data Access or Management Layer (DAL)**: This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server.

### Three-Tier Setup

- A Laptop or PC to serve as a client
- An EC2 Linux Server as a web server (This is where you will install WordPress)
- An EC2 Linux server as a database (DB) server

> Use RedHat OS for this project

> **Note**: for Ubuntu server, when connecting to it via SSH/Putty or any other tool, we used ubuntu user, but for RedHat you will need to use ec2-user user. Connection string will look like ec2-user@<Public-IP>
