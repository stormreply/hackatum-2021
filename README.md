# Hackatum-2021 - Build a Proof Of Concept IoT solution with AWS

This workshop will show you how to create a proof of concept project on AWS. During the workshop you will create a virtual IoT device, connect the device to the internet, send and receive MQTT messages and remotely operate and update the device.

## Overview

In this workshop, we will simulate an IoT device with an EC2 instance. We will build and install some open source device software, namely the AWS IoT Device Client on the virtual IoT device. The AWS IoT Device Client will enable your virtual IoT device to work with AWS IoT Core, AWS IoT Device Management, and AWS IoT Device Defender features. You will learn how to:

- Create a "Thing" - an identity for your device on AWS IoT Core
- Set up and configure the AWS IoT Device Client on your IoT device, and securely connect your "Thing" to AWS IoT Core
- Exchange MQTT messages between your IoT device and AWS IoT Core
- Define and deploy an Over-The-Air (OTA) remote operation to your IoT device with Jobs, a feature of AWS IoT Device Management

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/1.PNG)

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

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/2.PNG)

You should see a webpage similar to the screenshot below (color scheme may be different):

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/3.PNG)

The Cloud9 IDE offers a built-in terminal that is used to type commands on the EC2 instance. Open a new terminal using the + sign in the tab bar.

### Enable AWS IoT Core to publish event messages

#### Event messages

AWS IoT publishes event messages when certain events occur. For example, the AWS IoT Core Registry generates events when things are created, updated, or deleted. Each event triggers a single message. Event messages are published over MQTT with a JSON payload. The content of the payload depends on the type of event.

Since event messages incur MQTT messaging charges on your AWS account, the AWS IoT Console doesn’t subscribe to events by default.

#### Enable event messages in the AWS IoT Console

To enable event messages, go to the Settings tab of the AWS IoT Core console  and then, in the Event-based messages section, choose Manage events. Select all the events and Confirm.

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/4.PNG)

Select all events and confirm

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/5.PNG)

This completes the procedure

## 3. Setup the virtual IoT device

In this section, you will provision and connect your IoT device. For this, you will do two things:

- You will "Create a Thing" on AWS IoT Core - as part of this, you will create an AWS-recognizable "identity" for your IoT device, and provide it a policy to interact with your AWS resources.
- You will complete a device-side setup, install the AWS IoT Device Client software on your IoT device, and use the identity you created to connect your device securely to AWS IoT.

Let's proceed to next section to start the provisioning and connection process.

### Create a Thing on AWS IoT

First, let's navigate to the AWS IoT console. From the Cloud9 interface, go to the dashboard

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/6.PNG)

In the search bar type AWS IoT Core and select it.

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/7.PNG)

In the AWS IoT console select Manage > Things and then select Create Things

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/8.PNG)

Create Single Thing

Name your thing: deviceClientThing

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/9.PNG)

Auto-generate certificates (recommended)

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/10.PNG)

Create policy (this will open a new tab)

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/11.PNG)

In the Policy creation page, follow steps as displayed below and create a policy.

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/12.PNG)

Go back to the earlier Thing creation page and complete the creation process using the policy you just created.

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/13.PNG)

Once Thing creation is complete, you will see the following pop-up:

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/14.PNG)

Now that we have all four files, let's proceed to the device-side setup.

With this step, you have successfully created a Thing on AWS IoT Core, along with an AWS-recognizable identity (the certificates and keys), and provided a policy to define how your Thing interacts with AWS resources.

### Prepare the virtual IoT device and install the AWS IoT Device Client

Now that we have all four files, let's upload these files to the right directory location on our EC2 instance in our Cloud9 environment. First, in your Cloud9 environment, click the little gear in the directory navigation pane, and select Show Home in Favorites:

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/15.PNG) 

Next, navigate to the folder */home/ubuntu/workshop_dc/certs* in the directory navigation pane

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/16.PNG)

Upload all the four downloaded files to our Cloud9 instance folder */home/ubuntu/workshop_dc/certs*

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/17.PNG)

Please make sure you have uploaded following 3 files ending with: .crt, .pem.key and AmazonRootCA.pem specifically to the */home/ubuntu/workshop_dc/certs* directory before proceeding.

