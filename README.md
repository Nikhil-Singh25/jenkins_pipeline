# Jenkins-AWS

## **[1]** **Create an security group** **(Jenkins-Master)**
    
   * Inbound rule- Allow port 22(SSH),80(HTTP) & 443(HTTPs)</br>
   * Outbount rule - All traffic
    
 ## **[2]** **Create an EC2 instance**(Jenkins-Master)
   - Create and EC2  Instance->`Jenkins-Master` & use AMI - ubuntu 22.2.0</br>
   - Create a Key pair(Save it in you local machine to SSH into Jenkins-Master Server)</br>
   - Attached the previously created sceurity group(Jenkins-Master)</br>
   - Create an Elastic IP and attach to the EC2 Instance</br></br>
 ## [3] **Configure EC2 instance (Jenkins-Master)**</br>
     → switch to super user `sudo su -`  &</br>
     → get updates `apt-get update` & upgrade the packages `apt-get upgrade -y` </br>
        
### [3.1] Installing and Configuring jenkins :
        
- Install java-17 → `apt install openjdk-17-jdk -y`
- Add Jenkins Repository to ubuntu system and install **Jenkins**
1.  Importing GPG Key, The GPG key verifies package integrity       
 ```bash
 curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc &gt; /dev/null
 ```
2.  Importing GPG Key, The GPG key verifies package integrity        
```bash
 echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list &gt; /dev/null
```
3. Add the Jenkins software repository to the source list and provide the authentication key       
 ```bash
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list &gt; /dev/null
```
4. installing jenkins (Update the system repository one more time. Updating refreshes the cache and makes the system aware of the new Jenkins repository)      
```bash
 apt update
 apt install jenkins -y
 ```
 5. Check the running status of Jenkins        
   ```
   systemctl status jenkins | grep Active
   ```  
### [3.2] Installing & Configuring Nginx :
        
- Install nginx 
    ```
    apt install nginx -y
    ```
- remove the default configuration file for Nginx, which is stored in the `/etc/nginx/sites-enabled` directory use command:
    ```
    unlink /etc/nginx/sites-enabled/default
    ```
- open the Vim text editor and creates a new configuration file for Nginx, specifically for Jenkins.
    ```
    vim /etc/nginx/conf.d/jenkins.conf
    ```   
    This file will be stored in the `/etc/nginx/conf.d`directory.
- using Vim, add the following lines to the configuration file:
               
               upstream jenkins {
               server 127.0.0.1:8080;
                }
            
                server {
                    listen 80 default_server;
                    listen [::]:80  default_server;
                    location / {
                        proxy_pass http://jenkins;
                        proxy_set_header Host $host;
                        proxy_set_header X-Real-IP $remote_addr;
                    }
                }
            
 - These lines configure Nginx to act as a reverse proxy for Jenkins. The `upstream` block defines a group of servers that Nginx can proxy requests to, in this case only one server running on `127.0.0.1` on port `8080`. The `server` block defines a virtual server that listens on port `80 (HTTP)` and forwards requests to the Jenkins server defined In the `upstream` block. The `location` block defines how requests to the root URL (/) should be handled, by passing them on to the Jenkins server and setting some HTTP headers in the process.    
            
