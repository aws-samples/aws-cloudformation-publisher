## AWS Cloudformation Publisher

AWS CloudFormation Publisher packages your CloudFormation templates into an S3 bucket in every AWS region and creates "launch stack" links that you can use in your documentation so that your customers can easily launch stacks in their AWS accounts from your CloudFormation template.

[![Build Status](https://travis-ci.org/aws-samples/aws-cloudformation-publisher.svg?branch=master)](https://travis-ci.org/aws-samples/aws-cloudformation-publisher)

## Overview

AWS CloudFormation Publisher packages your CloudFormation templates into an S3 bucket in every AWS region through a CodeBuild project which, when you start a build, can be overridden to point at your project's source location.

AWS CloudFormation Publisher creates S3 buckets for each region in your account. The bucket names have the following format:

`${bucket prefix}-{region name}`

For example: `cfn-0123456789012-us-east-1`

Your projects' CloudFormation templates will be stored as `{project name}/${version}/main.template`.

## Installation

Deploy the supplied CloudFormation template into your account; this will create a CodeBuild project named `cfn-publish`.

```
aws cloudformation create-stack --stack-name cfn-publish --template-body file://./cfn.template --capabilities CAPABILITY_IAM
```

## Configuration

AWS CloudFormation Publisher looks for a file in the root of your repository called `cfn-publish.config` which can be used to override:
* The location of the CloudFormation template in the repository that you wish to publish (the default is `cfn.template`)
* The regions to publish to (the default is all regions)
* The prefix to use in bucket names (the default is `cfn-<aws_account_id>`)
* The ACL to use for uploaded artefacts (the default is `private` i.e. only IAM principals in your account who have explicitly been granted access to the bucket can launch the template) 
* The project name to prefix all artefact object keys with in S3 (if using a git source, the default is the repo name. if using an S3 object source, the default is file name, without it's extension)

Your `cfn-publish.config`, if you need one, should look something like this:
```
template="main.template"
regions="eu-west-1 eu-west-2 eu-central-1"
bucket_name_prefix="my-custom-prefix"
acl="private"
project="my-project"
extra_files="backend.zip frontend.zip"
```

## Usage

To publish your repository's CloudFormation template, take the following steps:

* Open the [CodeBuild console](https://console.aws.amazon.com/codesuite/codebuild/projects/)
* Select the `cfn-publish` project
* Press "Start Build"
* Press "Advanced build overrides"
* Configure the "Source" section
* Press "Start build"

You will see the bucket and template names in the build output.

## Notifications

An SNS topic is created as part of the solution which you can subscribe to in order to receive
notifications regarding cfn-publish execution statuses. Notifications will be in the format: 

```
Publication of '<SOURCE_LOCATION>' has reached the status '<SUCCEEDED|FAILED|IN_PROGRESS|STOPPED|TIMED_OUT|STOPPED>'
```

## Versioning

cfn-publish will look for the environment variable VERSION to determine which version to use in the object key. If the VERSION is set, `main.template` from the **most recent** execution will be available at both `{project name}/{version}/main.template` and `{project name}/latest/main.template`. If the VERSION is not set, then it will only be available at `{project name}/latest/main.template`.

Before publishing a version, cfn-publish will first wipe any existing files it finds for that version from the S3 buckets.

## Contributing

Contributions are more than welcome. Please read the [code of conduct](CODE_OF_CONDUCT.md) and the [contributing guidelines](CONTRIBUTING.md).

## License Summary

This sample code is made available under the MIT-0 license. See the LICENSE file.
