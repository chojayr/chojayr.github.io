---
layout: post
title:  "AWS Backup CloudFormation template sample"
date:   2019-07-07 05:05:09 +0800
categories: tech
---


I was recently working on the creation of the AWS Backup(to backup our EFS file-system) using the CloudFormation, good thing AWS just recently announced that ![AWS Backup now supports CloudFormation](https://aws.amazon.com/about-aws/whats-new/2019/05/aws-backup-now-supports-aws-cloudformation/), but the docs seems lack in sample snippets for the developers to use as a reference so I decided to post my templates as a sample 



&nbsp;

### CF Template

**aws-backup.yaml**

```YAML
---
AWSTemplateFormatVersion: '2010-09-09'
Description: Create AWS Backup Vault, Backup Plan and Backup Selection
Parameters:
# Backup
  CreateNewBackupVault:
    Type: String
    AllowedValues:
      - true
      - false
  BackupVaultName:
    Type: String
  BackupPlanName:
    Type: String
    Description: AWS Backup name for BackupPlan
  BackupSelectionName:
    Type: String
    Description: AWS Backup name for BackupSelection
# Backup Rules
  BackupPolicy:
    Type: String
    Description: AWS Backup frequency choose between backup once, twice, thrice or four times a day
    AllowedValues:
      - BackupOnceDaily
      - BackupTwiceDaily 
      - BackupThriceDaily
      - BackupFourTimesDaily
  BackupDefaultRole:
    Type: String
    Description: IAM service role for the BackupSelection
  DeleteAfterDays:
    Type: Number
    Description: Number of days before the backup will be deleted
# Tags
  Team:
    Type: String
    Description: Team name that own's the backup, this will be use for tag 
  Email:
    Type: String
    Description: Team email address

Conditions:
  CreateNewVault: !Equals [ !Ref CreateNewBackupVault, "true" ]
  OnceDaily: !Equals [ !Ref BackupPolicy, "BackupOnceDaily" ]
  TwiceDaily: !Equals [ !Ref BackupPolicy, "BackupTwiceDaily" ]
  ThriceDaily: !Equals [ !Ref BackupPolicy, "BackupThriceDaily" ]
  FourTimesDaily: !Equals [ !Ref BackupPolicy, "BackupFourTimesDaily" ]
  
Resources:
  StorageBackupVault:
    Type: AWS::Backup::BackupVault
    Condition: CreateNewVault
    Properties:
      BackupVaultName: !Ref BackupVaultName
      BackupVaultTags: {
        "Team": !Ref Team,
        "Email": !Ref Email
      }

  StorageBackupPlan:
    Type: AWS::Backup::BackupPlan
    Properties:
      BackupPlan: 
        BackupPlanName: !Ref BackupPlanName
        BackupPlanRule:
          - 
            RuleName: !Ref BackupPolicy
            TargetBackupVault: !If [ CreateNewVault, !Ref StorageBackupVault, !Ref BackupVaultName ]
            ScheduleExpression: 
              !If
                [ OnceDaily, "cron(0 1 * * ? *)",
                !If
                  [ TwiceDaily, "cron(0 0/12 * * ? *)",
                  !If 
                    [ ThriceDaily, "cron(0 0/8 * * ? *)", "cron(0 0/6 * * ? *)" ]
                  ]
                 ]
            Lifecycle: {
                DeleteAfterDays: !Ref DeleteAfterDays
            }
            RecoveryPointTags: {
              "Team": !Ref Team,
              "Email": !Ref Email
            }
      BackupPlanTags: {
        "Team": !Ref Team,
        "Email": !Ref Email
      }

  StorageBackupSelectionByTags:
    Type: AWS::Backup::BackupSelection
    DependsOn: StorageBackupPlan
    Properties:
      BackupSelection: 
        SelectionName: !Ref BackupSelectionName
        IamRoleArn: !Ref BackupDefaultRole
        ListOfTags:
          - 
            ConditionType: "STRINGEQUALS"
            ConditionKey: "Backup"
            ConditionValue: !Ref BackupSelectionName
      BackupPlanId: !Ref StorageBackupPlan
 

Outputs:
  BackupSelectionName:
    Description: Tag:Value you need to put on your resource along with the Tag:Key Backup
    Value: !Ref BackupSelectionName
  BackupSelectionId:
    Description: Backup Selection ID
    Value: !Ref StorageBackupSelectionByTags
  BackupVaultArn:
    Description: Backup Vault ARN
    Condition: CreateNewVault
    Value: !GetAtt StorageBackupVault.BackupVaultArn
  BackupPlanArn:
    Description: BackupPlan Arn
    Value: !GetAtt StorageBackupPlan.BackupPlanArn
  BackupPlanId:
    Description: BackupPlan ID
    Value: !Ref StorageBackupPlan
  BackupPlanVersionId:
    Description: BackupPlan Version ID
    Value: !GetAtt StorageBackupPlan.VersionId
```

&nbsp;


### Sample parameter file

**sample-params.json**

```javascript
[
    {
        "ParameterKey": "CreateNewBackupVault",
        "ParameterValue": "true"
    },
    {
        "ParameterKey": "BackupVaultName",
        "ParameterValue": "mybackup-vault-sample"
    },
    {
        "ParameterKey": "BackupPlanName",
        "ParameterValue": "daily-backup-sample"
    },
    {
        "ParameterKey": "BackupSelectionName",
        "ParameterValue": "daily-backup"
    },
    {
        "ParameterKey": "Team",
        "ParameterValue": "AppTeamSample"
    },
    {
        "ParameterKey": "Email",
        "ParameterValue": "team-email@yourdomain.com"
    },
    {
        "ParameterKey": "BackupPolicy",
        "ParameterValue": "BackupOnceDaily"
    },
    {
        "ParameterKey": "BackupDefaultRole",
        "ParameterValue": "arn:aws:iam::1122333344:role/service-role/AWSBackupDefaultServiceRole"
    },
    {
        "ParameterKey": "DeleteAfterDays",
        "ParameterValue": 30
    }
]
```

&nbsp;

### Test the template

```bash
$ aws cloudformation create-stack --stack-name <stack name> --template-body file://aws-backup.yaml --parameters=file://sample-params.json 
```

NOTE:
* This CF template will create the AWS Backup components BackupVault, BackupPlan and BackupSelection 

* This template include the creation of new BackupVault if desired just set the "CreateNewBackupVault" to "true" or use the existing BackupVault e.g. "Default"

* It also contain a condition for the Backup policy backup frequeny (BackupOnceDaily, BackupTwiceDaily, BackupThriceDaily and BackupFourTimesDaily) you can change it the schedule you want just follow the ![Schedule Expression for Rules](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html)

*  Please check here for the supported AWS service https://aws.amazon.com/backup/features


&nbsp;

Github Repo: ![aws-backup-cf](https://github.com/chojayr/aws-backup-cf)
