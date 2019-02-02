# AWS Cross Account Codepipeline
Use this to create a cross account AWS Codepipeline which runs in a dedicated account and deploys artifacts in other accounts

- Source - github
- Buid - AWS CodeBuild
- Deploy - AWS Codepipeline ECS deployment

![cross-account-pipeline](./images/cross-account-code-pipeline.png)

## How to use
To create the ECS clusters in the respective accounts, you can use the [ecs templates](../ecs/). To setup the cross account codepipeline, follow these steps - 

### Step 1
First up, execute the base template `codepipeline-base.yaml` to setup the Codepipeline S3 artifact bucket, ECR repo and the KMS Key in the **tools** account (In the account where the pipeline will reside). Note down the ARN this template outputs, we will need it next.

```
aws cloudformation validate-template --template-body file://codepipeline-base.yaml \
--profile aws-tools-account

aws cloudformation create-stack --stack-name codepipeline-base-stack \ 
--template-body file://codepipeline-base.yaml \
--profile aws-tools-account --capabilities CAPABILITY_IAM
```

### Step 2
Next, execute the `codepipeline-access.yaml` in **ALL** the accounts you want the codepipeline to deploy to. For example, you might run this in **dev**, **qa** and **prod** accounts where your ECS clusters are. This template needs the KMS Key ARN as input from step 1.

```
aws cloudformation validate-template --template-body file://codepipeline-access.yaml \
--profile aws-dev-account

aws cloudformation create-stack --stack-name codepipeline-access-stack \
--template-body file://codepipeline-access.yaml \
--parameters ParameterKey=KMSKeyArn,ParameterValue="KMS_KEY_ARN" \
--profile aws-dev-account --capabilities CAPABILITY_NAMED_IAM

aws cloudformation create-stack --stack-name codepipeline-access-stack \
--template-body file://codepipeline-access.yaml \
--parameters ParameterKey=KMSKeyArn,ParameterValue="KMS_KEY_ARN" \
--profile aws-qa-account --capabilities CAPABILITY_NAMED_IAM

aws cloudformation create-stack --stack-name codepipeline-access-stack \
--template-body file://codepipeline-access.yaml \
--parameters ParameterKey=KMSKeyArn,ParameterValue="KMS_KEY_ARN" \
--profile aws-prod-account --capabilities CAPABILITY_NAMED_IAM
```

### Step 3
Finally, execute the `codepipeline-stack.yaml` in the **tools** AWS account to setup the codepipeline, codebuild project, IAM roles (for codepipeline, codebuild), SNS Topics (for approve/notify emails), cloudwatch event rules (for notifying on pipeline and buildproject state change).

```
aws cloudformation validate-template --template-body file://codepipeline-stack.yaml \
--profile aws-tools-account

aws cloudformation create-stack --stack-name codepipeline-stack \
--template-body file://codepipeline-stack.yaml \
--profile aws-tools-account --capabilities CAPABILITY_IAM
```

## Contact

Drop me a note or open an issue if something doesn't work out.

Cheers! :thumbsup:
