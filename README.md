# CodePipeline
Case Study: How To Deploy Web App From S3 Bucket To EC2 Instance on AWS Using CodePipeline
   
# AWS Services needed or prerequisite

+ AWS CodeCommit :  It is a  fully-managed source control service that hosts secure Git-based repositories. CodeCommit makes it easy for teams to collaborate on code in a secure and highly scalable ecosystem.
  
+ AWS CodeBuild : A fully managed continuous integration service that compiles source code, runs tests and produces software packages that are ready to deploy, on a dynamically created build server.

+ AWS CodeDeploy :  A fully managed deployment service that automates software deployments to a variety of computing services such as Amazon EC2, AWS Fargate, AWS Lambda, and your on-premises servers.
  
+ AWS CodePipeline :  A fully configured continuous delivery service that helps the user to automate their released pipelines for fast and reliable application and infrastructure updates.
  
+S3 will be used as code repository
----------------------------------------------------------------------------
Step 1: Create an S3 bucket for your application

1) Open the Amazon S3 console and Choose Create bucket and In Bucket name, enter a name for your bucket, and also don’t forget to enable Versioning.

2) Next, download the sample code and save it into a folder or directory on your local computer.
--------------------------------------------------------------------------------
Step 2: Create Amazon EC2 Windows instances and install the CodeDeploy agent

1) Create an IAM role that will be required to grant permission to EC2 instance.

Add or Edit default Ec2 trust relationships with following:

ref: https://docs.aws.amazon.com/codedeploy/latest/userguide/getting-started-create-service-role.html

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Principal": {
                "Service": ["codedeploy.amazonaws.com"]
            },
            "Action": "sts:AssumeRole"
        }
    ]
}

++++ Attach following policies to IAM Role

AmazonEC2RoleforAWSCodeDeploy
AWSCodeDeployRole
AmazonEC2ReadOnlyAccess

2) Launch instance on which our code will be deployed.
   
3) Just remember to add the IAM role that we have created. and In Auto-assign Public IP, choose Enable. Expand Advanced Details, and in User data, As text selected, enter the following:
<powershell>
New-Item -Path c:\temp -ItemType “directory” -Force
powershell.exe -Command Read-S3Object -BucketName bucket-name/latest -Key codedeploy-agent.msi -File c:\temp\codedeploy-agent.msi
Start-Process -Wait -FilePath c:\temp\codedeploy-agent.msi -WindowStyle Hidden
</powershell>

Note: bucketname: is the name of the S3 bucket that contains the CodeDeploy Resource Kit files for your region. For example, for the US West (Oregon) Region, replace the bucket-name with AWS-code deploy-us-west-2. For a list of bucket names.

ref: https://docs.aws.amazon.com/codedeploy/latest/userguide/resource-kit.html#resource-kit-bucket-names

4) On the Configure Security Group page, allow port 80 communication so you can access the public instance endpoint. Then follow the default configuration and launch the instance

Step 3: Create an application in CodeDeploy


1) Initially create an application in CodeDeploy, and In Compute Platform, choose EC2/On-premises.Choose to Create application.

2) On the page that displays your application, choose to Create a deployment group. In service, role creates an IAM role under code deploy category. Under Deployment type, choose In-place.

3) Under Environment configuration, choose Amazon EC2 Instances.

4) Under Deployment configuration, choose CodeDeployDefault.OneAtaTime.

5) Under Load Balancer, clear Enable load balancing, leave the defaults then choose to Create a deployment group.


Step 4: Create your first pipeline in CodePipeline

1) Open the CodePipeline console. Choose pipeline settings, Enter your desired name and in Service role, Choose New service role to allow CodePipeline to create a new service role in IAM.  To know more about AWS IAM refer to our blog on AWS Identity And Access Management (IAM).

2) In the Add source stage, select Source provider, choose Amazon S3. Under the S3 object key, enter the object key with or without a file path, and remember to include the file extension.

3) In the Add build stage, choose to Skip build stage, and then accept the warning message by choosing Skip again. Choose Next.

4) In the Add deploy stage, in Deploy provider, choose AWS CodeDeploy.Then enter your application name or choose the application name from the list. In the Deployment group, enter MyDemoDeploymentGroup, or choose it from the list, and then choose Next.


   FINALLY : Congratulations! You just created a simple pipeline in CodePipeline. you can verify that by coping EC2 Public DNS address and then past it into the address bar of your web browser
