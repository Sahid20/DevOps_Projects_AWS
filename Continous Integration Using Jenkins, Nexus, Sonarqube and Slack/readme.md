# Project: Continuous Integration Using jenkins, Nexus, Sonarqube and Slack


## Pre-requisities:

 * AWS Account
 * GitHub account
 * Jenkins
 * Nexus
 * SonarQube
 * Slack </br>
## Flow Chart
![flow-chart](https://user-images.githubusercontent.com/73986565/236692376-b9b9e72e-68e2-4bac-93a0-eb3d44d7e456.png)
</br>
## CI Pipeline
![cp](https://user-images.githubusercontent.com/73986565/236692545-921c80c0-bd40-4b36-97c9-d81105f4f6b5.jpg)

### Step-1: Create Keypair

 - A key-pair is required  while launching our instances. Create a keypair and download the private key to your local system. Make sure to remember where to download your key. We will need this key to SSh our servers.

### Step-2: Create Security Groups for Jenkins, Nexus and SonarQube

* Jenkins SecGrp
```sh
Name: jenkins-SG
Allow: SSH from MyIP
Allow: 8080 from Anywhere IPv4 and IPv6 (We will create a Github webhook which will trigger Jenkins)
```

* Nexus SecGrp
```sh
Name: nexus-SG
Allow: SSH from MyIP
Allow: 8081 from MyIP and Jenkins-SG
```

* SonarQube SecGrp
```sh
Name: sonar-SG
Allow: SSH from MyIP
Allow: 80 from MyIP and Jenkins-SG
```

-Once we created `sonar-SG`, we will add another entry to jenkins Inbound rule as Allow access on 8080 from sonar-SG. SonarQube will send the reports back to Jenkins.
