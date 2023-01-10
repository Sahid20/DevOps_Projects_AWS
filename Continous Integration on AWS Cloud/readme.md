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