Let's set the correct security permissions on your IoT device for the certificates, private key, and directory they are stored in (if you saved your certificates in another directory please set the folder path to that)

```
chmod 700 certs
chmod 600 /home/ubuntu/workshop_dc/certs/*-private.pem.key
chmod 644 /home/ubuntu/workshop_dc/certs/*-certificate.pem.crt
chmod 644 /home/ubuntu/workshop_dc/certs/AmazonRootCA1.pem
```

Let's create following file to store the data incoming (Sub) from AWS IoT Core (cloud), this will keep record of what data we received (this is optional step and required if you want to save data to review later on what was posted by device and what was received from cloud)

- Subscribe file

```
touch  /home/ubuntu/workshop_dc/subfile.txt
chmod 600 /home/ubuntu/workshop_dc/subfile.txt
```

In a Cloud9 terminal, please copy-paste the following commands to build and compile the AWS IoT Device Client software for your IoT device:

```
cd /home/ubuntu/workshop_dc
chmod ugo+x build_execute.sh
./build_execute.sh
```

The above commands

- Clone the open source repo of the Device Client on your IoT device, and
- Build the source code
-- Generate a native build environment
-- Compile source code
-- Create libraries
-- Generate wrappers
-- Build executable
- When the previous commands all succeed, you should see a stream of output that ends with:

```
[ 12%] Built target aws-c-common
[ 12%] Built target aws-c-compression
[ 55%] Built target s2n
[ 56%] Built target aws-c-cal
[ 62%] Built target aws-c-io
[ 67%] Built target aws-c-http
[ 69%] Built target aws-c-mqtt
[ 71%] Built target aws-c-iot
[ 71%] Built target aws-checksums
[ 73%] Built target aws-c-event-stream
[ 79%] Built target aws-c-auth
[ 85%] Built target aws-crt-cpp
[ 85%] Built target IotDeviceCommon-cpp
[ 86%] Built target IotSecureTunneling-cpp
[ 92%] Built target IotJobs-cpp
[ 95%] Built target IotIdentity-cpp
[ 95%] Built target IotDeviceDefender-cpp
[100%] Built target aws-iot-device-client
```

Let's now configure the Device Client to connect to AWS IoT Core.

We will run the set up script (setup.sh) to configure AWS IoT Device Client on your IoT device.

```
cd ~/workshop_dc/aws-iot-device-client/
sudo ./setup.sh 
```

#### Step 1

```
Do you want to interactively generate a configuration file for the AWS IoT Device Client? y/n
```

Select y (inputs are case sensitive, all choices will be lower-case)

#### Step 2

```
Specify AWS IoT endpoint to use:
```

In your second terminal window, run the following command: (change region to the one your using if you aren't using the Ireland region):

```
aws --region eu-west-1 iot describe-endpoint --endpoint-type "iot:Data-ATS"
```

Use the output from the command above to specify the AWS IoT endpoint we will use (without quotes)

```
Specify path to public PEM certificate:
```

In your second terminal window obtain the complete file path of your public PEM certificate.

```
cd /home/ubuntu/workshop_dc/certs
ls
```

You will see a list of 4 files (see image below).

Click on the file with the suffix -certificate.pem.crt, click on Copy Special, and click on Copy File Path

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/18.PNG)

paste the file path in the configuration utility in your first terminal window

when you paste, you should see a file path that looks like */home/ubuntu/workshop_dc/certs/xxxxxxxxxxxxxxxxxxxxxxxx-certificate.pem.crt*

```
Specify path to private key:
```

From your second terminal window obtain the complete file path of your private PEM key.

use the same procedure above to copy its full file path, and paste it in the configuration utility in your first terminal window

when you paste, you should see a file path that looks like */home/ubuntu/workshop_dc/certs/xxxxxxxxxxxxxxxxxxxxxxxx-private.pem.key*

Specify path to ROOT CA certificate:

From your second terminal window obtain the complete file path of your Amazon Root CA certificate.

use the same procedure above to copy its full file path, and paste it in the configuration utility in your first terminal window

when you paste, you should see a file path that looks like */home/ubuntu/workshop_dc/certs/AmazonRootCA1.pem*

```
Specify Thing name:
```

If you followed instructions in the "Create a Thing on AWS IoT" step, use the name deviceClientThing. Otherwise, enter the name of the thing you created in that step (case sensitive).

