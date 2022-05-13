#### Tooling Website deployment automation with Continuous Integration. Introduction to Jenkins

This project further builds upon Project 8 by incorporating one of the most popular CI/CD tools -Jenkins. In this project, Jenkins is used to make sure that every change made to the GitHub source code will be automatically updated to the Tooling website.

#### Task: Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server

```
sudo apt update
sudo apt install default-jdk-headless
```
Install Jenkins:
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
Ensure that Jenkins is up and running:
```
sudo systemctl status jenkins
```
![Project 9](https://user-images.githubusercontent.com/41236641/130793902-2d9904e8-5c68-481f-997d-80b131d15b94.JPG)

- Allow traffic inbound on Port 8080 on the Jenkins server. Jenkins is now accessible on the browser at http://(Jenkins-Server-Public-IP-Address-or-Public-DNS-Name):8080
To retrieve the default admin password:
 ```
 sudo cat /var/lib/jenkins/secrets/initialAdminPassword
 ```
 
  Install selected plugins. 
  After the plugin installations, create an admin user. Jenkins will then provide a URL.
  The Jenkins setup is now complete.
  
  #### Step 2 - Configure Jenkins to retrieve source codes from GitHub using Webhooks
 
 Here we configure a Jenkins Build that will be triggered by GitHub webhooks. The Build will execute a task to retrieve code from the GitHub repository and store it locally on the Jenkins server. 
 
First step is to enable webhooks in the GitHub repository settings:
Next, on the Jenkins web console, click “New Item” and create a “Freestyle project” and name it "tooling_github"

Configure the project to connect with your GitHub repository by entering the URL and credentials. Save the configuration and click on the "Build Now" link. 

The Workspace for the first job has retrieved all the files/directories from the GitHub repository and stored it locally.

This build is triggered only when activated manually. 

###### Adding additional configurations to the project "tooling_github"

Click “Configure” your job/project and add these two configurations: 

Configure triggering the job from GitHub webhook:

Configure “Post-build Actions” to archive all the files - files resulted from a build are called “artifacts”

The changes to the GitHub master branch automatically trigger a new build on the Jenkins server and the artifacts are updated and archived. The updated artifacts can also be accessible at ``` ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/ ```

#### Step 3 - Configure Jenkins to copy files to NFS server via SSH

The next step is to copy the artifacts to the NFS server to /mnt/apps directory. For this, a plugin called "Publish over SSH" can be used. 

###### Install “Publish Over SSH” plugin 
On main dashboard select “Manage Jenkins” and choose “Manage Plugins” menu item.

On “Available” tab search for “Publish Over SSH” plugin and install it:

###### Configure the job/project to copy artifacts over to NFS server

On main dashboard select “Manage Jenkins” and choose “Configure System” menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

![important forconnectingoverssh](https://user-images.githubusercontent.com/41236641/130794524-48e2a6d7-53fe-458a-ad1b-c03084d2efbb.JPG)

Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
Arbitrary name
Hostname - can be private IP address of your NFS server
Username - ec2-user (since NFS server is based on EC2 with RHEL 8)
Remote directory - /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

![project 9g](https://user-images.githubusercontent.com/41236641/130794570-f314d588-85f5-4dda-8351-e613e053b4a6.JPG)


###### Save the configuration, open your Jenkins job/project configuration page and add another one “Post-build Action”

Configure it to send all files produced by the build into the previously defined remote directory. In our case we want to copy all files and directories - so we use **. 
Save this configuration and change something in README.MD file in your GitHub Tooling repository.
At this point. when I check Jenkins, I encountered a permission denied error:

Debugging: I tried changing the ownership of /mnt/apps directory on the NSF server:
```
sudo chown -R ec2-user:ec2-user /mnt/apps
```

This did not help. Then I tried changing the ownership of the /var/www on both webservers:
```
sudo chown -R ec2-user:ec2-user /var/www
```

This was the problem. The Build was successful upon running the above command.
![project9done](https://user-images.githubusercontent.com/41236641/130794683-0d410231-5a5b-4d17-9f77-7a562be59776.JPG)
![image](https://user-images.githubusercontent.com/41236641/130794843-45371e21-86c4-455f-b584-ddf4943a186a.png)
