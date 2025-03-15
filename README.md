# Devops Harness practice exercise
## What is this?
This repository is the result of me working on a devops exercise to practice Harness pipeline creations.

The instructions where as follows:

```
Create a Harness pipeline with the following requirements:
 - Clone the repository of a simple "Hello world" Node Js App
 - Generate a build and run tests on it
 - Upload artifact to Docker Hub/ECR
 - Deploy the app using Terraform
 - Include approval gates
```
This project consists on one main pipeline and two sub-pipelines: one for the Terraform provisioning, and the other one for the ECS deployment.

Here are the links to all three repositories:
- https://github.com/rafaelencinasr94/devops-harness-practice-nodejs
- https://github.com/rafaelencinasr94/devops-harness-practice-terraform
- https://github.com/rafaelencinasr94/devops-harness-practice-ecs

IMPORTANT: make sure to fork the repositories to your own GitHub account, otherwise you won't be able to import the pipelines to Harness.

## Terraform pipeline
This pipeline consists of a single "Infrastructure" stage, with separate steps for each "terraform init", "terraform plan", and "terraform apply" commands, with an intermediate approval step just before the plan is applied.

As part of Harness services, there is an "Infrastucture as Code" section, where users can create "workspaces" to manage their infrastructure provisioning, which makes it easy to know its state. A "workspace" is also required for Terraform provisioning as this is where the repository containing the terraform files is configured.

The terraform file creates the following elements:
- VPC
- 2 x public subnets
- Route tables and its associations
- Internet Gateway
- Security group
- Autoscaling Group
- EC2 launch template
- ECS cluster
- ECS capacity provider and its association
- ALB (implementation pending)

And has three outputs to be used later on by the ECS deployment pipeline:
- ECS cluster name
- Subnet id
- Security group id

## ECS deployment pipeline
IMPORTANT: In order for this pipeline and consequently the "NodeJs App" pipeline to work correctly a Harness Delegate needs to be installed inside an EC2 instance. Instructions for this can be found on the "EcsDeployment.md" file.

This pipeline deploys an ECR artifact to an ECS cluster, via a service running on Fargate. The instructions for the ECR repository creation and other information can be found in the "devops-harness-practice-ecs".

The ECS task and service definitions are created using Harness plugins and services, and are defined with two files stored within Harness file system:
- taskDefinition.json
- CreateServiceRequest.json

A template for both files can be found in the "devops-harness-practice-ecs" repository with instructions on how to create and modify them with your own AWS account ID. As you will see in the "CreateServiceRequest.json" the terraform output for the Subnet ID and the Security group ID are being used here.

Also in the service configuration a connection to the ECR artifact source is made.

To link the pipeline to the actual ECS cluster the connection is made in the "Environment" tab, more specificaly on the "Specify Infrastructure" section.

For the deployment strategy a simple rolling deployment is used but more options such as "Canary" and "Blue Green" are available.

## NodeJs App pipeline
This pipeline clones a GitHub repository containing the files of a simple "Hello World" NodeJs App, runs it, creates and Docker image from it, and then pushes it to Docker Hub and ECR. It then pulls the image from Docker Hub and runs it to check that everything works fine with a curl request.

Behind approval gates, it then implements the previous two sub-pipelines ("Terraform", and "ECS deployment") as separate stages.

# Common instructions and configuration
## AWS Connector
Throughout multiple parts of the pipeline configuration process you will need an "AWS Connector", here are the instructions to create it. 

Note: you can reuse the same connector so once you've configure it there's no need to create a new one each time.