for eg. mine is: *deviceClientThing*

```
Would you like to configure the logger? y/n
```

Select *y*

```
Specify desired log level: DEBUG/INFO/WARN/ERROR
```

Type *DEBUG*

```
Specify log type: STDOUT for standard output, FILE for file
```

Type *FILE*

```
Specify path to desired log file (if no path is provided, will default to /var/log/aws-iot-device-client/aws-iot-device-client.log):
```

Just hit Enter without typing anything to use the default log file location

```
Enable Jobs feature? y/n
```

Select *y*

```
Specify absolute path to Job handler directory (if no path is provided, will default to /home/ubuntu/.aws-iot-device-client/jobs):
```

Press *Enter* without typing anything to use default path

```
Enable Secure Tunneling feature? y/n
```

Select *y*

```
Enable Device Defender feature? y/n
```

Select *y*

```
Specify an interval for Device Defender in seconds (default is 300):
```

Hit Enter without typing anything to use the default time duration in seconds

```
Enable Fleet Provisioning feature? y/n
```

Select *n*

```
Enable Pub Sub sample feature? y/n
```

Select *y*

```
Specify a topic for the feature to publish to:
```

Type:

```
/topic/workshop/dc/pub
```

```
Specify the path of a file for the feature to publish (Leaving this blank will publish 'Hello World!'):
```

Hit *Enter* without typing anything to use the default setting

```
Specify a topic for the feature to subscribe to:
```

Type:

```
/topic/workshop/dc/sub
```

```
Specify the path of a file for the feature to write to (Optional):
```

```
/home/ubuntu/workshop_dc/subfile.txt
```

```
Enable Config Shadow feature? y/n
```

Select *n*

```
Enable Sample Shadow feature? y/n
```

Select *n*

Review the following:

```
Does the following configuration appear correct? If yes, configuration will be written to /home/ubuntu/.aws-iot-device-client/aws-iot-device-client.conf: y/n

 {
      "endpoint":       "xxxxxxxxxxxxxxxxxxxxxxxx-ats.iot.eu-west-1.amazonaws.com",
      "cert":   "/home/ubuntu/workshop_dc/certs/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx-certificate.pem.crt",
      "key":    "/home/ubuntu/workshop_dc/certs/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx-private.pem.key",
      "root-ca":        "/home/ubuntu/workshop_dc/certs/AmazonRootCA1.pem",
      "thing-name":     "deviceClientThing",
      "logging":        {
        "level":        "DEBUG",
        "type": "FILE",
        "file": "/home/ubuntu/workshop_dc/dc.log"
      },
      "jobs":   {
        "enabled":      true,
        "handler-directory": "/root/.aws-iot-device-client/jobs"
      },
      "tunneling":      {
        "enabled":      true
      },
      "device-defender":        {
        "enabled":      true,
        "interval": 300
      },
      "fleet-provisioning":     {
        "enabled":      false,
        "template-name": "",
        "template-parameters": "",
        "csr-file": "",
        "device-key": ""
      },
      "samples": {
        "pub-sub": {
          "enabled": true,
          "publish-topic": "/topic/workshop/dc/pub",
          "publish-file": "",
          "subscribe-topic": "/topic/workshop/dc/sub",
          "subscribe-file": ""
        }
      },
      "config-shadow":  {
        "enabled":      false
      },
      "sample-shadow": {
        "enabled": false,
        "shadow-name": "",
        "shadow-input-file": "",
        "shadow-output-file": ""
      }
    }
```

Select y if everything is correct or select n to remedy any mistakes

```
Do you want to copy the sample job handlers to the specified handler directory (/home/ubuntu/.aws-iot-device-client/jobs)? y/n
```

Select *y*

```
Do you want to install AWS IoT Device Client as a service? y/n
```

Select *y*

```
Enter the complete directory path for the aws-iot-device-client. (Empty for default: ./build/aws-iot-device-client)
```

Hit *Enter* without typing anything to use default settings

```
Enter the complete directory path for the aws-iot-device-client service file. (Empty for default: ./setup/aws-iot-device-client.service)
```

Hit *Enter* without typing anything to use default settings

```
Do you want to run the AWS IoT Device Client service via Valgrind for debugging? y/n
```

Select *n*

