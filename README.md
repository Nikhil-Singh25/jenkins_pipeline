# Jenkins-AWS

- **[1]** **Create an security group** **(Jenkins-Master)**
    
    →Inbound rule- Allow port 22(SSH),80(HTTP) & 443(HTTPs)
    
    →Outbount rule - All traffic
    
- **[2]** **Create an EC2 instance**
    - Create and EC2  Instance->`Jenkins-Master` & use AMI - ubuntu 22.2.0
    - Create a Key pair(Save it to SSH into Jenkins-Master)
    - Attached the previously created sceurity group(Jenkins-Master)
    - Create an Elastic IP and attach to the EC2 Instance
- **[3]** **Configure EC2 instance (Jenkins-Master)**
    - → switch to super user `sudo su -`  &
     → get updates `apt-get update` & upgrade the packages `apt-get upgrade -y`
        
        ### **[1]** **Installing and Configuring jenkins :**
        
        - Install java-17 → `apt install openjdk-17-jdk -y`
            - Add Jenkins Repository to ubuntu system and install **Jenkins**
        
        ```bash
        curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc &gt; /dev/null
        ```
        
        ```bash
        echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list &gt; /dev/null
        ```
        
        ```bash
        echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list &gt; /dev/null
        ```
        
        ```bash
        apt update
        apt install jenkins -y
        ```
        
        ```bash
        systemctl status jenkins | grep Active
        ```
        
        ### **[2] Installing & Configuring Nginx :**
        
        - Install nginx : `apt install nginx -y`
        - `unlink /etc/nginx/sites-enabled/default` →removes the default configuration file for Nginx, which is stored in the `/etc/nginx/sites-enabled` ****directory
        - `vim /etc/nginx/conf.d/jenkins.conf` → This command opens the Vim text editor and creates a new configuration file for Nginx, specifically for Jenkins. This file will be stored in the **`/etc/nginx/conf.d`**
         directory.
        → In Vim, the following lines are added to the configuration file:
            
            ```bash
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
            ```
            
        - `nginx -t` →  Test the configuration file of the Nginx web server for syntax errors
                         → When this command is executed, Nginx parses the configuration file and checks for any syntax errors or issues that might prevent Nginx from starting properly. If any errors are found, the command will print an error     message and exit with a non-zero status code.
            
            ![Untitled](Jenkins-AWS%203683dc75bf734895ad814002410185bf/Untitled.png)
            
        - `systemctl reload nginx` → reload the configuration of the Nginx web server without stopping it.
        
        - 4. **Configuring Build Server (Jenkins-Slave)**
            
            1- Create IAM Role for the build Server
            
            2- Create a secutiry group and Key Pair for Build server.
            
            3- Creating ************************& Configuring build server.************************
            
            4- Connecting Jenkins-Master & Jenkins-Slave
            
- **[5]** **Connect Jenkins with Webhooks.**
    - **What is a webhook?**
        
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

Environment name → Jenkinsbeanstalk-env
