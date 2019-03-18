## Prereq

This section is only completed once and likely already has been done for you.

- We need a few roles for CodePipeline

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
    --stack-name <your_cicd_stack_name> ^
    --template-body file://0-ci-cd.yml ^
    --parameters file://0-parameters.json
  ```
  _Mac/Linux:_
  ```
  aws cloudformation create-stack \
    --stack-name <your_cicd_stack_name> \
    --template-body file://0-ci-cd.yml \
    --parameters file://0-parameters.json
  ```
4. The CodeCommit and CodePipeline resources should be created in less than 30
seconds. Run the following command to check the status. Or check the console.
```
aws cloudformation describe-stacks --stack-name <your_cicd_stack_name>
```
5. When the pipeline is created you can grab just the repository URL and clone it:
  _Windows:_
  ```
  aws cloudformation describe-stacks ^
    --stack-name <your_cicd_stack_name> ^
    --query "Stacks[0].Outputs[0].OutputValue" ^
    --output text
  ```
  ```
  git clone <repo_url>
  ```
  _Mac/Linux:_
  ```
  aws cloudformation describe-stacks \
    --stack-name <your_cicd_stack_name> \
    --query 'Stacks[0].Outputs[0].OutputValue' \
    --output text | xargs -I {} \
        git clone {}
  ```
6. Stop here and take a moment to review the resources you have just created.

__Review:__
- [CloudFormation stack](https://us-east-1.console.aws.amazon.com/cloudformation)
- [CodeCommit repository](https://console.aws.amazon.com/codesuite/codecommit/repositories?region=us-east-1)
- [CodePipeline ...pipeline](https://us-east-1.console.aws.amazon.com/codesuite/codepipeline)

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
4. Monitor the [pipeline](https://us-east-1.console.aws.amazon.com/codesuite/codepipeline) through the console.
5. Once the 'Deploy' stage is complete review the following:
  - Your Code Pipeline has created a [CloudFormation](https://us-east-1.console.aws.amazon.com/cloudformation) stack with your name
  - [EC2 instance](https://us-east-1.console.aws.amazon.com/ec2) is created (Console > EC2 > Instances)
  - CPU has spiked to ~100% (Select instance > Monitoring tab > CPU Utilization)
5. Adjust the size of the instance and create a new commit.
  - Set the size to `t2.small`
6. Verify your change in the console.
7. Change the user data to run less frequently and create a new commit.
  - Set the cron to `*/2 * * * *`
8. Verify your change in the console.

## Module 2 - Autoscale

Review the CloudFormation template, `2-auto-scale.yml`, for this module and try to answer the questions below before getting started.

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
  - Set the new value to `120` seconds
3. Verify your change in the console.
4. Adjust the max number of instances and create a new commit.
  - Set the new value to `3`
5. Verify your change in the console.

__Review:__
- [CodePipeline](https://us-east-1.console.aws.amazon.com/codesuite/codepipeline)
- [CloudFormation](https://us-east-1.console.aws.amazon.com/cloudformation)
- [Auto scale group, Scaling policy](https://us-east-1.console.aws.amazon.com/ec2)
- [CloudWatch alarm](https://us-east-1.console.aws.amazon.com/cloudwatch)

## Module 3 - Patch / Refresh

Review the CloudFormation template, `3-refresh.yml`, for this module and try to answer the questions below before getting started.

__Questions:__
- How could we apply patches to an instance?

__Steps:__
1. Complete the 'Approve' step in your pipeline.
2. Confirm the 'Delete' step completes.
3. Open `3-parameters.json` and provide a value for `YourName`, `SubnetId`, and `OS` (use "AmazonLinux2" as the OS value)
4. Save this file.
5. Deploy resources through your CI/CD pipeline:

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
6. Change the `OS` value to `RedHat7` in your parameters file and create a new commit.
7. Watch what happens to your instances in the console.

__Review:__
- [CodePipeline](https://us-east-1.console.aws.amazon.com/codesuite/codepipeline)
- [EC2 Instances](https://us-east-1.console.aws.amazon.com/ec2)
- [EC2 Auto Scaling](https://us-east-1.console.aws.amazon.com/ec2/autoscaling)

## Module 4 - Disaster Recovery

Review the CloudFormation template, `4-dr.yml`, for this module and try to answer the questions below before getting started.

__Questions:__
- What region and availability zone will the resources be created in?
- Where is the AMI magic happening? How would you expand this?

__Steps:__
1. Change your profile to another region

  ```
  aws configure set region us-west-2
  ```
2. Deploy CloudFormation

  _Windows:_
  ```
  aws cloudformation create-stack ^
    --stack-name <your_dr_stack_name> ^
    --template-body file://4-dr.yml ^
    --parameters file://4-parameters.json
  ```
  _Mac/Linux:_
  ```
  aws cloudformation create-stack \
    --stack-name <your_dr_stack_name> \
    --template-body file://4-dr.yml \
    --parameters file://4-parameters.json
  ```
3. Add another availability zone and create a new commit.

__Review:__
- [CloudFormation](https://us-west-2.console.aws.amazon.com/cloudformation)
- [EC2 instance and user data](https://us-west-2.console.aws.amazon.com/ec2)

## Clean up

1. Go to [CodePipeline](https://us-east-1.console.aws.amazon.com/codesuite/codepipeline) and complete the 'Approve' stage. Confirm the 'Delete' stage completes.
2. Tear down the DR stack:

  ```
  aws cloudformation delete-stack --stack-name <your_dr_stack_name>
  ```
3. Change your profile back to the original region

  ```
  aws configure set region us-east-1
  ```
4. Delete the contents of the CI/CD artifact bucket:

  _Windows:_<br>
  _(Or use the console because I'm not sure how to write this as a one-liner)_
  ```
  aws cloudformation describe-stack-resources ^
    --stack-name <your_cicd_stack_name> ^
    --query "StackResources[?LogicalResourceId==`Artifacts`].PhysicalResourceId" ^
    --output text
  ```
  ```
  aws s3 rm s3://<output_from_line_above> --recursive
  ```
  _Mac/Linux:_
  ```
  aws cloudformation describe-stack-resources \
    --stack-name <your_cicd_stack_name> \
    --query 'StackResources[?LogicalResourceId==`Artifacts`].PhysicalResourceId' \
    --output text | xargs -I {} \
        aws s3 rm s3://{} --recursive
  ```

5. Tear down the CI/CD stack:
```
aws cloudformation delete-stack --stack-name <your_cicd_stack_name>
```
6. (Optional) If you created the prereq resources, tear those down:
```
aws cloudformation delete-stack --stack-name iac-prereq-network
aws cloudformation delete-stack --stack-name iac-prereq-access
```

## Miscellaneous

Required software:
- AWS CLI
- jq library

List Amazon Linux 2 AMIs in a region:
```
aws ec2 describe-images \
  --filters "Name=owner-alias,Values=amazon" \
    "Name=name,Values=amzn2-ami-hvm-*-gp2" \
    "Name=architecture,Values=x86_64" \
  --owner "amazon" \
  --region us-east-1 | \
  jq '.Images | sort_by(.CreationDate) | .[] | {Name: .Name, Created: .CreationDate, Id: .ImageId}' \
  > amazonlinux-east-2
```

List Red Hat AMIs in a region:
```
aws ec2 describe-images \
  --filters "Name=name,Values=RHEL*" \
    "Name=architecture,Values=x86_64" \
  --region us-east-1 | \
  jq '.Images | sort_by(.CreationDate) | .[] | {Name: .Name, Created: .CreationDate, Id: .ImageId}' \
  > rhel-east-2
```
