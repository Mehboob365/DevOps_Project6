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

   
   Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
   
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

    
