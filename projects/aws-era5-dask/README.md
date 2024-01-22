# Dask on ECS/Fargate Environment

This folder contains a sample CloudFormation template and Dockerfile for creating a containerized Dask environment on Amazon ECS and Fargate.

## Quick Deployment

If you are running this as part of an AWS managed workshop that does not already have the environment deployed, click on the following link to proceed to the "Quick create stack" screen in the AWS CloudFormation console:

https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://era5-workshop-data.s3.amazonaws.com/dask-environment.yaml&stackName=dask-environment

Review the parameters, acknowledge the capabilities then click "Create Stack".

Once the stack is deployed, you will need to open the HTTP port in the Dask Scheduler security group, by following the instructions here: https://catalog.workshops.aws/climatedata/en-US/lab-2/own#open-the-security-group-so-you-can-access-the-dask-dashboard 

## Downloading the template
As an alternative, you can download the CloudFormation template from [dask-environment.yaml](dask-environment.yaml) and proceed from step 2 here: https://catalog.workshops.aws/climatedata/en-US/lab-2/own.
