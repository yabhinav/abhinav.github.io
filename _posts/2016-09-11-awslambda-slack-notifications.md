---

title: Monitor Ec2 Instances with Slack and Ambari Lambda
excerpt: "Monitor Number Running EC2 Instance or EC2 Instance evnets using a Lambda Function and send notifications to slack via incoming webhook integrated to the channel"
tags: [AWS, Slack, lambda, EC2]
categories: [ cloud ]
comments: true
published: true
share: true
---

{% include toc title="Slack it!!" icon="file-text" %}

# Overview

Slack is a wonderful tool to communicate with the Team. Amazon Webservices makes things simple for a hadoop developer, spawn a couple instances, install your hadoop and test your applications.

I have already worked on automating the launch of a full fledged HDP cluster or an Sandbox on AWS. The only loopside is these instances are costly and If we forget to stop the instances when not in use it can rack up a $100 bill over the weekend.

**AWS Cloudwatch** is a good place to start, It can monitor the instances you spawn and using SNS you can certainly trigger a message to mobile or send a mail, but it is so last decade technology. Nobody notices them and also they don't carry the rich features of an Messenger.

Instant Messengers are the latest trend and they have penetrated to office network with **Slack** and **HipChat**.
Slack has wonderful plugins like  *github* and *jira* Integration. It also has **Incoming Webhooks App** to which we can post a json request that will be translated to Slack Message.

## Cloudwatch-SNS-Slack

For Elastic instances with cloudwatch triggers enabled, an SNS Topic will be triggered and Subscribers will be notified of an event, which can be sent as a message to Slack channel using AWS Lamba Function. We have to read the event data from SNS and then translate it into a json request , post it to the slack-webhook. For this we will configure our lambda function as a subscriber to the SNS Topic.

## Cloudwatch-Slack

The only drawback with SNS Triggered Slack messages is it will consume both SNS and Lambda free quota per month. So Instead we can schedule a Cloudwatch trigger on lamba function which uses *aws-sdk* to query instance state for a given region like *ec2_describe*. It roughly takes 10 ms to query the status of 20 instances and process the result. One more advantage is that a Cloudwatch Sheduler rule doesn't require SNS and doesn't consume more lambda code execution time compared to the SNS triggered on an average when there is more activity, as it is not triggered per event.


>What Is AWS Lambda?

AWS Lambda is a compute service where you can upload your code and the service can run the code on your behalf using AWS infrastructure. AWS Lambda executes your code only when needed and scales automatically, from a few requests per day to thousands per second. On top of that few hundered thousand seconds of execution time per month are completely free, See [pricing](https://aws.amazon.com/lambda/pricing/).


>What Is SNS?

Amazon Simple Notification Service (Amazon SNS) is a fast, flexible, fully managed push notification service that lets you send individual messages or to fan-out messages to large numbers of recipients. Amazon SNS makes it simple and cost effective to send push notifications to mobile device users, email recipients or even send messages to other distributed services.

With Amazon SNS, you can send notifications to Apple, Google, Fire OS, and Windows devices, as well as to Android devices in China with Baidu Cloud Push. You can use SNS to send SMS messages to mobile device users worldwide.

Beyond these endpoints, Amazon SNS can also deliver messages to Amazon Simple Queue Service (SQS), AWS Lambda functions, or to any HTTP endpoint.Like Lambda we have free 1 million requests to SNS Push Services , See [pricing](https://aws.amazon.com/sns/pricing/)



# Slack it!! with SNS-Lambda
First we will try to leverage SNS with Lambda function to deliver more customizable and pleasing messages to Slack.

## Configure Slack Channel
To send notifications to our slack channel we will need to first Integrate an Incoming Webhook App to the channel.
Follow the steps as suggested and select your channel name.

![Slack Channel #aws-sns]({{ site.url }}{{ site.baseurl }}/images/slackit/slack_channel.gif)

 Copy the Webhook URL, we will be needing it later for our lambda function.

## Create Lambda Function
The next part is to create a lambda function that could read event data spit out by Cloudwatch to the SNS Topic. It will be a json and we will be using NodeJS to read the data, translate it and send the message to slack.

<img src="{{ site.url }}{{ site.baseurl }}/images/slackit/aws_lambda.gif" alt="AWS-SNS-Lambda" class="full">


## Configure SNS
Next we will need to configure an SNS Topic and our new lamda function as a subscriber , optionally we can subscriber an emailID for the same topic.

<img src="{{ site.url }}{{ site.baseurl }}/images/slackit/sns_topic_lambda.gif" alt="SNS Topic - Lambda Subscriber">

[comment]: # ( TODO: Image of message and code )

## Configure Cloudwatch
Next step is to setup the cloudwatch rules to trigger SNS when an event occurs on your Elastic Cloud Instances.

![Cloudwatch Rules SNS]({{ site.url }}{{ site.baseurl }}/images/slackit/cloudwatch_sns_lambda.gif)


# Slack it!! with lambda
Cloudwatch can not only trigger SNS/Lamba when an event occurs(Say Running -> Shutdown or CPU Utilization of Particular cluster of instances is higher and you would want to add one more node), it can also schedule an aws lambda function. When we would like to check periodically the status of an Instance(s) or the number of running instances, we can use Cloudwatch as a Scheduler to trigger lambda function, thereby removing the need of SNS.

## Configure Slack Channel
As described previously configure a webhook with your slack channel.

![Slack Channel]({{ site.url }}{{ site.baseurl }}/images/slackit/slack_channel.gif){: .full}

## Create lambda Function
Now we will try a different language, python2.7 is recently supported by lambda. We will also leverage *aws-sdk* and *boto3* to retrieve information about running instances.

<img src="{{ site.url }}{{ site.baseurl }}/images/slackit/aws_lambda_ec2.gif" alt="AWS Lambda Ec2 Monitor">


Here is the lambda code :

<script src="https://gist.github.com/yabhinav/a9161582c7e520c5beaf9e11663dc20f.js"></script>


## Configure Cloudwatch

We will add a Cloudwatch Scheduler to trigger our lambda function , say every 5hours.

![Cloudwatch Rules SNS]({{ site.url }}{{ site.baseurl }}/images/slackit/cloudwatch_lambda.gif)


# We can Slack now!!

The final message delivered to slack would look something like this ...

![Your Slack Message]({{ site.url }}{{ site.baseurl }}/images/slackit/slack_message.jpg)


All Done!! Save some Money, be Vigil!! See you later !!
