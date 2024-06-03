# DABs CI/CD Pipeline Example on AWS

This example demonstrates how to set up a CI/CD pipeline for deploying Databricks jobs and pipelines using Databricks Asset Bundles (DABs) on the AWS platform.

## Solution Introduction

This solution provides an AWS CloudFormation template to provision the CI/CD pipeline resources on AWS, including a CodeCommit repository, CodePipeline, AWS Secret Manager and IAM resources. The CI/CD pipeline retrieves the Databricks PAT token from the registered AWS Secrets Manager to authenticate to the target workspaces. It then deploys the sample Databricks job and pipeline using the Databricks bundle template and Python sample code in this repository.

## Prerequisites

1. All Databricks workspaces (local dev workspace, QA workspace, and Prod workspace) and PAT tokens used in this solution are created outside of the solution template.
2. The AWS user or role has **sufficient permissions** to deploy the CloudFormation template and create the required resources.
3. The sample AWS CloudFormation template creates the AWS Secrets Manager secret pair but does not upload the actual token value. Upload the secret value via the console or API before running the pipeline.
4. The workspaces used in this demo must have access to the public internet to run `pip install` for downloading public resources, such as [nutter](https://github.com/microsoft/nutter).
5. For the introduction to Databricks Asset Bundles and Databricks CLI, please refer to:
    - [Databricks Asset Bundles Documentation](https://docs.databricks.com/en/dev-tools/bundles/index.html)
    - [Databricks CLI Installation Guide](https://docs.databricks.com/en/dev-tools/cli/install.html)

## High-level Architecture Diagram

![High-level Architecture Diagram](./images/high_level_architecture.png)

## High-level Workflow

1. Users can use DABs `databricks bundle` commands to deploy the code in a dev workspace, and use this workspace to debug the notebook code and the DABs YAML template code.
     - ( *Optional* ) Users can also directly associate the dev workspace with the remote repository using the [Databricks Git integration](https://docs.databricks.com/en/repos/index.html) to push code or create pull requests.
2. After debugging, users can push the local DABs folder/code to the AWS CodeCommit remote repository using Git, and then merge the code into the target branch (e.g., "main") via a Pull Request in AWS CodeCommit.
3. Once the PR is merged into the target branch, it triggers the AWS CodePipeline.
4. In the QA build stage of the AWS CodePipeline, the pipeline retrieves the QA workspace PAT token from AWS Secrets Manager for authentication and deploys the resources defined in the DABs template to the QA workspace. In this sample solution, the QA stage runner also runs nutter tests to perform a sample test on one of the notebook functions.
5. If the QA stage completes successfully, it moves to the manual review stage. This stage requires manual approval from a reviewer with the appropriate permissions to promote the code to production.
6. Upon approval, the Prod stage runner retrieves the Prod workspace PAT token and deploys the resources defined in the DABs template to the Prod workspace, such as jobs or DLT pipelines.

##  Deployment Instruction 
