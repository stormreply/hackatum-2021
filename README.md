# Hackatum-2021 - Build a Proof Of Concept IoT solution with AWS

This workshop will show you how to create a proof of concept project on AWS. During the workshop you will create a virtual IoT device, connect the device to the internet, send and receive MQTT messages and remotely operate and update the device.

## Overview

In this workshop, we will simulate an IoT device with an EC2 instance. We will build and install some open source device software, namely the AWS IoT Device Client on the virtual IoT device. The AWS IoT Device Client will enable your virtual IoT device to work with AWS IoT Core, AWS IoT Device Management, and AWS IoT Device Defender features. You will learn how to:

- Create a "Thing" - an identity for your device on AWS IoT Core
- Set up and configure the AWS IoT Device Client on your IoT device, and securely connect your "Thing" to AWS IoT Core
- Exchange MQTT messages between your IoT device and AWS IoT Core
- Define and deploy an Over-The-Air (OTA) remote operation to your IoT device with Jobs, a feature of AWS IoT Device Management

IMAGE

Set up a device "heart-beat", monitor device health metrics and security using AWS IoT Device Defender
Set up synchronous access (SSH) to your IoT device with Secure Tunneling, a feature of AWS IoT Device Management

## 1. Introduction

### Prerequisites

- You must have access to an AWS account with Administrator Access privileges
- Basic knowledge of Linux
- Basic understanding of programming languages

### Using an AWS Cloud9 instance
You do not need a physical IoT device for this workshop. Instead, we create a virtual "Thing" using an EC2  instance. On this EC2instance, we use Ubuntu Linux as the operating system, and the standard user is named "ubuntu". We interact with our EC2 instance using AWS Cloud9.

The AWS Cloud9 IDE offers a rich code-editing experience with support for several programming languages and runtime debuggers, and a built-in terminal. It contains a collection of tools that you use to code, build, run, test, and debug software and helps you release software to the cloud.

You access the AWS Cloud9 IDE through a web browser.

The AWS Command Line Interface (AWS CLI) will be installed and configured on the AWS Cloud9 Instance. If at any point, this workshop's instructions refer to the AWS CLI, use it on a terminal in the Cloud9 environment that is running on your EC2 instance.

## 2. Hands On

### Steps

- Login to your account
- Select the region you will work on
- Launch the Cloudformation stack ( template is in the template folder )
- Leave the default values for Stack name, AWS Cloud9 Instance Type untouched
- Capabilities (at the bottom of the page):
- Check I acknowledge that AWS CloudFormation might create IAM resources with custom names.
- Check I acknowledge that AWS CloudFormation might require the following capability: CAPABILITY_AUTO_EXPAND
- Select Create stack
- Wait until the full stack is created. It should take about 10 minutes.

IMAGE 2

You should see a webpage similar to the screenshot below (color scheme may be different):

IMAGE 3

The Cloud9 IDE offers a built-in terminal that is used to type commands on the EC2 instance. Open a new terminal using the + sign in the tab bar.

### Enable AWS IoT Core to publish event messages

#### Event messages

AWS IoT publishes event messages when certain events occur. For example, the AWS IoT Core Registry generates events when things are created, updated, or deleted. Each event triggers a single message. Event messages are published over MQTT with a JSON payload. The content of the payload depends on the type of event.

Since event messages incur MQTT messaging charges on your AWS account, the AWS IoT Console doesn’t subscribe to events by default.

#### Enable event messages in the AWS IoT Console

To enable event messages, go to the Settings tab of the AWS IoT Core console  and then, in the Event-based messages section, choose Manage events. Select all the events and Confirm.

IMAGE 4

Select all events and confirm

IMAGE 5

This completes the procedure

## 3. Setup the virtual IoT device

In this section, you will provision and connect your IoT device. For this, you will do two things:

- You will "Create a Thing" on AWS IoT Core - as part of this, you will create an AWS-recognizable "identity" for your IoT device, and provide it a policy to interact with your AWS resources.
- You will complete a device-side setup, install the AWS IoT Device Client software on your IoT device, and use the identity you created to connect your device securely to AWS IoT.

Let's proceed to next section to start the provisioning and connection process.

### Create a Thing on AWS IoT

First, let's navigate to the AWS IoT console. From the Cloud9 interface, go to the dashboard

IMAGE 6

In the search bar type AWS IoT Core and select it.

IMAGE 7

In the AWS IoT console select Manage > Things and then select Create Things

IMAGE 8

Create Single Thing

Name your thing: deviceClientThing

IMAGE 9

Auto-generate certificates (recommended)

IMAGE 10

Create policy (this will open a new tab)

IMAGE 11

In the Policy creation page, follow steps as displayed below and create a policy.

IMAGE 12

Go back to the earlier Thing creation page and complete the creation process using the policy you just created.

IMAGE 13

Once Thing creation is complete, you will see the following pop-up:

IMAGE 14

Now that we have all four files, let's proceed to the device-side setup.

With this step, you have successfully created a Thing on AWS IoT Core, along with an AWS-recognizable identity (the certificates and keys), and provided a policy to define how your Thing interacts with AWS resources.

### Prepare the virtual IoT device and install the AWS IoT Device Client

TODO

### Exchange MQTT messages with the AWS IoT  Core

TODO

## Deploy an Over-The-Air (OTA) remote operation with AWS IoT Jobs

TODO

## Next steps

If you want to proceed with this workshop and you want to learn:
- How to monitor and audit your virtual IoT device security with AWS IoT Device Defender
- Synchronously access your IoT device with AWS IoT Device Management ( Secure Tunneling)
- How to troubleshoot

Please read the links in the sources.

These links will help you to have a complete overview of the AWS IoT services needed for a POC.

## Sources

- https://aws.amazon.com/blogs/iot/build-a-proof-of-concept-iot-solution-in-under-3-hours-with-the-aws-iot-device-client/
- http://getstartedwithawsiot.com/






