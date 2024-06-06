# DABs CI/CD Pipeline Example on AWS

This example demonstrates how to set up a CI/CD pipeline for deploying Databricks resources such as jobs and pipelines using **[Databricks Asset Bundles (DABs)](https://docs.databricks.com/en/dev-tools/bundles/index.html)** on the AWS platform.

## Solution Introduction

This solution provides an AWS CloudFormation template to provision the CI/CD pipeline resources on AWS, including a CodeCommit repository, CodePipeline, AWS Secrets Manager, and IAM resources. The CI/CD pipeline retrieves the Databricks PAT token from the registered AWS Secrets Manager to authenticate to the target workspaces. It then deploys the sample Databricks job and pipeline using the Databricks bundle template and Python sample code in this repository.

The Python notebooks and pipelines used in this sample solution are **basic examples** that can be directly obtained using the databricks bundle init command. This solution aims to demonstrate how to deploy and integrate Databricks Asset Bundles within an AWS CI/CD pipeline. For more complex ML solutions, please refer to other Databricks ML references, such as the Databricks MLOps stack.

## Prerequisites

1. All Databricks workspaces (local dev workspace, QA workspace, and Prod workspace) and PAT tokens used in this solution are created outside of the solution template.
2. The AWS user or role has **sufficient permissions** to deploy the CloudFormation template and create the required resources.
3. The sample AWS CloudFormation template creates the AWS Secrets Manager secret pair but does not upload the actual token value. Upload the secret value via the console or API before running the pipeline.
4. The workspaces used in this demo must have access to the public internet to run `pip install` for downloading public resources, such as [nutter](https://github.com/microsoft/nutter).
5. For the introduction to Databricks Asset Bundles and Databricks CLI, please refer to:
    - [Databricks Asset Bundles Documentation](https://docs.databricks.com/en/dev-tools/bundles/index.html)
    - [Databricks CLI Installation Guide](https://docs.databricks.com/en/dev-tools/cli/install.html)


## Workspace Authentication
Databricks [personal access tokens (PATs)](https://docs.databricks.com/en/dev-tools/auth/pat.html) are one of the most well-supported types of credentials for resources and operations at the Databricks workspace level. In this solution, we use PATs to allow CI/CD pipeline execution jobs to authenticate to the target workspaces. This sample solution does NOT load the PATs from the CloudFormation [template parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html). Instead, please use the AWS console, AWS API, or SDK to upload the PAT token value to the AWS Secret Manager.

We also recommend creating the Databricks PATs with an expiration date. Using access tokens set to never expire is highly not recommended, because this significantly increases the risk of token misuse or compromise.

## High-level Architecture Diagram

![High-level Architecture Diagram](./images/high_level_architecture.png)

## High-level Workflow

1. Users can use DABs `databricks bundle` commands to deploy the code in a dev workspace, and use this workspace to develop the notebook code and the DABs YAML template code.
     - ( *Optional* ) Users can also directly associate the dev workspace with the remote repository using the [Databricks Git integration](https://docs.databricks.com/en/repos/index.html) to push code or create pull requests.
2. After development and debugging, users can push the local DABs folder/code to the AWS CodeCommit remote repository using Git, and then merge the code into the target branch (e.g., "main") via a Pull Request in AWS CodeCommit.
3. Once the PR is merged into the target branch, it triggers the AWS CodePipeline.
4. In the QA build stage of the AWS CodePipeline, the pipeline retrieves the QA workspace PAT token from AWS Secrets Manager for authentication and deploys the resources defined in the DABs template to the QA workspace. In this sample solution, the QA stage runner also runs nutter tests to perform a sample test on one of the notebook functions.
5. If the QA stage completes successfully, it moves to the manual review stage. This stage requires manual approval from a reviewer with the appropriate permissions to promote the code to production.
6. Upon approval, the Prod stage runner retrieves the Prod workspace PAT token and deploys the resources defined in the DABs template to the Prod workspace, such as jobs or DLT pipelines.

## Deployment Instruction

1. **Create CloudFormation Stack**:
    - Use the sample `codepipeline-stack.template` provided in this repository to create a stack in CloudFormation in your chosen AWS region. For example, if all your workspaces are in the us-east-1 region, it is recommended to create the CloudFormation stack in this region as well.
    - Review and fill in the CloudFormation stack parameters.
    - ![CloudFormation Stack](./images/create_stack.png)
    - On the next page, use the default CloudFormation options, confirm the creation of IAM roles, and create the stack.
    - ![CloudFormation Stack Diagram](./images/code_commit_repo.png)

2. **Upload Databricks PAT Tokens to AWS Secrets Manager**:
    - After the CloudFormation stack is successfully created, you will find the newly created repository in the CodeCommit service page and a "DABS_DEMO" secret in the AWS Secrets Manager.
    - Use an AWS user or role with sufficient permissions to store the Databricks PAT tokens from your Dev and QA workspaces in the corresponding secret values. [Databricks PAT Token Documentation](https://docs.databricks.com/en/dev-tools/auth/pat.html)
    - ![Secrets Manager Step 1](./images/secret_1.png)
    - ![Secrets Manager Step 2](./images/sceret_2.png)

3. **Connect to the CodeCommit Repository**:
    - Connect to the CodeCommit repository created by the CloudFormation stack. You can directly commit and push to the branch that triggers the pipeline, or you can commit to a development branch first and then create a Pull Request to merge your changes into the trigger branch (in this example, we use the "main" branch).
    - Note: Before pushing the code to the remote repository, ensure that the `databricks.yml` and the Python notebooks are correctly configured.
    - ![CodeCommit Main Branch](./images/codecommit_main_branch.png)

4. **Trigger the CI/CD Pipeline**:
    - After pushing the code to the trigger branch, the CodePipeline run will be automatically triggered. Click "View details" on each stage to review the runner execution log.
    - [Runner Output Example](./images/runner_log.png)

5. **Manual Approval**:
    - Once the staging phase is completed, click "Approve" in the manual review stage to promote the pipeline to the production stage.
    - [Review and Promote](./images/review.png)

6. **Production Deployment**:
    - After the final "Production" stage is completed, you can view the runner execution output and logs by clicking "View details."
    - ![Prod runner log](./images/prod_runner_log.png)

7. **Review Databricks Resources**:
    - You can review the deployed resources by logging into the QA and Prod workspaces in Databricks.
    - ![Review resources in workspace](./images/workspace_jobs_details.png)
    - ![Prod workspace output](./images/prod_workspace_output.png)

## Clean Up
1. Delete the resources created in the Databricks workspace, either through the console or using the Databricks CLI.
2. Delete the CloudFormation stack and its resources from your AWS account.

## Reference

1. [What are Databricks Asset Bundles?](https://docs.databricks.com/en/dev-tools/bundles/index.html)
2. [Databricks Asset Bundles for MLOps Stacks](https://docs.databricks.com/en/dev-tools/bundles/mlops-stacks.html)
3. [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
4. [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
5. [AWS CodePipeline with AWS CodeBuild to test code and run builds](https://docs.aws.amazon.com/codebuild/latest/userguide/how-to-create-pipeline.html)
