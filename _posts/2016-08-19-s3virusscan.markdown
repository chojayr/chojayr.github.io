---
layout: post
title:  "S3 Virus Scanning"
date:   2016-08-19 00:35:09 +0800
categories: tech
---

A simple solution to implement an additional security on your data on AWS S3. this is to make an antivirus scanning on every new data/object added  on your S3 bucket. 

### Feature
* Uses ClamAV to scan "newly" added files on S3 buckets
* Updates ClamAV database every 3 hours automatically
* Publishes a message to SNS in case of a finding
* Can optionally delete compromised files automatically

### How it works
* Using the S3 event notification. the S3 bucket will send events to the SQS queue in case of newly created file(object)
* A ruby script running as a daemon in the background on the EC2 instance will act as a worker that will pull the queue messages from SQS which the contents are the events from the S3 buckets of newly created objects.
* The script will download the newly created object from the S3 bucket then perform clamscan on the downloaded object to verify if there's a threat on the file
* Once a threat found on a scanned file it will delete the file and send an SNS notification to the subscribers
* The script also perform an antivirus update every 3 hours


![s3virusscanning](/img/s3virusscan.png){:class="img-responsive"}



### Automation

**CloudFormation template**

The whole stack to implement the AWS S3 virus scan including the creation of SQS queue, SNS topic and the provisioning of the EC2 instance will be build automatically thru CloudFormation template, here's the sample CF template. for the s3-virusscan-stack. the repo contains 2 JSON file, the template and the params, You only need to modify the content of the params replace it with the value of your own.

<sub>***Repository:***</sub> <sub>[s3-virusscan-stack][virusscan]</sub> 

&nbsp;

**Salt State**

The Salt state will be the one responsible for the setting-up of the scripts and the installation of the script dependencies like Ruby, Gems, script configuration files and the installation of the ClamAV. checkout the salt state here


<sub>***Repository:***</sub> <sub>[salt-s3clamavscan][salt-virusscan]</sub> 

&nbsp;

**Deploying the Stack**

To deploy the CloudFormation template, You need to modify the "s3_virusscan_params.json" based on the environment you will going to deploy the stack. 

{% highlight bash %}
$ aws cloudformation create-stack --stack-name <your_stack_name> --template-body file://s3_virusscan_template.json --parameters file://s3_virusscan_params.json
{% endhighlight %}

&nbsp;

**Add the S3 Bucket**

Using the S3 event notification feature we can now send message to the SQS when a new object is added to the bucket or an existing object is overwritte

![S3EventNotification](/img/s3conf.png){:class="img-responsive"}

For more details about S3 event notification kindly check it [here][s3noti]



[virusscan]: https://github.com/chojayr/s3-virusscan-stack
[salt-virusscan]: https://github.com/chojayr/salt-s3clamavscan
[s3noti]: https://aws.amazon.com/blogs/aws/s3-event-notification/

