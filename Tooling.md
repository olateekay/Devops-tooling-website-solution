# Devops Tooling Website Solution #
As a member of a DevOps team, you will implement a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

In this project you will implement a solution that consists of following components:

**Infrastructure:** AWS

**Webserver Linux:** Red Hat Enterprise Linux 8

**Database Server:** Ubuntu 20.04 + MySQL

**Storage Server:** Red Hat Enterprise Linux 8 + NFS Server

**Programming Language:** PHP

**Code Repository:** GitHub


### STEP 1 : CONFIGURE NFS SERVER ###

On your AWS console, Spin up a 5 new EC2 instances;  
3 of which would be webservers with RHEL Linux 8 Operating System,
1 of which would be an NFS server also with RHEL Linux 8 Operating System,
the last one would be a database server with Ubuntu 20.0.4 Operating system

	
![alt text](image1.jpg)

Create a logical volume with size 15gb,the same AZ as your server then attach this logical volume to your NFS server.

![alt text](image2.jpg)

Connect to your NFS server and run 
`lsblk` to view the disc  attached

![alt text](image3.jpg)

Create a partition  on the disc, create a physical  volume, then create a volume group then create a  logical volume

![alt text](image4.jpg)

![alt text](image5.jpg)

Instead of formating the disks as `ext4` you will have to format them as `xfs`
Ensure there are 3 Logical Volumes. `lv-opt` `lv-apps`, and `lv-logs`.

![alt text](image7.jpg)

![alt text](image6.jpg)


Create mount points on `/mnt `directory for the logical volumes as follow: 

Mount `lv-apps` on `/mnt/apps` - To be used by webservers 

Mount `lv-logs` on /`mnt/logs` - To be used by webserver logs 

Mount `lv-opt` on `/mnt/opt`


![alt text](image8.jpg)

![alt text](image9.jpg)


Install NFS server, configure it to start on reboot and make sure it is u and running

```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service

```

![alt text](image10.jpg)

Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

```

Restart the NFS Server
```
sudo systemctl restart nfs-server.service
```

![alt text](image11.jpg)


Configure access to NFS for clients within the same subnet .To check your subnet cidr - open your EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link:

```
sudo vi /etc/exports

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

sudo exportfs -arv
```
![alt text](image12.jpg)



Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)

```
rpcinfo -p | grep nfs

```

![alt text](image13.jpg)

NB:in my case,i opened traffic to all


### Step2: CONFIGURE THE DATABASE SERVER ###

Install `MySQL` server on your db instance

```
Sudo install mysql-server
Sudo systemctl start mysql
Sudo systemctl enable mysql
Sudo systemctl status mysql
```

Create a database and name it `tooling`

Create a database user and name it `webaccess`

Grant permission to `webaccess` user on `tooling` database to do anything only from the webservers subnet cidr

![alt text](image14.jpg)

![alt text](image15.jpg)

### Step3: CONFIGURE THE WEB SERVERS ###
Lauch one of your web EC2 instance with RHEL 8 Operating System and Install NFS client

```
sudo yum install nfs-utils nfs4-acl-tools -y
```

Mount /`var/www/ `and target the NFS server’s export for apps

```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

```
![alt text](image16.jpg)

![alt text](image17.jpg)


Verify that NFS was mounted successfully by running
 `df -h`

  Make sure that the changes will persist on Web Server after reboot:

`sudo vi /etc/fstab`

`<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`

Install Apache
```
$ sudo yum install httpd -y

$ sudo systemctl start httpd
$ sudo systemctl enable httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /
usr/lib/systemd/system/httpd.service.
 ```
 Repeat steps 1-5 for the other two Web Servers.

Verify that Apache files and directories are available on the Web Server in /`var/www` and also on the NFS server in /mnt/apps. If you see the same files - it means NFS is mounted correctly.

Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs.
Before mounting, try to backup the log file. mounting would replace everything in there 

```
$ sudo mv /var/log/httpd /var/log/httpdyke

```

```
sudo mount -t -o nfs rw,nosuid 10.154.0.6:/mnt/logs
```
Fork the tooling source code from Darey.io Github Account to your Github account. 
Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to `/var/www/html`