- Run `nginx -t` to test the configuration file of the Nginx web server for syntax errors</br>
     When this command is executed, Nginx parses the configuration file and checks for any syntax errors or issues that might prevent Nginx from starting properly. If any errors are found, the command will print an error     message and exit with a non-zero status code.
            
    ![ngin -t result](https://github.com/Nikhil-Singh25/Images_logos/blob/main/Untitled.png)
            
- `systemctl reload nginx` → reload the configuration of the Nginx web server without stopping it.
        
## [4] Configuring Build Server (Jenkins-Slave)
            
**[4.1] Creating IAM role, Security group and Key pair for build-Server  </br>**
1. AWS Console → IAM console → "Roles" (in left navigation menu).
2. Click "Create role" button.
3. Choose "AWS service" as the trusted entity → "EC2" as the service that will use the role.
4. Click on "Next: Permissions" button. Search for the "AWS Elastic Beanstalk Full" it will give full access to the build-server.
5. (Optional) Add tags to the role for better organization and tracking → Click on "Next: Review" button.
6. Give the role a name and description, and review the policies attached to the rol & Click on the "Create role" button to create the role.
7. Go to the EC2 console and select the Jenkins-Slave Ec2 instance.
8. Click on the "Actions" button and select "Instance Settings" and then "Attach/Replace IAM Role" & Select the role you just created and click on the "Apply" button.
9. Create a secutiry group and Key Pair for Build server save the private key to you local system.</br>
    Security Group :</br>
        **In bound rules**: Allow SSH (80) from SG Jenkins-Master created for Jenkins-Master server & Allow  SSH from 0.0.0.0/0 .</br>
10. Create Jenkins-Slave EC2 instance with same configuration as Jenkins-Master add the IAM role choose Jenkins-Slave SG and Create a new key pair and save the .pem file.  

**[4.2]FConnecting Jenkins-Master & Jenkins-Slave</br>**
1. Start the jenkins-build server and connect using SSH and run following commands:
```bash
#!/bin/bash
#install dependecies 
yum -y group install "Development Tools"
yum -y install bzip2-devel.x86_64
yum -y install openjdk-17-jdk -y
yum -y install libffi-devel
yum -y install ncurses-devel
yum -y install openssl-devel
yum -y install python3
yum -y install readline-devel.x86_64
yum -y install sqlite-devel.x86_64
yum -y install zlib-devel

curl -0 https://bootstrap,pypa,io/get-pip.py
/usr/bin.python get-pip,py
/usr/local/bin/pip3 install awsebcli
```
→ The above commands can be used in user data so that when the build-server boots up everything is ready automatically.
2. Open Jenkins(Jenkins-Master) → (left menu) 'Build Executor Status'</br>
3. Follow the below step to create a new node:</br>
    1. Click on "Build Executors status"
        ![jen1](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/fb0e2709-0dd0-4112-a2af-9dc38d5faa26/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230303%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230303T225015Z&X-Amz-Expires=86400&X-Amz-Signature=a32d51812371903c9ded9e2d77e93ff098b0de63eb9d4ebc5acd345077302758&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

2. Copy the private Ipv4 address of jenkins-master server

## [5] Connect Jenkins with Webhooks.
**What is a webhook?**      
→ A webhook is a way for an application or service to provide real-time information to another application or service. It is a simple event-notification mechanism that sends data to a URL (also known as a webhook endpoint) when a particular event occurs.When an event occurs in the source application, the application will generate a message that contains information about the event and sends it to the webhook endpoint. The receiving application can then process the message and use the information to take further actions or trigger additional events.
        
    
**Creating a webhook and testing it for first time**:
    
 - Go to jenkins server → `New Item` → Name the job (webhook test) choose `freestyle`
        
→ General: paste the github url in the GitHub project option 
        
→SCM : choose `GitHub hook trigger for GITScm polling`
→Build Steps:  `Execute shell` →command `cat README.md` 
        
→ Apply & Save
        
- Create a github repository (keep it public)
- go to : settings → webhooks → add webhook → paste the DNS link of Jenkins-Master add `github-webhook/` at the end of url 
→ This url is endpoint in our jenkins server which will handle the webhook events(here,any update on github repo)
    
Now make some changed in your README.md file and commit to github, you webhook-test job sould automatically run
    
it should look like this→ [http://ec2-43-205-2-42.ap-south-1.compute.amazonaws.com/github-webhook/](http://ec2-43-205-2-42.ap-south-1.compute.amazonaws.com/github-webhook/)
    
- **[6]** **Deployment to Elastic Beanstalk**
    
    → Go to AWS Beanstalk console create a new Beanstalk application and name it `Jenkins-Beanstalk`

    → Create a Jenkins freestyle job:

    → General : Add the github url in github project .

    → Source Code Management : check- Git → **Repository URL** : add the url of  repo & change **Branch Specifier** : */main accordingly.
         In additional behaviours select **Check out to specific local branch**

    → Build Triggers : **GitHub hook trigger for GITScm polling**

    → Build Steps :  add the below code in the **command** and save

```bash
#!/bin/bash -xe

APPLICATION=Jenkins-Beanstalk
REGION=ap-south-1
ENVIRONMENT=Jenkinsbeanstalk-env
PLATFORM=python-3.8 

# get identity
aws sts get-caller-identity

# intialize the application
eb init "${APPLICATION}" --platform "${PLATFORM}"  --region "${REGION}"

# select the environment
eb use "${ENVIRONMENT}"

# deploy the application
eb deploy

# get the deployment's health and status information
eb health
eb status
```

→ Check your aws beanstalk console for environement updation you make chenges to the webapp file to see noticeable changes or can check the job console where the commit message can also confirm that everything is fine!
