---
title: "AWS Notes"
date: 2019-12-01T19:16:47+05:30
draft: true
tags: [
    "AWS"
]
---
Personal notes

1. Each region contains minimum 3 AZ (availability zone). The purpose of the AZ is to provide disaster recovery.
2. AWS console and UI is region scoped except for       S3
3. IAM stands for Identity and Access Management. It is meant for security. It contains following
    a. Users: These are meant for identifying the physical person. 1 IAM user per physical person.
    b. Groups: Group of users
    c. Roles: Given to the machines. 1 role per application.
4. One should never use root account, it is meant for just account creation.
5. IAM contains policies written in JSON format. Policies are rules about what a user, group, role can and cannot do.
6. IAM has a global view, hence any created user, group or role is available across all the regions.
7. IAM Federation is a way by which large enterprise can connect their active directories with AWS. This enables companies to login into AWS using their company credentials. IAM federation use SAML (active directory) standard.  
8. PEM stands for privacy enhanced mail. It stores DER in base 64 encoded format. One of the use is to SSH into a machine. 
9. Steps to connect to a ec2 instance:
    a. Run `ssh ec2-user@<public IP of the machine>`. ec2-user is the user using which we can connect.
    b. This will not work as we have not provided as it is supposed to work only with a pem file. Provide it with option `-i` which stands for identity file.
    c. Run `ssh -i <pem file path> ec2-user@<public IP of the machine>`. Even this will not work and will fail with an error that the pem file permissions are too open. It is by default 644
    d. Run `chmod 400 <pem file path>`. This will make file read only.
    e. Now run `ssh -i <pem file path> ec2-user@<public IP of the machine>`. EC2 machine's console should get started
    f. To connect from Windows machines prior Windows 10, use putty as ssh is not available. However above Windows 10, either putty/ssh can be used.
    g. There is smart option to connect which is `EC2 Instance Connect`. This allows one to control the EC2 from browser.
10. ssh stands for secure shell    
11. Unix file permissions format:
    -               -    -     -           -    -     -              -    -     -    
    file/directory  read write execute     read write execute        read write execute
                    For owner              For user group            For others   
    -: file         r: read
    d: directory    w: write
                    x: execute
                    -: no permission
12. 1: execute
    2: write
    3: execute + write
    4: read
    5: read + execute
    6: read + write
    7: read + write + execute
13. Running command `chomod 742 sample` will apply 
    a. Read + write + execute for owner
    b. Read for user group
    c. Write for others   
14. Security group
    a. It controls the inbound and outbound traffic between EC2 instance and the external world. b. Security group controls
        i. Type of the communication (http/ssh/...)
        ii. protocol (TCP/UDP/...)
        ii. Port which is exposed
        iii. From where you can connect (Source IP)
    c. One EC2 can have multiple security groups, one security group can be configured with multiple EC2
    d. These are scoped by a region and  a VPC (virtual private cloud)
    e. Security group lives outside of EC2 and hence any request rejected from security group won't be visible in EC2 instance.
15. On restarting EC2 instance the public IP changes (unless it is configured to be an elastic IP)
16. Install Apache web server on EC2
    a. `sudo su`: This will switch user from `ec2-user` to `root`
    b. `yum update -y`: Update packages
    c. `yum install httpd.x86_64 -y`: Install http deamon
    d. `systemctl start httpd.service`: To start the http deamon
    e. `systemctl enable httpd.service`: To start the http deamon on instance startup.
    f. `echo "Hellow world from $(hostname -f)" > /var/www/html/index.html`: This will write the hello world message to index.html which will be returned by Apache server.
    g. `curl localhost:80`: Will hit the Apache server test page and respond with html content.
    h. Hit `http:<public ip>:80` from your browser to access the test page from outside world. It will not work as security group will block it.
    h. Add a rule in security group to allow `http` connection over port `80` from `anywhere` and try again, a test page will be displayed in the browser.
  17. The entire commands run above can be automated using `EC2 user data`. Put all commands in user data while creating the EC2 instance and it will run while booting the EC2.  