This will create a config file at this location: /root/.aws-iot-device-client/aws-iot-device-client.conf

When you have successfully completed configuration, you will see the following terminal output:

Here we can see that the AWS IoT Device Client is installed as a service, active and loaded in memory.

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/19.PNG)

With this step, you have successfully prepared your IoT device, and installed the AWS IoT Device Client as a service. It will automatically connect to AWS IoT.

### Exchange MQTT messages with the AWS IoT  Core

If you successfully create a Thing on AWS IoT, and complete the setup of the AWS IoT Device Client on your IoT device, you will receieve a "Hello World" message on the topic */topic/workshop/dc/pub*. The initial startup message will only be published once. So if you do not see the message in the MQTT test client already, run following command: *sudo systemctl restart aws-iot-device-client*. One "Hello World" message is posted every time Device client service starts.

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/20.PNG)

Let's tail the logs from the file location we saw during the Device Client configuration process (if you have created another log file path, please use that)

```
sudo tail -F /var/log/aws-iot-device-client/aws-iot-device-client.log
```

Let's post some data from AWS IoT Core to the Device Client running on our EC2 instance. Go over to the MQTT test client in the AWS IoT Console, and publish a message of your choice to */topic/workshop/dc/sub*

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/21.PNG)

Almost instantly, in your Cloud9 terminal where you're tailing the logs, you will see a log message similar to the one below:

```
2021-09-10T17:01:09.739Z [DEBUG] {samples/PubSubFeature.cpp}: Message received on subscribe topic, size: 45 bytes
```

**If you created file and defined an alternate subscribe file in the setup step you will see data saved in that file.**

Subscribe data file will contain the JSON payloads of all the MQTT messages that we post from AWS IoT Console.

```
cat /home/ubuntu/workshop_dc/subfile.txt
```

You should see the message you posted from the AWS IoT Console. For instance, I see the following output

```
{
  "message": "Hello from AWS IoT console"
}
```

**With this step, you have successfully transmitted MQTT messages between your IoT device and AWS IoT Core. You can extend the samples provided to exchange MQTT messages with your custom device-side IoT application. MQTT data that is recieved on AWS IoT Core can further routed through the AWS IoT Core Rules Engine to other AWS services.**

## Deploy an Over-The-Air (OTA) remote operation with AWS IoT Jobs

### AWS IoT Device Management - Jobs

Jobs is a feature of AWS IoT Device Management that can be used to define a set of remote operations in a Job Document that are sent to and executed on one or more target devices connected to AWS IoT. You can use AWS IoT Jobs for various types of remote operations, including reboots, factory resets, and firmware/app/OS updates.

The corresponding Jobs feature of the AWS IoT Device Client provides device-side functionality to execute the instructions contained in a Job document on your IoT device. When the Jobs feature starts within the AWS IoT Device Client, the feature will attempt to subscribe to all relevant MQTT notification topics, and publish a request to the AWS IoT Device Management service to receive the latest pending Job from the cloud. It will use the shared MQTT connection to subscribe/publish to these topics.

When a new Job is received via these topics, the AWS IoT Device Client Jobs feature will look for and extract the instructions in the JSON payload, and execute the Job on the device side. On the IoT device, the Jobs feature contains Job Handlers that know how to process specific instructions that are sent from the cloud. The Device Client comes with default Handlers for common remote operations like

- reboot
- install / update / un-install applications
- start / stop / restart application
- download file
- verify package installation / removal

You can also create your own Job Handlers for custom remote operations on your IoT devices and trigger them using Jobs. Customers today use Jobs to complete fully customized, very diverse remote operations including configuration updates, application updates, firmware updates, OS patches, state changes, and certificate rotation.

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/22.PNG) 

In this procedure, we demonstrated how Jobs works with AWS IoT Device Client. We uploaded our JSON Jobs document to S3 and then sent the stated message to the Device Client to execute the task. We then verified success both in the cloud as well as on the device side.

With this step, you have successfully defined and deployed an Over-The-Air remote operation with AWS IoT Device Management Jobs on your IoT device running the AWS IoT Device Client.

We will first create an S3 bucket to store our JSON Job document. We will send our device the S3 URL of our Job document so that it can fetch the information required to execute the remote operation.

Let's create our S3 bucket for this workshop and upload our JSON Job document to it.

