---
layout: post
title:  "Deploy AWS FSx with Self Managed Active Directory on CloudFormation using fsx-build-custom-resource"
date:   2019-08-25 08:20:09 +0800
categories: tech
---


## fsx-build-custom-resource

An automation to build the FSx share volume using the [CloudFormation Custom Resource](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html "AWS CloudFormation Custom Resource") and Python Library [custom-resource-helper](https://github.com/aws-cloudformation/custom-resource-helper "custom-resource-helper")


---
### Why using a CloudFormation Custom Resource?

As of now the FSx is still in the early stage and the AWS FSx release the support of using the ["SelfManagedActiveDirectory"](https://aws.amazon.com/about-aws/whats-new/2019/06/amazon-fsx-for-windows-file-server-now-enables-you-to-use-file-systems-directly-with-your-organizations-self-managed-active-directory/), before it only support the use of AWS DFS, the AWS FSx(Windows) CloudFormation does not support(for now, soon it will be supported) the configuration of the "SelfManagedActiveDirectory" and the only way to provision the FSx share volume with self managed active directory support on CloudFormation is by use the CloudFormation Custom Resource

&nbsp;

---
### Components

* fsx-build-function - Components to deploy the required IAM permission, Lambda Function and CloudWatch Logs that will be use by the fsx-build-resource
* fsx-build-resource - Components to deploy the custom resource stack
* encrypt_value.py - To encrypt the AD user password before putting it to the parameter file

---
### How to use?

NOTE: **KMS Key**

You need to create a KMS key before deploying the stack, for more details about creation of KMS key please go to [KMS docs](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html), Incase you already had a CMK you can add the IAM role that will be created by the fsx-build-function stack on the "User" list of your CMK

---
#### Deploying the fsx-build-function


**Download boto3 and crhelper and then package include the fsx**

```bash
$ cd fsx-build-function/fsx-build-lambda-function

$ pip install -r requirements.txt -t .

$ zip -9 -r fsx-build-lambda.zip *

```

**Upload you fsx-build-lambda.zip to your S3 bucket and update the fsx-build-function.parameters**


```bash
[
    {
        "ParameterKey": "S3BucketName",
        "ParameterValue": "<your s3 bucket>"
    },
    {
        "ParameterKey": "S3BucketKey",
        "ParameterValue": "<location/of/your/fsx-build-lambda.zip>"
    },
    {
        "ParameterKey": "AppPrefix",
        "ParameterValue": "<name that will be use as a prefix for the created resources>"
    },
    {
        "ParameterKey": "Handler",
        "ParameterValue": "fsx-build.handler"
    }
]
```

```bash
$ aws --region us-east-1 cloudformation create-stack --stack-name <stack name> --template-body file://template/fsx-build-function.yaml --parameters file://fsx-build-function.parameters --capabilities CAPABILITY_NAMED_IAM

```

---
#### Deploying the fsx-build-resource


---
**Encrypt the Service User Password**

NOTE: Your user role should be included in CMK admin or user role with Encrypt permission

```bash
$ ./encrypt_value.py -h
usage: encrypt_value.py [-h] [-v VAL] [-r REGION] [-k KEY]

encrypt a string using the KMS key

optional arguments:
  -h, --help            show this help message and exit
  -v VAL, --val VAL     A string value that will be encrypted
  -r REGION, --region REGION
                        The AWS region where the KMS key located
  -k KEY, --key KEY     The KMS key ID that will use to encrypt the string
                        value
$ ./encrypt_value.py -v <Password> -k <your KMS_key_id> -r <region>
AQICAXXXxxxxXXXxxxXXxxxXX2O+DxJ3DlfspJTsXXxxXXxxXXxxxXXxxXXxzN266sqHlx//cmNUahAAAAAaDBmBgkqhkiG9w0BBwagWTBXAgEAMFIGCSqGSIb3DQEHATAeBglghkgBSASDSXXXXxxxYa8/lWWwJGAgEQceV1wwy0XXXsSSSSSSSrBha+jZpjn5X3/XxxxXXXXXz/123eas
```
---
**Input needed information on the parameter(fsx-build-resource.parameters)**

```bash
$ cd fsx-build-resource
```

```bash
[
    {
        "ParameterKey": "FSxBuildFunctionStackName",
        "ParameterValue": "<stack name of the deployed fsx-build-function>"
    },
    {
        "ParameterKey": "FSxShareName",
        "ParameterValue": "<fsx-share-volume-name>"
    },
    {
        "ParameterKey": "StorageCapacity",
        "ParameterValue": "300"
    },
    {
        "ParameterKey": "Subnet",
        "ParameterValue": "<your subnet id>"
    },
    {
        "ParameterKey": "SG1",
        "ParameterValue": "<security group1>"
    },
    {
        "ParameterKey": "SG2",
        "ParameterValue": "<security group2>"
    },
    {
        "ParameterKey": "SG3",
        "ParameterValue": "<security group3>"
    },
    {
        "ParameterKey": "AppPrefix",
        "ParameterValue": "<name that will be use as prefix to your fsx share"
    },
    {
        "ParameterKey": "Team",
        "ParameterValue": "<your team name for tagging purposes>"
    },
    {
        "ParameterKey": "Service",
        "ParameterValue": "<service name for tagging purposes>"
    },
    {
        "ParameterKey": "KmsKey",
        "ParameterValue": "<kms key id>"
    },
    {
        "ParameterKey": "Domain",
        "ParameterValue": "<fully qualified domain name of the self-managed AD>"
    },
    {
        "ParameterKey": "OU",
        "ParameterValue": "<fully qualified distinguished name of the organizational unit within your self-managed AD>"
    },
    {
        "ParameterKey": "AdminGroup",
        "ParameterValue": "<name of the domain group whose members are granted administrative privileges>"
    },
    {
        "ParameterKey": "UserName",
        "ParameterValue": "<user name for the service account on your self-managed AD>"
    },
    {
        "ParameterKey": "Password",
        "ParameterValue": "<password for the service account on your self-managed AD that has been encrypted by the encrypt_value.py script>"
    },
    {
        "ParameterKey": "DNSip1",
        "ParameterValue": "<DNS server 1>"
    },
    {
        "ParameterKey": "DNSip2",
        "ParameterValue": "<DNS server 2>"
    },
    {
        "ParameterKey": "Throughput",
        "ParameterValue": "8"
    }
]
```

**Deploy the FSx resource (fsx-build-resource)**

```bash
$ aws --region us-east-1 cloudformation create-stack --stack-name <your fsx resource stack name> --template-body file://template/fsx-build-resource.yaml --parameters file://fsx-build-resource.parameters
```


**Github Repo**: [fsx-build-custom-resource](https://github.com/chojayr/aws-backup-cf)
