# Cross Account CodePipeline

The AWS CloudFormation template in this repository can be deployed in both a _"Deployer"_ account (as depicted below) and any number of _"Target"_ accounts. It provides a number of cross-account CloudFormation, CodePipeline and CodeBuild IAM roles, which can be assumed from the deployer account; in addition to an Artifact bucket and KMS key which are ready to be configured with CodePipeline.

![Diagram](.github/images/diagram.png "Cross-Account CodePipeline Diagram")

Cross-account CodePipeline artifacts also have their ACL modified by a Lambda function to ensure the deployer account can access them in subsequent actions within the pipeline.

When implementing cross-account pipelines, a macro is provided (as demonstrated in the example template) to enable you to alias accounts with the target IAM role arns; enabling templates to refer to target accounts by alias, as opposed to any hardcoded account IDs. The configuration of this is documented below.

Where possible, roles and policies are locked down to ensure that objects are secured and encrypted.

## Getting Started

Before doing anything, create an `aliases.json` file, which contains a map between logical account names, such as "Dev", "Staging" and "Production", with the associated AWS Account IDs, as described in the example below.

```json
{
  "Development": "999789015432",
  "Staging": "999465729483",
  "Production": "999968243760"
}
```

> These aliases are used within a [CloudFormation Macro](), to help generate IAM Role Arns to the aliases mapped within the file. They also help you to easily identify which account an action is being performed on within the pipeline. Usage of this macro is demonstrated within the provided examples.

Once you've created the aliases you wish to use, the content will be used within the `AccountAliasesJSON` parameter when deploying the CloudFormation template. 

At this point, if you're happy using the provided defaults, you're ready to deploy the template. You will need to repeat the command for every account you wish to enable cross-account deployments on, this includes the deployer account. Try executing the following command:

```shell
aws cloudformation deploy \
  --template-file template.yml \
  --stack-name cross-account-codepipeline \
  --capabilities CAPABILITY_NAMED_IAM
  --parameter-overrides \
      DeployerAccountId="675847109412" \
      TargetAccountIds="999789015432,999465729483,999968243760" \
      AccountAliasesJSON=`cat aliases.json`
```

| Parameter | Description |
|:-----|:-------|
| `DeployerAccountId` **(Required)**             | The AWS Account ID which will contain CodePipelines. |
| `TargetAccountIds` **(Required)**              | A comma-delimited list of AWS Account IDs which will receive deployments. |
| `AccountAliasesJSON` **(Required)**            | A valid JSON strucutre containing target account IDs and their human-readable aliases. |
| `ArtifactsBucketName` (Optional)               | The name (suffix) of the S3 bucket which will contain CodePipeline artifacts. |
| `CloudFormationExecutionRoleName` (Optional)   | The name of the IAM Role used to execute CloudFormation deployments. |
| `CodePipelineExecutionRoleName` (Optional)     | Name of the IAM Role used to execute CodePipeline deployments. |
| `LambdaACLExecutionRoleName` (Optional)        | Name of the Lambbda ACL Execution role name used to update artifact ACLs from target accounts. |