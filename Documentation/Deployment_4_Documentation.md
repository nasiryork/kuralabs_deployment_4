<h1 align=center>Deployment 4 Documentation</h1>

## Deployment goal:
Learn how to Deploy an application while Using Terraform to create the architecture 

## Software and Tools Used:
Terraform, VPC, EC2, Jenkins, GitHub, Nginx, Gunicorn, and Slack

## Set Up: 
- I first started this deployment off by creating a standard EC2 that will host the Jenkins Pipeline. 

## Jenkins Credentials: 
- Before I began making the Jenkins pipeline I needed to add credentials to the Jenkin. This would connect Jenkins with AWS and allow for access to create my infrastructure from my Terraform files.

## GitHub:
- Once my credentials were set up I forked the deployment repo. Inside was an intTerraform folder, a Jenkinsfile, and the application to deploy.
- The intTerraform Folder contained 3 files, one for the security group for a terraform instance, a deploy.sh file to deploy the application, and a main.tf file to create an instance.
- The Jenkins file initially had 5 stages which caused problems throughout the deployment. The Initial 5 stages were (Build, Test, Init, Plan, and Apply). The last 3 stages contained the credentials I made earlier and the Terraform commands to Initialize Terraform, Plan out the Architecture from the main.tf file, and Create the architecture.

## Jenkins Pipeline 1:
- The first pipeline that I set up was just to test if the application would deploy to an EC2 instance made from the terraform file. After fixing some configuration errors the application was deployed in the EC2.
![image](https://github.com/nasiryork/kuralabs_deployment_4/blob/main/static/First%20Pipeline.png)
- After the application was deployed I added a destroy stage to teardown the previous infrastructure. 
```
stage('Destroy') {
       steps {
        withCredentials([string(credentialsId: 'AWS_ACCESS_KEY', variable: 'aws_access_key'), 
                        string(credentialsId: 'AWS_SECRET_KEY', variable: 'aws_secret_key')]) {
                            dir('intTerraform') {
                              sh 'terraform destroy -auto-approve -var="aws_access_key=$aws_access_key" -var="aws_secret_key=$aws_secret_key"'
```
- Adding this stage pushed the resources on my EC2 too far which caused problems with the pipeline. 

## Terraform:
- Now that I confirmed that the application could run I now needed to create a VPC and add the Application Ec2 to it. I decided that I wanted to gain some more experience with using modules within Terraform. I then found a VPC module and configured it to fit the deployment. I changed the CIDR, Regions, Public subnets, and removed the Private subnet for this deployment.
- After I finished my configurations for the VPC module I added it to the main.tf file within GitHub. 

## Jenkins Pipeline 2:
- With the VPC added I was able to rebuild the pipeline. After fixing a few problems along the way I was able to get my application running within an architecture created entirely by Terraform.
![image](https://github.com/nasiryork/kuralabs_deployment_4/blob/main/static/D4%20Full%20Pipeline.png)
## Slack:
- I once again added the Slack Notification Plugin to my pipeline to inform me of the status of any future builds.

## Diagram:
![image](https://github.com/nasiryork/kuralabs_deployment_4/blob/main/Documentation/Deployment%204%20Diagram.drawio.png)
## Challenges:
- During this deployment I ran into 2 challenges that I needed to solve. 
- The First Challenge occurred when I was trying to add the Destroy Stage to my Jenkins pipeline.
- This problem was caused by the lack of resources provided by the t2.micro. I realized that if I wanted to use that specific instance type I would need to be efficient with the ram usage for my pipeline. The easiest fix would be to use an instance type with more resources, but that would also incur more charges.
-The second problem that I ran into occurred when creating the custom VPC architecture in Terraform.
- Initially, I was receiving an error during the apply stage of my Jenkins pipeline that stated that my ec2 instance could not be created since the security group and subnets were in different networks. The security group that was being created was automatically being placed within the default AWS VPC, meaning that I could not access it. I fixed this by adding this line to my security groups file:
```
vpc_id    = "vpc-082c41c746c8fc580”
```
This is a temporary fix since it requires the custom VPC to already be created.

## Final Thought:
- After completing this Deployment I now understand not only the importance of Terraform but how powerful modules can be. I was able to create an entire VPC with all of its components in minutes rather than manually creating everything which could take double or triple the time.
