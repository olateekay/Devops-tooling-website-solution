As a member of a DevOps team, you will implement a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

In this project you will implement a solution that consists of following components:

**Infrastructure:** AWS

**Webserver Linux:** Red Hat Enterprise Linux 8

**Database Server:** Ubuntu 20.04 + MySQL

**Storage Server:** Red Hat Enterprise Linux 8 + NFS Server

**Programming Language:** PHP

**Code Repository:** GitHub


### STEP 1 ###

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

Create mount points on `/mnt `directory for the logical volumes as follow: 

Mount `lv-apps` on `/mnt/apps` - To be used by webservers 

Mount `lv-logs` on /`mnt/logs` - To be used by webserver logs 

Mount `lv-opt` on `/mnt/opt`





