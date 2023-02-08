

We shall need 3 servers

### 1. Let us launch 3 EC2 Instances


- A staging server - Check and test
- A production server - main server
- A Jenkins server

![Our 3 servers](./images/our-three-servers.png)

### 2. Let us install Jenkins on our Jenkins instance

- SSH into our 3 instances
 
 ![ssh into jenkins instance](./images/jenkins-ssh.png)

### Let's update the system package repository of each instance
 ```
sudo apt-get update
 ```

 ### Next we shall install Java on our jenkins,staging and production nodes. The slave machines will need java to connect to jenkins
 ```
sudo apt install openjdk-11-jdk -y
 ```

### 3. Install Jenkins
```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update

sudo apt-get install jenkins
```
### 4. Check the jenkins installation status

![Check Jenkins installation](./images/jenkins-running.png)

### 5. Set up Jenkins. Open up port 8080 and go to browser at jenkinsserveripaddress:8080


![Jenkins set up](./images/jenkins-start.png)

### Retrieve the Jenkins default password

```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

![Jenkins set up](./images/jenkins-start2.png)

We shall select install suggested plugins

Create first Admin user

![Create Admin user](./images/jenkins-start3.png)

![Jenkins is ready](./images/jenkins-start4.png)


### ADD SLAVE NODES TO JENKINS 
- We shall add slave nodes to Jenkins using JNLP connections/protocol

- Manage Jenkins
-- Configure global security
---- Set TCP ports for inbound agents to random

-- Manage Nodes and Clouds

We have one node so far.   Built in node. (Our Jenkins Node which is the master.)

---- Let us add new nodes for staging and production

### SET THE FOLLOWING for both slave nodes

Remote home directory:    /home/ubuntu/jenkins

Launch method:   Launch agent by connecting it to the controller/master


![Nodes](./images/conf1.png)


![Nodes](./images/conf2.png)


![Nodes](./images/conf3.png)


![Nodes](./images/conf1a.png)


![Nodes](./images/conf2a.png)

Our slave nodes are marked as red above as they are unconnected 

We will need to get the connection code from each Jenkins slave node to run on our servers and enable the connection.

Let us run the above code from the command line for the staging and production instances
```
curl -sO http://3.236.127.45:8080/jnlpJars/agent.jar

java -jar agent.jar -jnlpUrl http://3.236.127.45:8080/manage/computer/Production/jenkins-agent.jnlp -secret 1bcc07b3adab01e2c33eb0131ef9d7b176f39292b049243ca464140640188b3b -workDir "/home/ubuntu/jenkins"
```

#### Nodes are now connected

![Nodes](./images/node-connected.png)


### Now that our nodes are connected, we can start creating a pipeline

### We shall copy a git repo to the slaves filesystem with Jenkins (Jenkins master)

### The website we shall be deploying will be containerized with docker using Jenkins

1. Pull source code from Git. Download in docker repository
2. Containerise code into Apache container
3. Test code
4. If successful, push to production


## OUR FIRST JOB - GIT JOB

- Select New Item

-- I will call it Git Job (for simplicity)
-- Choose Freestyle Project

-- From our terminal I will create a Git Repository called Ourwebsite and push website code to our remote Git repo.
```
mkdir website
```
```
cd website

sudo vi index.html
```

```
<!doctype html>
<html>
  <head>
    <title>Jenkins CI/CD Pipeline with Dele!</title>
  </head>
  <body>
    <p>This is a pipeline that uses Git, Docker and Jenkins. CI/CD is sich a <strong>beautiful thing.</strong> Reduce as much human error with <strong>automation.</strong> </p>
  </body>
</html>
```

```
git init

git add .

git branch -M main

git remote add origin https://github.com/deleonab/ourwebsite.git

git commit -m "Added website files"

git push origin main
```

![Website Git](./images/ourwebsitegit.png)

We shall now copy the repo link and continue our Jenkins configuration.

![Website Git](./images/ourwebsitegit2.png)


Sourcecode Management: Git

Repository url:  https://github.com/deleonab/ourwebsite.git

Branch specifier: */main

Build Triggers: checkbox select Github hook trigger for gitSCM polling

Restrict where job can be run: staging

![Website Git](./images/restrict.png)

![Website Git](./images/sourcecode.png)



