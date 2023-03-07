# DEVOPS TOOLING WEBSITE SOLUTION

In [project6](https://github.com/StrangeJay/DevOps_Project6) you implemented a WordPress-based solution that is ready to be filled with content and can be used as a full-fledged website/blog. Moving further we will add some more value to our solutions that your DevOps team could utilize. We want to introduce a set of DevOps tools that will help our team in day-to-day activities in managing, developing, testing, deploying and monitoring different projects.  

The tools we want our team to be able to use are well-known and widely used by multiple DevOps teams, so we will introduce a single DevOps Tooling Solution that will consist of: 

1. [Jenkins[(https://www.jenkins.io/) – free and open source automation server used to build [CI/CD](https://en.wikipedia.org/wiki/CI/CD) pipelines. 
2. [Kubernetes](https://kubernetes.io/) – an open-source container-orchestration system for automating computer application deployment, scaling, and management.
3. [Jfrog Artifactory](https://jfrog.com/artifactory/) – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.
4. [Rancher(https://rancher.com/products/rancher/) – an open-source software platform that enables organizations to run and manage [Docker](https://en.wikipedia.org/wiki/Docker_(software)) and Kubernetes in production.
5. [Grafana](https://grafana.com/) – a multi-platform open-source analytics and interactive visualization web application.
6. [Prometheus](https://prometheus.io/) – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.
7. [Kibana](https://www.elastic.co/kibana) – Kibana is a free and open user interface that lets you visualize your [Elasticsearch](https://www.elastic.co/elasticsearch/) data and navigate the [Elastic Stack](https://www.elastic.co/elastic-stack).  

> **Note** Do not feel overwhelmed by the tools and technologies listed above, we will get ourselves familiar with them in upcoming projects! 

## Side Self Study 
Read about [Network-attached storage (NAS)](https://en.wikipedia.org/wiki/Network-attached_storage), Storage Area Network (SAN) and related protocols like [NFS](https://en.wikipedia.org/wiki/Network_File_System), [(s)FTP](https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol), [SMB](https://en.wikipedia.org/wiki/Server_Message_Block), and [iSCSI](https://en.wikipedia.org/wiki/ISCSI). Explore what [Block-level storage](https://en.wikipedia.org/wiki/Block-level_storage) is and how it is used by Cloud Service providers, and know the difference from [Object storage.](https://en.wikipedia.org/wiki/Object_storage)
On the [example of AWS services](https://dzone.com/articles/confused-by-aws-storage-options-s3-ebs-amp-efs-explained) understand the difference between Block Storage, Object Storage and Network File System.  

## Setup and technologies used in Project 7
As a member of a DevOps team, you will implement a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.   
In this project you will implement a solution that consists of following components:
1. **Infrastructure:** AWS
2. **Webserver Linux:** Red Hat Enterprise Linux 8
3. **Database Server:** Ubuntu 20.04 + MySQL
4. **Storage Server:** Red Hat Enterprise Linux 8 + NFS Server
5. **Programming Language:** PHP
6. **Code Repository:** [GitHub](https://github.com/darey-io/tooling.git)  

On the diagram below you can see a common pattern where several [stateless Web Servers](https://www.geeksforgeeks.org/what-is-a-stateless-server/) share a common database and also access the same files using [Network File Sytem (NFS)](https://en.wikipedia.org/wiki/Network_File_System) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers it look like a local file system from where they can serve the same files.  

![3-tier](https://user-images.githubusercontent.com/105195327/217787752-e970d3f2-a2d0-4722-8a14-fd028a6bafdf.png)   


It is important to know what storage solution is suitable for what use cases, for this, you need to answer the following questions:  

- **What data will be stored?**  
- **In what format?**  
- **How would this data be accessed?**  
- **By whom?**  
- **From where?**  
- **How frequently?**    
etc. Based on this you will be able to choose the right storage system for your solution.  


---
# STEP 1 – PREPARE NFS SERVER  
- Spin up a new EC2 instance with RHEL Linux 8 operating System. 
Based on your LVM experience from [Project 6](https://github.com/StrangeJay/DevOps_Project6), configure LVM on the server. 
- Create 3 EBS volumes and attach them to your instance 
- Check what block services are attached to the server using **lsblk** 
![lsblk](https://user-images.githubusercontent.com/105195327/223113737-d9ad749d-4a79-4780-abdd-597f54e7088d.png)  

- Confirm that the created blocks recide in the **/dev/** directory. 
![confirm blocks](https://user-images.githubusercontent.com/105195327/223114493-87636755-bebc-476e-a068-0fe993b1dd00.png)  

- Use the `df -h` command to see all mounts and free space on your server.  
![check mounts](https://user-images.githubusercontent.com/105195327/223115196-0d83251d-d620-4f44-b3cb-83c71760a2e4.png)  

- Create your partitions, install lvm2, mark each disk as physical volumes to be used by LVM, add all 3PVs to a volume group.  

- Create 3 logical volumes  
![3 LVS](https://user-images.githubusercontent.com/105195327/223135520-dc57aec2-6cfe-4b9f-8348-2b90b631a22e.png)  

- Instead of formatting the disks as ext4, you will have to format them as xfs. 
```
sudo mkfs -t xfs /dev/webdata-vg/apps-lv
sudo mkfs -t xfs /dev/webdata-vg/logs-lv
sudo mkfs -t xfs /dev/webdata-vg/opt-lv
``` 
![Lvs](https://user-images.githubusercontent.com/105195327/223141000-d8bc4e0e-9954-460f-bc6a-68819b012d05.png)  

- Create a directory /mnt/ with sub directories **apps**, **logs**, **opt** with the following command `sudo mkdir -vp /mnt/{apps,logs,opt}`  
![mnt dir](https://user-images.githubusercontent.com/105195327/223149215-d4426507-74c9-4a4c-9e34-f0ee7ff327c2.png)  

- Go to the /mnt directory

- Create mount points on /mnt directory for the logical volumes as follow:  
**Mount lv-apps on /mnt/apps** – To be used by webservers    
**Mount lv-logs on /mnt/logs** – To be used by webserver logs   
**Mount lv-opt on /mnt/opt** – To be used by Jenkins server in the next project  

```
sudo mount /dev/webdata-vg/apps-lv /mnt/apps 
sudo mount /dev/webdata-vg/apps-lv /mnt/logs 
sudo mount /dev/webdata-vg/apps-lv /mnt/opt 
```

![mount dir](https://user-images.githubusercontent.com/105195327/223395945-c7e07496-fa4f-4f71-8cf3-deceff392c40.png)  

- Install NFS server, configure it to start on reboot and make sure it is up and running.  
```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
![NFS working](https://user-images.githubusercontent.com/105195327/223396965-35c9340c-ff99-4c27-b0cb-6637614304e9.png)  

- Export the mounts for webservers’ **subnet CIDR** to connect as clients. For simplicity, you will install all three Web Servers inside the same subnet, but in the production set-up, you would probably want to separate each tier inside its own subnet for a higher level of security.
To check your **subnet cidr** – open your EC2 details in the AWS web console and locate the ‘Networking’ tab and open a Subnet link:  

![subnet id](https://user-images.githubusercontent.com/105195327/223399294-0c6eca93-7a33-4472-aca4-4ddc0a49e558.png)  

![subnet ip4](https://user-images.githubusercontent.com/105195327/223399327-f0e623d2-4456-42a7-9ec4-01315cd793cb.png)  

Make sure we set up permission that will allow our web servers to read, write and execute files on NFS: 

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt
 
sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt
 
sudo systemctl restart nfs-server.service
```








---
# STEP 2 — CONFIGURE THE DATABASE SERVER 






---
# STEP 3 — PREPARE THE WEB SERVERS 






---




