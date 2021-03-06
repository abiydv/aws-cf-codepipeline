# AWS Cross Account Codepipeline
[![CodeFactor](https://www.codefactor.io/repository/github/abiydv/aws-cf-codepipeline/badge)](https://www.codefactor.io/repository/github/abiydv/aws-cf-codepipeline)

![cli](https://github.com/abiydv/ref-docs/blob/master/images/logos/aws-cli_small.png)
![cf](https://github.com/abiydv/ref-docs/blob/master/images/logos/aws-cf_small.png)
![github](https://github.com/abiydv/ref-docs/blob/master/images/logos/github_small.png)
![cp](https://github.com/abiydv/ref-docs/blob/master/images/logos/aws-cp_small.png)
![cb](https://github.com/abiydv/ref-docs/blob/master/images/logos/aws-cb_small.png)
![ecr](https://github.com/abiydv/ref-docs/blob/master/images/logos/aws-ecr_small.png)
![iam](https://github.com/abiydv/ref-docs/blob/master/images/logos/aws-iamrole_small.png)
![sns](https://github.com/abiydv/ref-docs/blob/master/images/logos/aws-sns_small.png)
![s3](https://github.com/abiydv/ref-docs/blob/master/images/logos/aws-s3_small.png)
![kms](https://github.com/abiydv/ref-docs/blob/master/images/logos/aws-kms_small.png)
![cwe](https://github.com/abiydv/ref-docs/blob/master/images/logos/aws-cwevent_small.png)
![ecsf](https://github.com/abiydv/ref-docs/blob/master/images/logos/aws-ecs-fargate_small.png)

Use this to create a cross account AWS Codepipeline which runs in a dedicated account and deploys artifacts in other accounts

- Source - github
- Buid - AWS CodeBuild
- Deploy - AWS Codepipeline ECS deployment

## Architecture

This is a simplified view of the solution detailing out all the components in use.

![cross-account-pipeline](https://github.com/abiydv/ref-docs/blob/master/images/arch/CICD_GH.png)

The various pipeline stages can be visualized as - 

![pipeline-flow](https://github.com/abiydv/ref-docs/blob/master/images/arch/CICD_GH_STAGES.png)

## How to use
To create the ECS clusters in the respective accounts, you can use the [ecs templates](../ecs/). To setup the cross account codepipeline, follow these steps - 

### Step 1
First up, execute the base template `codepipeline-base.yaml` to setup the Codepipeline S3 artifact bucket, ECR repo and the KMS Key in the **tools** account (In the account where the pipeline will reside). Note down the ARN this template outputs, we will need it next.

```
aws cloudformation validate-template --template-body file://codepipeline-base.yaml \
--profile aws-tools-account
```

```
aws cloudformation create-stack --stack-name codepipeline-base-stack \ 
--template-body file://codepipeline-base.yaml \
--profile aws-tools-account --capabilities CAPABILITY_IAM
```

### Step 2
Next, execute the `codepipeline-access.yaml` in **ALL** the accounts you want the codepipeline to deploy to. For example, you might run this in **dev**, **qa** and **prod** accounts where your ECS clusters are. This template needs the KMS Key ARN as input from step 1.

```
aws cloudformation validate-template --template-body file://codepipeline-access.yaml \
--profile aws-dev-account
```

```
aws cloudformation create-stack --stack-name codepipeline-access-stack \
--template-body file://codepipeline-access.yaml \
--parameters ParameterKey=KMSKeyArn,ParameterValue="KMS_KEY_ARN" \
--profile aws-dev-account --capabilities CAPABILITY_NAMED_IAM
```

```
aws cloudformation create-stack --stack-name codepipeline-access-stack \
--template-body file://codepipeline-access.yaml \
--parameters ParameterKey=KMSKeyArn,ParameterValue="KMS_KEY_ARN" \
--profile aws-qa-account --capabilities CAPABILITY_NAMED_IAM
```

```
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
```

```
aws cloudformation create-stack --stack-name codepipeline-stack \
--template-body file://codepipeline-stack.yaml \
--profile aws-tools-account --capabilities CAPABILITY_IAM
```

## Contact

Drop me a note or open an issue if something doesn't work out.

Cheers! :thumbsup:
