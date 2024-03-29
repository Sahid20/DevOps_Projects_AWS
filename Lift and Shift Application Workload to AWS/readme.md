# Lift & Shift Application Workflow to AWS
### Prerequisites:

- &nbsp; AWS Account 

- &nbsp; Registered DNS Name

- &nbsp; Maven

- &nbsp; JDK8

- &nbsp; AWS CLI
### Flow of Execution
- &nbsp; Login to AWS Account
- &nbsp; Create Key Pairs
- &nbsp; Create Security Groups
- &nbsp; Launch Instances with user data (bash scripts)
- &nbsp; Update IP to name mapping route 53
- &nbsp; Build Application from Source Code
- &nbsp; Upload to S3 bucket
- &nbsp; Download Artifact to Tomcat Ec2 Instance
- &nbsp; Setup ELB with HTTPS (Cert from Amazon Certificate Manager)
- &nbsp; Map ELB Endpoint to website name IONOS
- &nbsp; Build Autoscaling Group for Tomcat Instances
### Architecture on DataCenter:

![architecture_data_centre](https://user-images.githubusercontent.com/73986565/210881651-6abe39f1-a972-47ea-ac37-8e788ca82996.png)

### Architecture on AWS:

![aws_architecture](https://user-images.githubusercontent.com/73986565/210882506-d4ff2b00-29f5-4e06-9644-a740ba3b7ffb.png)

### Step-1: Create Security Groups for Services
- &nbsp; We will create vprofile-ELB-SG first. We will configure Inbound rules to Allow both HTTP and HTTPS on port 80 and 443 respectively from Anywhere IPv4 and IPv6.
![ELB](https://user-images.githubusercontent.com/73986565/210899681-63fbc4cf-3923-43e4-8b11-7cf7559b2aba.PNG)
Inbound rules
![ELB_inbound_rules](https://user-images.githubusercontent.com/73986565/210899758-90c8f0b1-9b2b-4cdf-b227-7f608046e4ac.PNG)
- &nbsp; Next we will create vprofile-app-SG. We will open port 8080 to accept connections from vprofile-ELb-SG
![vprofile_app](https://user-images.githubusercontent.com/73986565/210899966-8d9b8af0-6d1d-4ab8-8468-1a60033c896e.PNG)
Inbound Rules
![vprofile_app_inbound_rules](https://user-images.githubusercontent.com/73986565/210900042-10a28564-f862-4e22-baaf-01a231637d69.PNG)

- &nbsp; Finally, we will create vprofile-backend-SG. WE need to open port 3306 for MySQL, 11211 for Memcached and 5672 for RabbitMQ server. We can check whcih ports needed fro aplication services to communicate each other from application.properties file under src/main/resources directory.We also need to open commucation AllTraffic from own SecGrp for backend services to communicate with each other.
![vprofile_backens_sg_security_group](https://user-images.githubusercontent.com/73986565/210900113-4aded4ac-98a4-4689-8bdf-6561d749390c.PNG)
Inbound Rules
![vprofile_backens_sg_security_group_inbound_rules](https://user-images.githubusercontent.com/73986565/210900197-b9f2cdb4-761e-4b21-ae14-161926008743.PNG)

### Step-2: Create KeyPair to Connect EC2 instancesWe 
- &nbsp; Now, I will create a Keypair to connect our instances via SSH.
![key_pair](https://user-images.githubusercontent.com/73986565/210902132-a44f8a4a-7947-46c6-9820-036dc1500974.PNG)
### Step-3: Provision Backend EC2 instances with UserData script
#### DB Instance:
- &nbsp;Create DB instance with below details.We will also add Inbound rule to vprofile-backend-SG for SSH on port 22 from MyIP to be able to connect our db instance via SSH. <br>
Name: vprofile-db01 <br>
Project: vprofile <br>
AMI: Centos 7 <br>
InstanceType: t2.micro <br>
SecGrp: vprofile-backend-SG <br>
UserData: mysql.sh <br>
- &nbsp;Once our instance is ready, we can SSH into the server and check if userdata script is executed.We can also check status of mariadb <br>
ssh -i vprofile-prod-key.pem centos@<public_ip_of_instance> <br>
sudo -i  <br>
curl http://169.254.169.254/latest/user-data <br>
systemctl status mariadb <br>

![mariadb_status](https://user-images.githubusercontent.com/73986565/210983682-1d98a883-a72b-4135-8df2-a8fb7f4966fd.PNG)

#### Memcached Instance:
##### Create Memcached instance with below details.
Name: vprofile-mc01<br>
Project: vprofile<br>
AMI: Centos 7<br>
InstanceType: t2.micro<br>
SecGrp: vprofile-backend-SG<br>
UserData: memcache.sh<br>
Once our instance is ready, we can SSH into the server and check if userdata script is executed.We can also check status of memcache service and if it is listening on port 11211.<br>
ssh -i vprofile-prod-key.pem centos@<public_ip_of_instance><br>
sudo -i
curl http://169.254.169.254/latest/user-data<br>
systemctl status memcached.service<br>
ss -tunpl | grep 11211<br>
![memcached_status](https://user-images.githubusercontent.com/73986565/210988528-641cca3f-3c69-4084-b105-ef530193e61b.PNG)


#### RabbitMQ Instance:
Create RabbitMQ instance with below details.<br>
Name: vprofile-rmq01<br>
Project: vprofile<br>
AMI: Centos 7<br>
InstanceType: t2.micro<br>
SecGrp: vprofile-backend-SG<br>
UserData: rabbitmq.sh<br>
Once our instance is ready, we can SSH into the server and check if userdata script is executed.We can also check status of rabbitmq service.
ssh -i vprofile-prod-key.pem centos@<public_ip_of_instance><br>
sudo -i<br>
curl http://169.254.169.254/latest/user-data<br>
systemctl status rabbitmq-server<br>
![rabbitmq_status](https://user-images.githubusercontent.com/73986565/210989844-a9116822-5dfd-4201-8cd0-81e44499dcdf.PNG)

Note: It may take some time to run userdata script after you connect to server. You can check the process ps -ef to see if the process start for service. If not wait sometime and check with systemctl status <service_name> command again

### Step-4: Create Private Hosted Zone in Route53
Our backend stack is running. Next we will update Private IP of our backend services in Route53 Private DNS Zone.Lets note down Private IP addresses.
rmq01 172.31.86.205<br>
db01 172.31.89.238<br>
mc01 172.31.89.107<br>
Create vprofile.in Private Hosted zone in Route53. we will pick Default VPC in N.Virginia region
![db01_record](https://user-images.githubusercontent.com/73986565/210995278-ac44ddf5-8f77-4ea2-9995-685e9d88644f.PNG)

- &nbsp;Now we will create records for our backend services. The purpose of this activity is we will use these record names in our application.properties file. Even if IP address of the services, our application won't need to change the config file.<br>
Simple Routing -> Define Simple Record<br>
Value/Route traffic to: IP address or another value
![hosted_zone](https://user-images.githubusercontent.com/73986565/210992778-b80033da-2367-4d55-a1ca-6cc423ebe131.PNG)
Details of records
![hosted_zone_details](https://user-images.githubusercontent.com/73986565/210993575-d92218b1-7953-40ad-8a76-c0fdaa3c3e32.PNG)
### Step-5: Provision Application EC2 instances with UserData script
Create Tomcat instance with below details.We will also add Inbound rule to vprofile-app-SG for SSH on port 22 from MyIP to be able to connect our db instance via SSH.
Name: vprofile-app01
Project: vprofile
AMI: Ubuntu 18.04
InstanceType: t2.micro
SecGrp: vprofile-app-SG
UserData: tomcat_ubuntu.sh



### Step-6: Create Artifact Locally with MAVEN
- &nbsp;Clone the repository.

Before we create our artifact, we need to do changes to our application.properties file under /src/main/resources directory for below lines.
jdbc.url=jdbc:mysql://db01.vprofile.in:3306/accounts?useUnicode=true&<br>

memcached.active.host=mc01.vprofile.in<br>

rabbitmq.address=rmq01.vprofile.in<br>
We will go to vprofile-project root directory to the same level pom.xml exists. Then we will execute below command to create our artifact vprofile-v2.war:<br>
mvn install

![buid_test](https://user-images.githubusercontent.com/73986565/211115024-92ad079d-74c1-4b9c-a905-3dd9b55c415d.PNG)

### Step-7: Create S3 bucket using AWS CLI, copy artifact
- &nbsp;We will upload our artifact to s3 bucket from AWS CLI and our Tomcat server will get the same artifact from s3 bucket.

- &nbsp; We will create an IAM user for authentication to be used from AWS CLI.<br>

name: vprofile-s3-admin<br>
Access key - Programmatic access<br>
Policy: s3FullAccess<br>
![iam](https://user-images.githubusercontent.com/73986565/210998323-863f99d9-eb32-46dd-96f5-3facf53c9647.PNG)

- &nbsp;Create bucket. Note: S3 buckets are global so the naming must be UNIQUE!
- &nbsp;Go to target directory and copy the artifact to bucket with below command. Then verify by listing objects in the bucket
- &nbsp;We can verify the same from AWS Console.
![s3bucket](https://user-images.githubusercontent.com/73986565/210998620-517e0467-a8f2-4543-92a8-3f5009e91daa.PNG)

### Step-8: Download Artifact to Tomcat server from S3
- &nbsp;In order to download our artifact onto Tomcat server, we need to create IAM role for Tomcat. Once role is created we will attach it to our app01 server.

Type: EC2<br>
Name: vprofile-artifact-storage-role<br>
Policy: s3FullAccess<br>
![artifact_roles](https://user-images.githubusercontent.com/73986565/211116007-9d09382f-bfde-4213-8c01-cb506b838ac9.PNG)
Before we login to our server, we need to add SSH access on port 22 to our vprofile-app-SG.<br>

Then connect to app011 Ubuntu server.

ssh -i "vprofile-prod-key.pem" ubuntu@<public_ip_of_server><br>
sudo su -<br>
systemctl status tomcat8<br>
We will delete ROOT (where default tomcat app files stored) directory under /var/lib/tomcat8/webapps/. Before deleting it we need to stop Tomcat server.<br>
cd /var/lib/tomcat8/webapps/<br>
systemctl stop tomcat8<br>
rm -rf ROOT<br>
Next we will download our artifact from s3 using aws cli commands. First we need to install aws cli. We will initially download our artifact to /tmp directory, then we will copy it under /var/lib/tomcat8/webapps/ directory as ROOT.war. Since this is the default app directory, Tomcat will extract the compressed file.
apt install awscli -y<br>
aws s3 ls s3://vprofile-artifact-storage-rd<br>
aws s3 cp s3://vprofile-artifact-storage-rd/vprofile-v2.war /tmp/vprofile-v2.war<br>
cd /tmp<br>
cp vprofile-v2.war /var/lib/tomcat8/webapps/ROOT.war<br>
systemctl start tomcat8<br>
We can also verify application.properties file has the latest changes.<br>
cat /var/lib/tomcat8/webapps/ROOT/WEB-INF/classes/application.properties<br>
We can validate network connectivity from server using telnet.<br>
apt install telnet<br>
telnet db01.vprofile.in 3306<br>
![telnet_db01](https://user-images.githubusercontent.com/73986565/211119100-c0dd0ca2-521c-4cee-b708-a09098608016.PNG)
telnet mc01.vprofile.in 11211
![telnet_mc01](https://user-images.githubusercontent.com/73986565/211119192-576ed3f5-8998-4670-a9b0-e7dbf3120021.PNG)

### Step-9: Setup LoadBalancer and DNS
- &nbsp;- &nbsp;Before creating LoadBalancer , first we need to create Target Group.<br>
Intances<br>
Target Grp Name: vprofile-elb-TG<br>
protocol-port: HTTP:8080<br>
healtcheck path : /login<br>
Advanced health check settings<br>
Override: 8080<br>
Healthy threshold: 3<br>
available instance: app01 (Include as pending below)<br>
![target_group](https://user-images.githubusercontent.com/73986565/211121319-1580dbb7-f809-435c-a3a3-263f6e67a157.PNG)
![target_group_vprofile_app_tg](https://user-images.githubusercontent.com/73986565/211121704-01dc7fd2-7a35-4945-a220-4dc751ac5203.PNG)

- &nbsp;Now we will create our Load Balancer.<br>
vprofile-prod-elb<br>
Internet Facing<br>
Select all AZs<br>
SecGrp: vprofile-elb-secGrp<br>
Listeners: HTTP, HTTPS<br>
Select the certificate for HTTPS<br>
![load_balancerr](https://user-images.githubusercontent.com/73986565/211122044-96dd9897-0c9d-49fc-bede-50d2d86019f6.PNG)


CNAME record
![cname_record](https://user-images.githubusercontent.com/73986565/211122807-74bc8a77-1977-44b5-b96b-2b2026c3458c.PNG)
### Step-10: Create Route53 record for ELB endpoint
- &nbsp;We will create an A record with alias to ALB so that we can use our domain name to reach our application.Lets check our application using our DNS.
- 
 We can securely connect to our application!
![admin](https://user-images.githubusercontent.com/73986565/210998890-b4c53582-3f6a-4c36-9b05-fc0c7619324c.PNG)

### Step-11: Configure AutoScaling Group for Application Instances
launch Configuration
![launch_configuration](https://user-images.githubusercontent.com/73986565/211124527-84d81031-d77f-41ac-b2e7-2ea77264bcc0.PNG)


- &nbsp;We will create an AMI from our App Instance.
Next we will create a Launch template using the AMI created in above step for our ASG.
Name: vprofile-app-LT
AMI: vprofile-app-image
InstanceType: t2.micro
IAM Profile: vprofile-artifact-storage-role
SecGrp: vprofile-app-SG
KeyPair: vprofile-prod-key
Our Launch template is ready, now we can create our ASG.
Name: vprofile-app-ASG
ELB healthcheck
Add ELB
Min:1
Desired:2
Max:4
Target Tracking-CPU Utilization 50
If we terminate any instances we will see ASG will create a new one using LT that we created.
### Step-12: Clean-up
- &nbsp;Delete all resources we created to avoid any charges from AWS.
 

