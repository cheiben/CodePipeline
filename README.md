# AWS CodePipeline Web App Deployment

## Case Study: Deploying a Web Application from S3 Bucket to EC2 Instance Using CodePipeline

This guide walks through the process of setting up a continuous delivery pipeline to deploy web applications from an S3 bucket to EC2 instances using AWS DevOps services.

## Prerequisites

This solution utilizes the following AWS services:

- **AWS CodeCommit**: Fully-managed source control service that hosts secure Git-based repositories for team collaboration.
- **AWS CodeBuild**: Fully managed continuous integration service that compiles source code, runs tests, and produces deployment-ready software packages.
- **AWS CodeDeploy**: Fully managed deployment service that automates software deployments to various computing services including Amazon EC2.
- **AWS CodePipeline**: Fully configured continuous delivery service that automates release pipelines for fast and reliable application updates.
- **Amazon S3**: Used as the code repository for this deployment workflow.

## Step 1: Create an S3 Bucket for Your Application

1. Open the Amazon S3 console
2. Choose **Create bucket**
3. Enter a unique name for your bucket
4. Enable **Versioning** for the bucket
5. Complete the bucket creation with default settings
6. Download the sample code to your local computer

## Step 2: Create Amazon EC2 Windows Instances and Install the CodeDeploy Agent

### Create Required IAM Role

1. Create an IAM role with the following trust relationships:
   ```json
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
   ```

2. Attach the following policies to the IAM role:
   - `AmazonEC2RoleforAWSCodeDeploy`
   - `AWSCodeDeployRole`
   - `AmazonEC2ReadOnlyAccess`

### Launch EC2 Instance

1. Launch an EC2 instance with the following configurations:
   - Attach the IAM role created above
   - Enable **Auto-assign Public IP**
   - Add the following script in **User data**:
     ```powershell
     <powershell>
     New-Item -Path c:\temp -ItemType "directory" -Force
     powershell.exe -Command Read-S3Object -BucketName bucket-name/latest -Key codedeploy-agent.msi -File c:\temp\codedeploy-agent.msi
     Start-Process -Wait -FilePath c:\temp\codedeploy-agent.msi -WindowStyle Hidden
     </powershell>
     ```
     > **Note**: Replace `bucket-name` with the S3 bucket containing CodeDeploy Resource Kit files for your region. For example, in US West (Oregon) Region, use `aws-codedeploy-us-west-2`. [See region-specific bucket names](https://docs.aws.amazon.com/codedeploy/latest/userguide/resource-kit.html#resource-kit-bucket-names).

2. Configure security group to allow inbound traffic on port 80
3. Launch the instance with default configurations

## Step 3: Create an Application in CodeDeploy

1. Open the CodeDeploy console
2. Choose **Create application**
3. Select **EC2/On-premises** as the compute platform
4. Create a deployment group with the following settings:
   - Create an IAM service role for CodeDeploy
   - Select **In-place** deployment type
   - Choose **Amazon EC2 Instances** under Environment configuration
   - Select **CodeDeployDefault.OneAtaTime** as the deployment configuration
   - Disable load balancing

## Step 4: Create Your Pipeline in CodePipeline

1. Open the CodePipeline console
2. Choose **Create pipeline** and configure:
   - Enter a pipeline name
   - Select **New service role** to allow CodePipeline to create necessary IAM roles
3. Configure source stage:
   - Select **Amazon S3** as the source provider
   - Enter the S3 bucket and object key for your application code
4. Skip the build stage (choose **Skip build stage**)
5. Configure deploy stage:
   - Select **AWS CodeDeploy** as the deploy provider
   - Choose your application name and deployment group

## Verification

After creating the pipeline, verify the deployment by accessing your EC2 instance's public DNS address in a web browser. The application should be successfully deployed and running.

## Additional Resources

- [CodeDeploy Service Role Documentation](https://docs.aws.amazon.com/codedeploy/latest/userguide/getting-started-create-service-role.html)
- [CodeDeploy Resource Kit Bucket Names](https://docs.aws.amazon.com/codedeploy/latest/userguide/resource-kit.html#resource-kit-bucket-names)
- [AWS Identity and Access Management (IAM) Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
