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
            
**[4.1] Create IAM Role for the build Server</br>**
* asdfas
            
2- Create a secutiry group and Key Pair for Build server..
            
3- Creating & Configuring build server.
            
4- Connecting Jenkins-Master & Jenkins-Slave
            
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
    
Now make some changed in your README.md file and commit to github, you webhook-test jobshouldautomatically run
    
it should look like this→ [http://ec2-43-205-2-42.ap-south-1.compute.amazonaws.com/github-webhook/](http://ec2-43-205-2-42.ap-south-1.compute.amazonaws.com/github-webhook/)
    
- **[6]** **Deployment to Elastic Beanstalk**
    
    

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


