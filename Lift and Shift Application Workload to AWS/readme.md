
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
### Step-3: Provision Backend EC2 instances with UserData script
### Step-4: Create Private Hosted Zone in Route53
### Step-5: Provision Application EC2 instances with UserData script
### Step-6: Create Artifact Locally with MAVEN
### Step-7: Create S3 bucket using AWS CLI, copy artifact
### Step-8: Download Artifact to Tomcat server from S3
### Step-9: Setup LoadBalancer
### Step-10: Create Route53 record for ELB endpoint
### Step-11: Configure AutoScaling Group for Application Instances
### Step-12: Clean-up




