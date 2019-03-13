## Prereq

This section is only completed once and likely already has been done for you.

- We need a few roles for CodePipeline and Systems Manager

  _Windows:_
  ```
  aws cloudformation create-stack ^
    --stack-name iac-prereq-access ^
    --template-body file://prereq-access.yml ^
    --capabilities CAPABILITY_NAMED_IAM
  ```
  _Mac/Linux:_
  ```
  aws cloudformation create-stack \
    --stack-name iac-prereq-access \
    --template-body file://prereq-access.yml \
    --capabilities CAPABILITY_NAMED_IAM
  ```
- We also need a VPC and subnet

  _Windows:_
  ```
  aws cloudformation create-stack ^
    --stack-name iac-prereq-network ^
    --template-body file://prereq-network.yml ^
    --parameters file://prereq-parameters.json
  ```
  _Mac/Linux:_
  ```
  aws cloudformation create-stack \
    --stack-name iac-prereq-network \
    --template-body file://prereq-network.yml \
    --parameters file://prereq-parameters.json
  ```
- Users creating a CodePipeline will need the ability to pass an IAM role. Here is the policy needed:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "IaCPassRole",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "*"
        }
    ]
}
```

## Module 0 - CI/CD pipeline

We're going to create CodeCommit and CodePipeline resources to help us execute
new CloudFormation templates after committing them to a source code repository

__Steps:__
1. Open `0-parameters.json` and provide a value for `YourName` and `RoleArn`.
2. Save this file.
3. Run the following command to set up CodeCommit and CodePipeline:

  _Windows:_
  ```
  aws cloudformation create-stack ^
    --stack-name <your_stack_name> ^
    --template-body file://0-ci-cd.yml ^
    --parameters file://0-parameters.json
  ```
  _Mac/Linux:_
  ```
  aws cloudformation create-stack \
    --stack-name <your_stack_name> \
    --template-body file://0-ci-cd.yml \
    --parameters file://0-parameters.json
  ```
4. The CodeCommit and CodePipeline resources should be created in less than 30
seconds. Run the following command to check the status. Or check the console.
```
aws cloudformation describe-stacks --stack-name <your_stack_name>
```
5. When the pipeline is created you can grab just the repository URL and clone it:
  _Windows:_
  ```
  aws cloudformation describe-stacks ^
    --stack-name <your_stack_name> ^
    --query "Stacks[0].Outputs[0].OutputValue" ^
    --output text
  ```
  ```
  git clone <repo_url>
  ```
  _Mac/Linux:_
  ```
  aws cloudformation describe-stacks \
    --stack-name <your_stack_name> \
    --query 'Stacks[0].Outputs[0].OutputValue' \
    --output text | xargs -I {} \
        git clone {}
  ```
6. Stop here and take a moment to review the resources you have just created.

__Review:__
- [CloudFormation stack](https://us-east-2.console.aws.amazon.com/cloudformation)
- [CodePipeline ...pipeline](https://us-east-2.console.aws.amazon.com/codesuite/codepipeline)

## Module 1 - Deploy infrastructure as code

Review the CloudFormation template, `1-instance.yml`, for this module and try to answer the questions below before getting started.

__Questions:__
- What are we creating?
- Where are the resources being created? How do we control this?
- What will the resources do when they start?
- What does CodePipeline expect our files to be named?
- Will these instructions cause any resources to be destroyed/recreated?

__Steps:__
1. Open `1-parameters.json` and provide a value for `YourName` and `SubnetId`.
2. Save this file.
3. Deploy infrastructure as code through the CI/CD pipeline:

  _Windows:_
  ```
  copy 1-instance.yml <your_repo_name>\template.yml
  copy 1-parameters.json <your_repo_name>\parameters.json
  cd <your_repo_name>
  git add -A
  git commit -m "Creating an instance"
  git push
  cd ..
  ```
  _Mac/Linux:_
  ```
  cp 1-instance.yml <your_repo_name>/template.yml
  cp 1-parameters.json <your_repo_name>/parameters.json
  cd <your_repo_name>
  git add -A
  git commit -m "Creating an instance"
  git push
  cd ..
  ```
4. Monitor the pipeline through CLI or console:

  ```
  aws codepipeline list-pipeline-executions --pipeline-name <your_repo_name>-pipeline
  ```
5. Adjust the size of the instance and create a new commit.
6. Change the user data and create a new commit.

__Review:__
- [CodePipeline](https://us-east-2.console.aws.amazon.com/codesuite/codepipeline)
- [CloudFormation](https://us-east-2.console.aws.amazon.com/cloudformation)
- [VPC and subnet](https://us-east-2.console.aws.amazon.com/vpc)
- [EC2 instance and user data](https://us-east-2.console.aws.amazon.com/ec2)

## Module 2 - Autoscale

__Questions:__
- What resources are being added?
- How will our resources scale? How can we control this?

__Steps:__
1. Deploy autoscale resources through your CI/CD pipeline:

  _Windows:_
  ```
  copy 2-auto-scale.yml <your_repo_name>\template.yml
  cd <your_repo_name>
  git add -A
  git commit -m "Creating instance with auto scale capabilities"
  git push
  cd ..
  ```
  _Mac/Linux:_
  ```
  cp 2-auto-scale.yml <your_repo_name>/template.yml
  cd <your_repo_name>
  git add -A
  git commit -m "Creating instance with auto scale capabilities"
  git push
  cd ..
  ```
2. Adjust the amount of time between scaling events and create a new commit.
  - Set the new value to 120 seconds
3. Verify your change in the console.
4. Adjust the max number of instances and create a new commit.
  - Set the new value to '3'
5. Verify your change in the console.

__Review:__
- [CodePipeline](https://us-east-2.console.aws.amazon.com/codesuite/codepipeline)
- [CloudFormation](https://us-east-2.console.aws.amazon.com/cloudformation)
- [Auto scale group, Scaling policy](https://us-east-2.console.aws.amazon.com/ec2)
- [CloudWatch alarm](https://us-east-2.console.aws.amazon.com/cloudwatch)

## Module 3 - Patch / Refresh

__Questions:__
- How could we apply patches to an instance?

__Steps:__
1. Complete the 'Approve' step in your pipeline.
2. Confirm the 'Delete' step completes.
3. Deploy resources through your CI/CD pipeline:

  _Windows:_
  ```
  copy 3-refresh.yml <your_repo_name>\template.yml
  copy 3-parameters.json <your_repo_name>\parameters.json
  cd <your_repo_name>
  git add -A
  git commit -m "Creating instances that need to be refreshed"
  git push
  cd ..
  ```
  _Mac/Linux:_
  ```
  cp 3-refresh.yml <your_repo_name>/template.yml
  cp 3-parameters.json <your_repo_name>/parameters.json
  cd <your_repo_name>
  git add -A
  git commit -m "Creating instances that need to be refreshed"
  git push
  cd ..
  ```
4. Change the AMI to `ami-0b500ef59d8335eee` and create a new commit.

__Review:__
-

## Module 4 - Disaster Recovery

__Questions:__
-

__Steps:__
1. Change your profile to another region

  ```
  aws configure set region us-west-2
  ```
2. Deploy CloudFormation

  _Windows:_
  ```
  aws cloudformation create-stack ^
    --stack-name <your_stack_name> ^
    --template-body file://3-refresh.yml ^
    --parameters file://parameters.json
  ```
  _Mac/Linux:_
  ```
  aws cloudformation create-stack \
    --stack-name <your_stack_name> \
    --template-body file://3-refresh.yml \
    --parameters file://parameters.json
  ```

__Review:__
- [CloudFormation](https://us-west-2.console.aws.amazon.com/cloudformation)
- [VPC and subnet](https://us-west-2.console.aws.amazon.com/vpc)
- [EC2 instance and user data](https://us-west-2.console.aws.amazon.com/ec2)

## Clean up

1. Delete the contents of the bucket FIRST:

  _Windows:_<br>
  _(Not sure how to write this as a one-liner)_
  ```
  aws cloudformation describe-stack-resources ^
    --stack-name cicd-pipeline ^
    --query "StackResources[?LogicalResourceId==`Artifacts`].PhysicalResourceId" ^
    --output text
  ```
  ```
  aws s3 rm s3://<output_from_line_above> --recursive
  ```
  _Mac/Linux:_
  ```
  aws cloudformation describe-stack-resources \
    --stack-name cicd-pipeline \
    --query 'StackResources[?LogicalResourceId==`Artifacts`].PhysicalResourceId' \
    --output text | xargs -I {} \
        aws s3 rm s3://{} --recursive
  ```

2. THEN delete the stack:
```
aws cloudformation delete-stack --stack-name cicd-pipeline
```