In further instructions, Replace that should look something like this *workshop-dc-bucket-name-date*

```
aws s3 mb s3://<YOUR-BUCKET-NAME>
```

Successful output: make_bucket: *workshop-dc-bucket-name-date*

On a Cloud9 terminal, we can create our JSON Job document. For now, we will keep it simple. Let's set up a Job that remotely echos a line of text on our IoT device. This line of text will be visible in the terminal window where you are tailing your device log

```
firstJson='{"operation": "echo", "args": ["Hello from <Enter Your Name>, AWS IoT Device Client Workshop user on <Enter Date>."]}'
echo $firstJson > firstJob.json
```

Replace with your previously created Bucket name

```
aws s3 cp firstJob.json s3://<YOUR-BUCKET-NAME>
```

Successful output looks like this: *upload: ./firstJob.json to s3://workshop-dc-bucket-name-date/firstJob.json*

Open AWS IoT Core console  and select Test then MQTT test client

- Subscribe to the following two topics:
-- Subscribe to $aws/events/job/workshopDC_Job/completed
-- Subscribe to $aws/events/jobExecution/workshopDC_Job/succeeded

We have now completed creating S3 bucket, defined our remote operation in the JSON Job document, and stored it in our S3 bucket. Let's open the AWS IoT Core console

From the AWS IoT Core console, navigate to Manage, select Jobs and then Create Job on the top right.

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/23.PNG) 

In the workflow, select Create Custom Job and choose next.

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/24.PNG) 

Give the Job a distinct name i.e. i'm using workshopDC_Job - name accordingly and give description (optional)

Then click on next

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/25.PNG) 

- In File configuration section:
- Under Job Targest > Things to run this Job:
- Select deviceClientThing
- Under File > Job File:
- Select Browse S3
- Then navigate over to your new S3 bucket
- Select Next

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/26.PNG) 

Filter with your bucket name if you have many S3 buckets in your account.

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/27.PNG) 

Select our stored JSON file i.e. firstJob.json and select Choose:

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/28.PNG) 

Select Next

In Job Configuration section:
- Select Snapshot
- Select Submit

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/29.PNG) 

There are three ways to verify the status of the Job.

- First, the MQTT test client in the AWS IoT Console can subscribe to relevant AWS IoT reserved topics to track success messages.
- Second, we can verify the status of the Job in the AWS IoT console on the Jobs dashboard in Manage > Jobs.
- Third, we can verify on the device side through terminal output in our Cloud9 environment

If you had subscribed to following two topics (in step 2) before creating the Job, you will see corresponding messages that confirm success.

Subscribe to *$aws/events/job/workshopDC_Job/completed*

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/30.PNG) 

Subscribe to *$aws/events/jobExecution/workshopDC_Job/succeeded*

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/31.PNG) 

We can also verify Status of the Job from AWS IoT Jobs console when we navigate to Manage > Jobs

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/32.PNG) 

In the Cloud9 terminal where we had the AWS IoT Device Client running, we should see the following output:

- *Hello from <Enter Your Name>, AWS IoT Device Client Workshop user on <Enter Date>*. This should match the instructions you put in the JSON Job document.
- We can also see Jobs executed successfully (use the in-line code formatting) as a command line output.

![IMAGE](https://github.com/stormreply/hackatum-2021/blob/main/.img/33.PNG) 

In this procedure, we demonstrated how Jobs works with AWS IoT Device Client. We uploaded our JSON Jobs document to S3 and then sent the stated message to the Device Client to execute the task. We then verified success both in the cloud as well as on the device side.

With this step, you have successfully defined and deployed an Over-The-Air remote operation with AWS IoT Device Management Jobs on your IoT device running the AWS IoT Device Client.


## Next steps

If you want to proceed with this workshop and you want to learn:
- How to monitor and audit your virtual IoT device security with AWS IoT Device Defender
- Synchronously access your IoT device with AWS IoT Device Management ( Secure Tunneling)
- How to troubleshoot

Please read the links in the sources. (Steps 5-8)

These links will help you to have a complete overview of the AWS IoT services needed for a POC.

## Sources

- https://aws.amazon.com/blogs/iot/build-a-proof-of-concept-iot-solution-in-under-3-hours-with-the-aws-iot-device-client/
- http://getstartedwithawsiot.com/






