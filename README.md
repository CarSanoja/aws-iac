# Udagram Deployment on AWS

http://demo-s-webap-exjohx4phhws-1733183803.us-east-1.elb.amazonaws.com/

## Introduction

This project deploys an App index html called Udagram to AWS using Infrastructure as Code (IaC) with CloudFormation. The infrastructure includes networking components, udagram servers, a bastion host, an S3 bucket for static content, and an Application Load Balancer.

## Architecture Overview

- **VPC**: A Virtual Private Cloud (VPC) with public and private subnets in two availability zones.
- **Subnets**: Two public and two private subnets for high availability.
- **NAT Gateway**: Provides internet access to instances in the private subnets.
- **Bastion Host**: Allows SSH access to instances in the private subnets.
- **Application Load Balancer (ALB)**: Distributes incoming HTTP requests to udagram instances.
- **Auto Scaling Group**: Ensures the udagram is always available by automatically adjusting the number of instances.
- **S3 Bucket**: Stores static content accessible by the udagram.

## Prerequisites

- AWS CLI installed and configured with appropriate permissions.
- A key pair created in the AWS region where the stack will be deployed.
- CloudFormation templates and parameter files.

## Files

- `network.yml`: CloudFormation template for networking resources.
- `udagram.yml`: CloudFormation template for udagram resources.
- `s3.yml`: CloudFormation for s3
- `index.html`: Static HTML file to be uploaded to the S3 bucket.

## Deployment Steps

### Step 1: Create the Network Stack

1. Create the network stack using the `network.yml` template.

    ```bash
    aws cloudformation create-stack --stack-name udagram-network --template-body file://network.yml --parameters file://network.json
    ```

2. Wait for the stack to be created. You can check the status using:

    ```bash
    aws cloudformation describe-stacks --stack-name udagram-network
    ```

### Step 2: Create the S3 and then the Application Stack

1. Create the udagram stack using the `udagram.yml` template.

    ```bash
    aws cloudformation create-stack --stack-name udagram-s3 --template-body file://s3.yml --parameters file://s3.json --capabilities CAPABILITY_NAMED_IAM
    ```

    Now upload the index.html to the s3

    ```bash
    aws cloudformation create-stack --stack-name udagram-app --template-body file://udagram.yml --parameters file://udagram.json --capabilities CAPABILITY_NAMED_IAM
    ```

2. Wait for the stack to be created. Check the status using:

    ```bash
    aws cloudformation describe-stacks --stack-name udagram-app
    ```

### Step 3: Upload Static Content to S3

1. Upload the `index.html` file to the S3 bucket created by the stack.

    ```bash
    aws s3 cp index.html s3://<your-s3-bucket-name>/
    ```

### Step 4: Connect to the Bastion Host

1. Connect to the bastion host using SSH.

    ```bash
    ssh -i path/to/dev2.pem ubuntu@<BastionHostPublicIP>
    ```

2. From the bastion host, connect to an instance in the private subnet.

    ```bash
    ssh -i /path/to/dev2.pem ec2-user@<PrivateInstanceIP>
    ```

## Clean Up

To clean up the resources, delete the CloudFormation stacks in reverse order.

1. Delete the udagram stack.

    ```bash
    aws cloudformation delete-stack --stack-name udagram-app
    ```

2. Delete the network stack.

    ```bash
    aws cloudformation delete-stack --stack-name udagram-network
    ```

## Troubleshooting

- Ensure that all AWS CLI commands are run with the correct permissions and in the correct region.
- Check the AWS CloudFormation console for detailed error messages if stack creation fails.
- Verify that the key pair name used in the parameters exists in the target region.

## Conclusion

This README provides the necessary steps to deploy and manage the Udagram udagram on AWS using CloudFormation. Following these steps will ensure a successful deployment and easy maintenance of the udagram infrastructure.
