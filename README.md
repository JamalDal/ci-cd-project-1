
# CICD pipeline on AWS

Creating a CICD pipeline on AWS.


I will be using five servers on AWS to build a complete CICD pipeline. I will name them as Developer, Jenkins, Ansible, and Webserver. I will be using a GitHub account instead of building a server for code management.

![image](https://user-images.githubusercontent.com/97054844/187052117-db137fd9-1f02-419d-b434-caf787fad48d.png)

Let’s make Jenkins server ready using cli
To install Jenkins, we need to install java first


```bash
  yum install java* -y
```

then Jenkins packages 

```bash
  wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
  rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
  yum install jenkins
```

Then start the daemon and enable it.
```bash
  systemctl start jenkins
  syatemctl enable jenkins

```

After all this configuration, access the Jenkins through browser using <public-ip>:8080
To get the password read the given file location after lauching Jenkins in the browser
like cat /var/lib/jenkins/secrets/initialAdminPassword


Now, create a user after the setup. 
username: 
password: 
Now access the Jenkins through url
http://<public-ip>:8080

Now, we will access Ansible server using cli and then install ansible package

```bash
  yum install ansible
  sudo amazon-linux-extras install ansible2
```
make an entry of webserver in hosts file of Ansible server
```bash
  vi /etc/ansible/hosts
```
Create a group and then attach private ip of webserver
[Web]
< private-ip >
Similarly, we can add multiple servers’ ip here.

![web](https://user-images.githubusercontent.com/97054844/187052069-45ce4216-7237-47ba-9bc2-5911014cf243.png)

Now ready webserver: Install apache package and start the daemon.
```bash
  Yum install httpd
  Systemctl start httpd
  Systemctl enable httpd
```
To check html file if there is any
```bash
  cd /var/www/html/
```

Before going ahead, let’s make the connection password less between all servers, and to do that, we need a user on each server. So, first create a user named root on all servers and assign a password.
```bash
  passwd root (user name: root, assign the password)
```
After that we need to make some changes on ssh_cofig file.
```bash
  vim /etc/ssh/sshd_config
  permitRootLogin yes 
  passwordAuthentication yes
```
to reflect these changes on the servers let’s restart ssh service
```bash
  systemctl restart sshd
```
![auth](https://user-images.githubusercontent.com/97054844/187052072-3b174247-69c8-40a5-b139-25c23923aca5.png)

### SSH KEY PART
Then generate a ssh key and share it 
```bash
  ssh-keygen
  ssh-copy-id -i root@<private-ip>
```
ssh key part is a critical step that must be completed on the development server for Jenkins, on Jenkins for Ansible, and on Ansible for Webserver. 

Now, let’s try something more and create a playbook at ansible
```bash
  mkdir sourcecode
  cd  /sourcecode >> vi playbook.yml
```
![playbook](https://user-images.githubusercontent.com/97054844/187052079-feda16db-eb2f-4153-88b9-7d7a5c731fb6.png)

Now, let’s go to GitHub and create a repo.

Create a file and name it index.html as we have mentioned in the playbook.


## INTEGRATE GITHUB WITH JENKINS
Now let’s integrate GitHub with the Jenkins server.

Take the Jenkins url (http://3.92.197.190:8080/) and go to GitHub (in particular, the particular repo).

Add webhook:
http://<jenkins-public-ip>:8080/github-webhook/

In secret: go to Jenkins and generate token
Jenkins> user> configure> add new token> generate
(114b22a149083c2b02dde4d4e1c######) > apply > save
Then go to git and add token (secret)
Then add webhook


Now, it is time to install plugin on Jenkins
Manage plugins
Add> Public Over SSH
Also install git package on Jenkins, otherwise we will get an error while integrating with GitHub.
```bash
  yum install git 
```
Now let’s crate a job > new items and type: Freestyle 
>SCM > GIT
(put source code’s GitHub url)
https://github.com/JamalDal/ci-cd-project-1.git

>>go to Build Triggers > select “GitHub hook trigger for GITScm polling”

### MANAGE JENKINS
>>> Manage Jenkins > configure system >> SSH Servers (at the end) > add > Jenkins > and in address-provide Jenkins server's private-IP in hostname > name: root >Use password authentication or a different key in advance.

This is the same password we have created for the root user on the Jenkins server.

Success> apply > save

NOTE: The same process we need to do for Ansible as well.

>> go to Build >> Add build > select “Send files or execute commands over SSH”
Select Jenkins
Exec command
```bash
  rsync -avh /var/lib/jenkins/workspace/ci-cd-project-1/*.html  root@<private-ip-of-ansible-server>:/opt/index.html
```
apply > save
The above process was done because I wanted to send the build file to the Ansible server.


>>go to Post-build Actions > Send build artifacts over SSH
Select ansible 
Exec command
```bash
  ansible-playbook /sourcecode/playbook.yml
```
The above process was done because I wanted to execute a build to deploy on a webserver.
Our final step:

Go to the developer server and install git.

Then clone the git repo of the project.

git clone https://github.com/JamalDal/ci-cd-project-1.git

Go to the index.html file and make some changes, then commit and push it to the GitHub repo.


## output

https://user-images.githubusercontent.com/97054844/187052201-0029f961-9f71-48e4-813d-3ae80c23ca2c.mp4

```bash
  The End. Thank you!
```



### References:

https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/

https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-jenkins.html

https://stackoverflow.com/questions/37869632/jenkins-execute-aws-cli-command-inside-a-pipeline-jenkins-file
