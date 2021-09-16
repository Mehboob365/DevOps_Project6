**WEB SOLUTION WITH WORDPRESS**

In this project we are tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. 
WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

**Project 6 consists of two parts:**

Configure storage subsystem for Web and Database servers based on Linux OS. 
The focus of this part is to give us practical experience of working with disks, partitions and volumes in Linux.

Install WordPress and connect it to a remote MySQL database server. 
This part of the project will solidify our skills of deploying Web and DB tiers of Web solution.

**LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”**

**Step 1 — Preparation of a Web Server**

1. Launch an EC2 instance that will serve as "Web Server".

   ![image](https://user-images.githubusercontent.com/67065306/132765248-e5a9ae85-94a1-4046-a22c-68e3013e7bb4.png)

   
   Create 3 volumes in the same AZ (Availability Zone)as your Web Server EC2, each of 10 GiB.
   
   ![image](https://user-images.githubusercontent.com/67065306/132765098-067d3d69-0b9c-4f30-941f-022ce324cebc.png)

   Add EBS Volume to an EC2 instance here:
   
2. Will attach all three volumes one by one to our Web Server EC2 instance

   ![image](https://user-images.githubusercontent.com/67065306/132767029-1b348425-2865-4f82-ba6f-827be3da4780.png)

   ![image](https://user-images.githubusercontent.com/67065306/132767183-3a550279-4c14-4afe-bf35-e25d33233dfc.png)
   
3. On the Linux terminal we begin configuration;

   Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. 
   
   All devices in Linux reside in /dev/ directory.
   
   Inspect it with ls /dev/ and we make sure we see all 3 newly created block devices  – their names will likely be xvdf, xvdh, xvdg.
  
    ![image](https://user-images.githubusercontent.com/67065306/132767999-6f44d028-47b8-40ed-be77-b651b1754cc3.png)

4. Use df -h command to see all mounts and free space on the web server
    
    ![image](https://user-images.githubusercontent.com/67065306/132769092-0f3e8594-1d9d-4113-b787-d94a1fe6a393.png)

5. We will use gdisk utility to create a single partition on each of the 3 disks

   sudo gdisk /dev/xvdf  then 
   
   sudo gdisk /dev/xvdg  then 
   
     ![image](https://user-images.githubusercontent.com/67065306/132874875-ed5044f1-f55b-428b-91a0-face95841222.png)

   sudo gdisk /dev/xvdh
   
    ![image](https://user-images.githubusercontent.com/67065306/132875889-a8a5d3fc-8939-4387-bef0-ad6e6b629449.png)


 6. Will install lvm2 package using sudo yum install lvm2. 
    
     ![image](https://user-images.githubusercontent.com/67065306/132772267-6bee1afc-51d0-4c18-a97f-81dd2629b715.png)
     ![image](https://user-images.githubusercontent.com/67065306/132772329-725225df-71ec-42f3-a0e3-3b8a74967c84.png)

     Next, Run sudo lvmdiskscan command to check for available partitions.
   
     ![image](https://user-images.githubusercontent.com/67065306/132876517-a36acc32-13fa-4557-a359-25dce53ac69c.png)

   We run finally run, lsblk to see our configuration;
  
    ![image](https://user-images.githubusercontent.com/67065306/132877580-6076755a-0c2f-496c-b3c9-9fe2b4a2c792.png)
    
   
 7. We will Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.
   
    sudo pvcreate /dev/xvdf1
    
    sudo pvcreate /dev/xvdg1
    
    sudo pvcreate /dev/xvdh1
    
    ![image](https://user-images.githubusercontent.com/67065306/132878284-50b19d79-e610-42c9-99bd-194ac788538a.png)
    
 8. Verify that your Physical volume has been created successfully by running sudo pvs
    
    ![image](https://user-images.githubusercontent.com/67065306/132878424-8d849639-1472-44e2-ac07-0c25d3a78c3e.png)

 9. Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

      sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
      
    ![image](https://user-images.githubusercontent.com/67065306/132879077-b6c7a665-a53a-4a47-a60b-a9d09789f8d2.png)

 10. Verify that your VG has been created successfully by running sudo vgs
 
     ![image](https://user-images.githubusercontent.com/67065306/132879434-a3106bbf-7c31-4b22-8877-c4d5c2a6dc94.png)
     
 11.  Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. 
      NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
      
     sudo lvcreate -n apps-lv -L 14G webdata-vg
     
     sudo lvcreate -n logs-lv -L 14G webdata-vg
     
   ![image](https://user-images.githubusercontent.com/67065306/132880012-6d1f461a-c7d7-4f78-be17-4401c55892ee.png)

  12.   We verify that our Logical Volume has been created successfully by running 
        
        sudo lvs

   ![image](https://user-images.githubusercontent.com/67065306/132881065-9c4298e1-6684-417e-b8ea-22c1160edb79.png)


 13. Verify the entire setup

      sudo vgdisplay -v #view complete setup - VG, PV, and LV
      
   ![image](https://user-images.githubusercontent.com/67065306/132881471-c1952c78-cff2-41f9-9dfd-55ff2e0160cf.png)

      sudo lsblk 
    
    ![image](https://user-images.githubusercontent.com/67065306/132881594-5b46c846-2870-4f50-b01c-b6b2e8ea5e5f.png)

 
  14. We will use mkfs.ext4 to format the logical volumes with ext4 filesystem

        sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
        
   ![image](https://user-images.githubusercontent.com/67065306/132923159-cec8301f-36a9-43e3-b3d8-4688eb6ab2f6.png)
     

         sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
        
   ![image](https://user-images.githubusercontent.com/67065306/132923386-d2152209-5feb-4485-8700-b3dae7b85190.png)
 
 
 15. We will create /var/www/html directory to store website files

      sudo mkdir -p /var/www/html
   
   ![image](https://user-images.githubusercontent.com/67065306/132923686-ff2c7a83-54cd-4e90-bd0d-19635c235f32.png)

16. We will create /home/recovery/logs to store backup of log data

       sudo mkdir -p /home/recovery/logs
      
   ![image](https://user-images.githubusercontent.com/67065306/132923803-343472ec-6124-48c4-bbdf-e89d48b206cd.png)
   
   
 17. We will mount /var/www/html on apps-lv logical volume

         sudo mount /dev/webdata-vg/apps-lv /var/www/html/
 
    ![image](https://user-images.githubusercontent.com/67065306/132924133-449ace27-9117-40c0-99fa-1b924d50b093.png)
    
  18. We use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

         sudo rsync -av /var/log/. /home/recovery/logs/

   ![image](https://user-images.githubusercontent.com/67065306/132924257-9a2d5e8a-cd69-47ea-ac04-32949cd59bcb.png)
   
 19. We will mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. 
    That is why step 15 above is very important)
    
       sudo mount /dev/webdata-vg/logs-lv /var/log
     
    ![image](https://user-images.githubusercontent.com/67065306/132924531-0d5f5bf0-b07f-48dc-ad8e-4b040ffecd5c.png)
    
 20. Restore log files back into /var/log directory

       sudo rsync -av /home/recovery/logs/log/. /var/log

   ![image](https://user-images.githubusercontent.com/67065306/132924926-5bbd7b0c-5820-4a25-bd6c-06928eb38d8f.png)


 21. We will update /etc/fstab file so that the mount configuration will persist after restart of the server.
 
      UPDATE THE `/ETC/FSTAB` FILE
      
      The UUID of the device will be used to update the /etc/fstab file;

          sudo blkid
   ![image](https://user-images.githubusercontent.com/67065306/132925151-3bd28da8-aaff-439a-81f1-6f8511ec3d8e.png)
  
    sudo vi /etc/fstab

   We will update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.
   
   ![image](https://user-images.githubusercontent.com/67065306/132950939-d45f2fc4-76e9-46ca-b2a7-c6f0ee6d21a2.png)


   1. We will test the configuration and reload the daemon

      sudo mount -a
      
      sudo systemctl daemon-reload

  ![image](https://user-images.githubusercontent.com/67065306/132926144-f6a81239-1d2e-48e9-957e-2afdf68450c7.png)
     
   2. We will verify our setup by running df -h, output must look like this:

  ![image](https://user-images.githubusercontent.com/67065306/132951028-69bc135c-be50-46b3-9c8e-27f95dd3e72e.png)


**Step 2 — Prepare the Database Server**

 1. We will launch a second RedHat EC2 instance that will have a role – ‘DB Server’

 Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.
 
 2. We will create the 3 volumes for the dbserver, as detailed below.
 
 ![image](https://user-images.githubusercontent.com/67065306/132951974-545e668c-5af7-4cda-ad6d-6a59a5ac1e19.png)
 
 3. We will use gdisk utility to create a single partition on each of the 3 disks

  sudo gdisk /dev/xvdf then sudo gdisk /dev/xvdf then sudo gdisk /dev/xvdf
  
  The 3 partitions can be viewed below;

 ![image](https://user-images.githubusercontent.com/67065306/132952192-560055f7-b1d2-4071-822d-f2fdc73eab1d.png)
 
 4. We will next install lvm2 package using 
 
    sudo yum install lvm2
 
 ![image](https://user-images.githubusercontent.com/67065306/132952448-adda4770-170a-487b-9be7-679fbdcdb651.png)

  5. Next, will Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
      
      sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
 
  ![image](https://user-images.githubusercontent.com/67065306/132952572-e9b62308-c0e2-4e27-9192-559e6692d0c3.png)
  

 6. We will Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.
   
    sudo pvcreate /dev/xvdf1
    
    sudo pvcreate /dev/xvdg1
    
    sudo pvcreate /dev/xvdh1
    
   ![image](https://user-images.githubusercontent.com/67065306/132956560-5aac2b2c-5e4f-40bb-8722-5f7bf2ac650a.png)

 7. We then run, lsblk to see our configuration followed by sudo pvs.
     
   ![image](https://user-images.githubusercontent.com/67065306/132956482-316e15ef-9f5c-4015-9d9d-6f03da58b87f.png)
    
  8. Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

      sudo vgcreate vg-database /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
      
      to check, we run the following
      
      sudo vgs

    ![image](https://user-images.githubusercontent.com/67065306/132956916-b1e3d4ae-d8be-4d3e-a2be-fe07c5a2087c.png)
    
  9. Next, we use lvcreate utility to create a logical volume
     
        sudo lvcreate -n db-lv -L 20G vg-database
      
     We verify that our Logical Volume has been created successfully by running 

       sudo lvs
     
    ![image](https://user-images.githubusercontent.com/67065306/132957254-3c219504-000e-4423-a73a-5f9a4e534ebd.png)

  10. We will use mkfs.ext4 to format the logical volumes with ext4 filesystem

        sudo mkfs.ext4 /dev/vg-database/db-lv
        
    ![image](https://user-images.githubusercontent.com/67065306/132957391-8205569d-b2bc-4005-a269-3c842004a872.png)

  11. We will mount /dev/vg-database/db-lv on /db logical volume.
    
       sudo mount /dev/vg-database/db-lv /db
      
      ![image](https://user-images.githubusercontent.com/67065306/132957477-6996d504-3bbf-421c-b1c2-46b976ea856f.png)

 12. We will update /etc/fstab file so that the mount configuration will persist after restart of the server.
 
      The UUID of the device will be used to update the /etc/fstab file;

          sudo blkid
          
   ![image](https://user-images.githubusercontent.com/67065306/132959201-f32ba476-cbed-45cb-ac33-81d4b8d5cfb3.png)

    sudo vi /etc/fstab

   We will update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.
   
   ![image](https://user-images.githubusercontent.com/67065306/132959311-1c902538-daec-452f-a3cd-322aa578afd7.png)
  
   We will show some disk space stats by running
   
      df -h
      
   ![image](https://user-images.githubusercontent.com/67065306/132959370-ac8fa0f2-dc5b-4520-aa47-a60aa9036440.png)

   
   **Step 3 — Install WordPress on your Web Server EC2**
   
   1. We will update the repository on both the webserver and database server.

         on database server run,
         
         sudo yum -y update 
         
 ![image](https://user-images.githubusercontent.com/67065306/132959711-04189b54-66b9-4460-a120-0d97c7f37eeb.png)

         On webserver run
         
         sudo yum -y update
         
  ![image](https://user-images.githubusercontent.com/67065306/132959759-6d2cc7ae-c237-45bb-b4f0-40e6b51cc835.png)

   2. Install wget, Apache and it’s dependencies on web server.

         sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
   
   ![image](https://user-images.githubusercontent.com/67065306/132960833-3d0a8b5f-0e17-4637-a340-7118456f5c80.png)
   
   
   3. **First install the epel repository.**
   
   a. First install the epel repository
   
          sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
   
  ![image](https://user-images.githubusercontent.com/67065306/132961299-a1664ec7-1baa-4c8d-9624-7bc7b3a202c0.png)

    b. Next, install yum utils and enable remi-repository
      
           sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpmall
      
 ![image](https://user-images.githubusercontent.com/67065306/132986526-5a8e7d29-de5d-4c02-ac5e-82a000b99d2b.png)

    c. Next we run the following command to see the minimal installed versions.
        
        sudo dnf module list php
        
  ![image](https://user-images.githubusercontent.com/67065306/132986798-cdcae51b-66d7-49f7-b5b3-ddf8437ef8d6.png)
 
  d. The output indicates that the currently installed version of PHP is PHP 7.2e.
    To install the newer release, PHP 7.4 reset the PHP modules. (further details: https://www.tecmint.com/install-lemp-on-centos-8/)

         sudo dnf module reset php
         
         sudo dnf module list php
         
  ![image](https://user-images.githubusercontent.com/67065306/132987218-07f1f617-0071-4dd9-8c8c-cc980b257988.png)


  e. Now, having reset PHP modules, enable the PHP 7.4 module by running.

         sudo dnf module enable php:remi-7.4
  
  ![image](https://user-images.githubusercontent.com/67065306/132987482-714fdd86-68f7-4079-a485-fa15755a5731.png)
  
  f. Finally, install PHP, PHP-FPM (FastCGI Process Manager) and associated PHP modules using the command.
    
       sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
    
   ![image](https://user-images.githubusercontent.com/67065306/132987823-49698046-5e44-4baa-95ae-af4af3ace427.png)

  g. Run the command to check
     
       sudo dnf module list php
     
   ![image](https://user-images.githubusercontent.com/67065306/132987867-94efd853-0810-4130-aa10-fd5ec12d61ae.png)
  
    To check the version, run;
    
    php -v
    
   ![image](https://user-images.githubusercontent.com/67065306/132988121-5cc2e48c-7906-4494-92b7-dfa8df666b41.png)


  h. Perfect, we now have PHP 7.4 installed. Equally important, we need to start and enable PHP-FPM on boot-up.

       sudo systemctl start php-fpm
       
       sudo systemctl enable php-fpm
       
       sudo systemctl status php-fpm
       
 ![image](https://user-images.githubusercontent.com/67065306/132988233-e8a633b0-9ac2-4c64-b1ff-0aebf48c9a2d.png)
 
 
 i. To instruct SELinux to allow Apache to execute the PHP code via PHP-FPM run;

        sudo setsebool -P httpd_execmem 1
        
  ![image](https://user-images.githubusercontent.com/67065306/132988604-3a1e2e6a-b669-453c-94a4-606c642e14a6.png)
  

  j. Finally, restart Apache webserver for PHP to work with Apache web server.

         sudo systemctl restart httpd
         
   ![image](https://user-images.githubusercontent.com/67065306/132988657-102076d2-e946-42c5-ab50-b5c0f4070ebd.png)

5. We start Apache
    
       sudo systemctl enable httpd

       sudo systemctl start httpd
 
       sudo systemctl status httpd
     
 ![image](https://user-images.githubusercontent.com/67065306/132989148-1eb595e3-d20c-4e93-a5bb-6d256410ba23.png)

 Next we go to the web browser and insert the public IP and we can see apache is installed.
 
 ![image](https://user-images.githubusercontent.com/67065306/132989367-afd976cc-00e4-4c65-9990-9bd8647527f5.png)
 
 6. Download wordpress and copy wordpress to var/www/html

    a. mkdir wordpress
    
    b. cd   wordpress
    
    c. sudo wget http://wordpress.org/latest.tar.gz
    
   ![image](https://user-images.githubusercontent.com/67065306/132989735-26aa175d-3bee-4bbe-bd31-16a4e81ae93f.png)

    d. sudo tar xzvf latest.tar.gz
    
   ![image](https://user-images.githubusercontent.com/67065306/132989823-8c2a422f-e632-4a74-bdfb-9b6c185c1d04.png)

    e. sudo rm -rf latest.tar.gz
    
    f. cd wordpress, then 
        
       sudo cp -R wp-config-sample.php wp-config.php
    
   ![image](https://user-images.githubusercontent.com/67065306/132990619-ac369680-fc0f-4d93-9089-311256cbb9f6.png)

    g. sudo cp -R wordpress/.  /var/www/html/
    
    ![image](https://user-images.githubusercontent.com/67065306/132991021-aafe2f58-53d2-4dea-9b40-c689b935456d.png)

     
   7. **Configure SELinux Policies** (not done)

         sudo chown -R apache:apache /var/www/html/wordpress
         
         sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
         
         sudo setsebool -P httpd_can_network_connect=1
        
 **Step 4 — Install MySQL on your DB Server EC2**
 
           sudo yum update
           
           sudo yum install mysql-server

  ![image](https://user-images.githubusercontent.com/67065306/132991397-a49c6545-4723-493f-82b4-47fb7e4f4528.png)

   
    We Verify that the service is up and running by using sudo systemctl status mysqld.
    if it is not running, restart the service and enable it so it will be running even after reboot:

           sudo systemctl restart mysqld
           
           sudo systemctl enable mysqld
    
    ![image](https://user-images.githubusercontent.com/67065306/132991589-8bf4ccb6-6299-41fa-84af-09419d4a81bc.png)
    
 
 **Step 5 — Configure DB to work with WordPress**
 
 ![image](https://user-images.githubusercontent.com/67065306/132991890-92533081-efc6-4100-955d-af87ed41b729.png)
 
      create database wordpress;
      
      show databases;
      
      CREATE USER 'wordpress'@'%' IDENTIFIED WITH mysql_native_password BY 'wordpress';
      
      GRANT ALL PRIVILEGES ON *.* TO 'wordpress'@'%' WITH GRANT OPTION;
      
      flush privileges;
      
      select user, host from mysql.user;

 ![image](https://user-images.githubusercontent.com/67065306/132992442-bf382864-8751-465a-a93f-f0893f359eee.png)
 
  ![image](https://user-images.githubusercontent.com/67065306/132993632-c9f4591e-a0bd-46d0-8b50-c6b41e6eb249.png)

 **Step 6 — Configure WordPress to connect to remote database**
 
 Hint: Do not forget to open MySQL port 3306 on DB Server EC2.
 For extra security, we shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32
 
   a. cd /var/www/html/
   
   b. sudo vi wp-config.php
   
 ![image](https://user-images.githubusercontent.com/67065306/132997306-6d0bc14e-0d6e-4d63-afe7-c2616b8b2a98.png)
   
   c. sudo systemctl restart httpd
   
   d. rename the /etc/httpd/conf.d/welcome.conf
   
  ![image](https://user-images.githubusercontent.com/67065306/132997501-5c0e9bd6-000d-4deb-b339-eb8c48731acd.png)
  
  1. Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client

         sudo yum install mysql
         
         sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
         

 ![image](https://user-images.githubusercontent.com/67065306/132997770-d003f8f3-0c8b-4f03-8485-6f59ab164c6d.png)
 
 2. We verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.

 ![image](https://user-images.githubusercontent.com/67065306/132998080-941a434e-f2aa-44e2-a4e7-953887237d0f.png)

3. We change permissions and configuration so Apache could use WordPress:
     
       sudo chown -R apache:apache /var/www/html/
       
![image](https://user-images.githubusercontent.com/67065306/132998300-2f938e79-f0b8-4374-9ffa-3d2f9cf5fb90.png)


4. We enable TCP port 80 in Inbound Rules configuration for oour Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

5. We try to access from our browser the link to our WordPress http://<Web-Server-Public-IP-Address>/wordpress/
   
   
  ![image](https://user-images.githubusercontent.com/67065306/133080083-062469db-d8d5-4958-85d6-1aa7c7e80166.png)

  ![image](https://user-images.githubusercontent.com/67065306/133080239-e9f99713-9926-4c3b-9b04-2373ed13abcb.png)

  ![image](https://user-images.githubusercontent.com/67065306/133080322-57cbe9c6-4819-40e1-baf6-e89e3005077a.png)
  
   ![image](https://user-images.githubusercontent.com/67065306/133079681-c7e13889-ca93-475f-8a9f-9026b9e1792e.png)
   
   DB credentials filled in and logged in.

   ![image](https://user-images.githubusercontent.com/67065306/133079911-30dc06b7-dd31-4e73-ba80-2593ee86b6e4.png)

    
    
   
   

   
   