1. The first time you are prompted for an AWS Connector click on the selection input and a modal window will appear.
2. Click on the "+ New Connector" button.
3. Type in "aws connector" as its name.
4. For credentials you may use whatever you want, here we will use "AWS Access Key". Select this option
5. On a new browser window open your AWS Console.
6. Navigate to the IAM service.
On the side panel navigate to "Policies", click on "Create policy"
7. Type in "harness-ecs-management-access" as the policy name and select the "JSON" editor and paste in the following policy:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:DescribeRepositories",
                "ecs:ListClusters",
                "ecs:ListServices",
                "ecs:DescribeServices",
                "ecr:ListImages",
                "ecr:DescribeImages",
                "ecs:RegisterTaskDefinition",
                "ecs:CreateService",
                "ecs:ListTasks",
                "ecs:DescribeTasks",
                "ecs:DeleteService",
                "ecs:UpdateService",
                "ecs:DescribeContainerInstances",
                "ecs:DescribeTaskDefinition",
                "application-autoscaling:DescribeScalableTargets",
                "application-autoscaling:DescribeScalingPolicies",
                "iam:ListRoles",
                "iam:PassRole",
                "iam:CreateRole"
            ],
            "Resource": "*"
        }
    ]
}
```
See here for more information about this policy: https://developer.harness.io/docs/continuous-delivery/deploy-srv-diff-platforms/aws/ecs/ecs-deployment-tutorial/#aws-iam-requirements

8. Create a new user.
9. When prompted to set permissions click on the "Create Group" button.
10. Type in the name for your group.
11. Include the following policies:
- AmazonEC2FullAccess
- AmazonECS_FullAccess
- EC2InstanceProfileForImageBuilderECRContainerBuilds
- harness-ecs-management-access
- IAMFullAccess
12. Click on the checkbox to add the user to the new group you've created.
13. Click on the user you've created and click on "Create access key"
14. Choose "Third-party service" and check the "I understand...."
15. Click on next until you see the "Retrieve access keys" screen.
16. Store your access keys somewhere safe
17. Back on Harness paste your access key on its input
18. Click on "Create or Select a Secret" input.
19. On the modal window, click on the "+ New Secret Text"
20. Type a name for your secret key
21. On the secret value paste your secret key.
22. Click on "Save" and "Apply selected"
23. In "Test Region" choose "US West (N. California). Click on "Continue".
24. Choose Fixed Delay.
25. Input "100" for "Fixed Backoff" (time in milliseconds), and "5" on "Retry Count". Click on "Continue".
26. Select "Connect thorugh Harness Platform". Click on "Save and Continue"
27. After the connection test is successful click on "Finish"

## Workspace
Before creating your Terraform and ECS deployment pipeline make sure to create a Workspace:
1. In Harness, navigate to the "Infrastructure as Code Management" service.
2. Click on "+ New Workspace"
3. Type in "aws_terraform" as its name.
4. For connector select the previously created AWS connector.
5. Choose "Terraform" for "Workspace Type" and the 1.5.7 version
6. In the "Repository" section choose "Third-party Git provider"
7. On Git Connector, click on the selector and create a new connector.
8. Choose "GitHub Connector", and type in "Devops Harness ECS connector" as its name. Click on "Continue".
9. For "URL Type" choose "Repository", and "HTTP" for its "Connecetion Type"
10. Input the URL the your fork of the [terreaform](https://github.com/rafaelencinasr94/devops-harness-practice-terraform) repository. Click on "Continue".
11. You may choose a different "Authentication" mode, but for here we will use "Username and Token"
12. Type in your GitHub username
13. Click on "Create or Select a Secret" and a modal window will appear.
14. Click on "+ New Secret Text"
15. Type in a name for your secret.
16. For the secret value you will need to create a GitHub access token (classic) with the following permissions scope: "admin:repo_hoo", "repo", "user", see here for more information: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic
17. Click on "Save", and then on "Apply Selected"
18. Select "Enable API access", you may use the same access token. Click on "Continue".
19. Choose "Connect through Harness Platform". Click on "Save and Continue".
20. Click on "Finish" after the connection test is successful.
21. The "Repository" and "Git Branch" should autopopulate with the name of the repository and "main" respectively
22. Leave "Folder Path" empty
23. Click on "Save"

## ECR repository creation
1. On the AWS console, navigate to the ECR service
2. Create a new private repository called "devops-harness"
3. For "Image tag mutability" keep the "Mutable" option selected
4. For "Encryption" choose AWS KMS -> AWS managed key.
5. Click on "Create"