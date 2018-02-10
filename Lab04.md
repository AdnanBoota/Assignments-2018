# Lab session 4: Creating a web application using cloud PaaS

This hands-on session is based on the [AWS Elastic Beanstalk tutorial](https://docs.aws.amazon.com/gettingstarted/latest/deploy/overview.html). Here, we are going to build a small web application using [Django](https://www.djangoproject.com/) which is a modern framework that uses Python to create web applications.

##  AWS IaaS: Elastic Beanstalk

Elastic Beanstalk is a high-level deployment tool that helps you get an app from your desktop to the web in a matter of minutes. Elastic Beanstalk handles the details of your hosting environment—capacity provisioning, load balancing, scaling, and application health monitoring—so you don't have to.

Elastic Beanstalk supports apps developed in Python as well as many other programming languages. It also admits multiple configurations for each platform. A platform configuration defines the infrastructure and software stack to be used for a given environment.

When you deploy your app, Elastic Beanstalk provisions a set of AWS resources that can include Amazon EC2 instances, alarms, a load balancer, security groups, and more.

The software stack that runs your application depends on the platform configuration type. For example, Elastic Beanstalk supports several configurations for Python: Python 3.x, Python 2.x.

You can interact with Elastic Beanstalk by using the **AWS Management Console**, the **AWS Command Line Interface (AWS CLI)**, or the **Elastic Beanstalk CLI**: a high-level CLI designed explicitly for Elastic Beanstalk.

For this tutorial, we'll use the `AWS Management Console` in conjunction with `Elastic Beanstalk CLI`.

## Deploying an example Web App Using Elastic Beanstalk

We will assume that you're working on a new subject on Cloud Computing that isn't ready for students to enroll yet, but in the meantime, you plan to deploy a small placeholder app that collects contact information from site visitors who sign up to hear more. The signup app will help you reach potential students who might take part in a private beta test of the laboratory sessions.

### The Signup App
The app will let your future students submit contact information and express interest in a preview of the new subject on Cloud Computing that you're developing.

To make our app look good, we use Bootstrap, a mobile-first front-end framework that started as a Twitter project.

### Amazon DynamoDB
We will use **Amazon DynamoDB**, a NoSQL database service, to store the contact information that users submit. DynamoDB is a schema-less database, so you need to specify only a primary key attribute. We'll use an email field as a key for our app.

#  Tasks for Lab session #4

## Prerequisites
You need to have `AWS CLI` and `AWS EB CLI` installed and configured. Please complete [Getting Started in the Cloud (with AWS)](https://github.com/CCBDA-UPC/Cloud-Computing-QuickStart/blob/master/Quick-Start-AWS.md) before beginning work on this assignment.

* [Task 4.1: Download the repository for the Web App](#Tasks41)
* [Task 4.2: Create an IAM Policy and Role](#Tasks42)
* [Task 4.3: Create a DynamoDB Table](#Tasks43)
* [Task 4.4: Test the web app locally](#Tasks44)
* [Task 4.5: Create the AWS Beanstalk environment and deploy a sample web app](#Tasks45)
* [Task 4.6: Configure Elastic Beanstalk CLI and deploy the target web app](#Tasks46)


<a name="Tasks41"/>

## Task 4.1: Download the repository for the Web App
You are going to make a few changes to the base Python code. Therefore, download the repository on your local disk drive as a zip file from *https://github.com/CCBDA-UPC/eb-django-express-signup-base*. Unzip the file and change the name of the folder to *eb-django-express-signup*.

Prepare a new **private** repository in GitHub named `eb-django-express-signup` to commit all the changes to your code. Invite `angeltoribio-UPC-BCN` to your new remote private repository as a collaborator.

Do not mix the repository containing the course answers with the repository that holds the changes to your web app. 

<a name="Tasks42"/>

## Task 4.2: Create an IAM Policy and Role

Next, you need to create an IAM role that grants your web app permission to put items into your DynamoDB table. You will apply the role to the EC2 instances that run your application when you create an AWS Elastic Beanstalk environment.

#### To create an IAM policy

1. Open the [AWS Identity and Access Management (IAM) console](https://console.aws.amazon.com/iam).

2. In the navigation pane, choose **Policies**.

3. Choose **Create policy**.

4. Next, select the **JSON** tab and paste the contents of the file `iam_policy.json` that you will find at the extra-file folder of the repository.

5. For Policy Name, enter **gsg-signup-policy**.

6. Choose **Create Policy**.

Create an IAM role and attach the policy to it.

#### To create an IAM role

1. In the navigation pane, choose **Roles**.

2. Choose **Create role**.

3. On the **AWS service** tab, select **EC2** service, and again **EC2** to allow EC2 instances to call AWS services on your behalf. Hit **Next:Permissions**.

    <p align="center"><img src="./images/Lab04-1.png " alt="Lab04-1" title="AWS service"/></p>

4. On the **Attach permissions policies** page, attach the following policies.

    - **gsg-signup-policy** – The policy that you created earlier.
    <p align="center"><img src="./images/Lab04-2.png " alt="Lab04-2" title="gsg-signup-polic"/></p>

    - **AWSElasticBeanstalkWebTier** – Elastic Beanstalk provided role that allows the instances in your environment to upload logs to Amazon S3.
    <p align="center"><img src="./images/Lab04-3.png " alt="Lab04-3" title="AWSElasticBeanstalkWebTier"/></p>

    To locate policies quickly, type part of the policy name in the filter box. Select both policies and then choose **Next Step**.

5. For Role name, enter **gsg-signup-role**.

6. Choose **Create role**.

For more information on permissions, see [http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts-roles.html](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts-roles.html) in the AWS Elastic Beanstalk Developer Guide.

<a name="Tasks43"/>

## Task 4.3: Create a DynamoDB Table
Our signup app uses a DynamoDB table to store the contact information that users submit.

#### To create a DynamoDB table

1. Open the DynamoDB console at [https://console.aws.amazon.com/dynamodb/home](https://console.aws.amazon.com/dynamodb/home).

2. In the menu bar, ensure that the region is set to **Ireland**.

3. Choose **Create table**.

4. For Table name, type **gsg-signup-table**.

5. For the `Primary key`, type `email`. Choose **Create**.

<a name="Tasks44"/>

## Task 4.4: Test the web app locally

Once you are inside the directory of the project issue the following commands to setup the configuration of the project using environment variables:

```
_$ export DEBUG="True"
_$ export STARTUP_SIGNUP_TABLE="gsg-signup-table"
_$ export AWS_REGION="eu-west-1"
```

You can also type in the command line:

```
_$ source extra-files/environment.sh 
```

Next, create a new Python virtual environment and install the packages required to run de web app. The package `boto3` is a library that hides de AWS REST API to the programmer and manages the communication between the web app and all the AWS services. Check [**Boto 3 Documentation**](https://boto3.readthedocs.io/en/latest/reference/services/index.html) for more details.

Please, note the different prompt **(eb-virt)_$** vs. **_$** when you are inside and outside of the new virtual environment.

```
_$ virtualenv ../eb-virt
_$ source ../eb-virt/bin/activate
(eb-virt)_$ pip install django
(eb-virt)_$ pip install boto3
```

You will now need to initialize the web app local database and run a local testing server.

```
(eb-virt)~$ python manage.py migrate
(eb-virt)~$ python manage.py runserver
System check identified no issues (0 silenced).
February 04, 2018 - 18:57:49
Django version 2.0.2, using settings 'eb-django-express-signup.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
(eb-virt)_$ deactivate
```
Check that you have configured the access to DynamoDB correctly by interacting with the web app through your browser [http://127.0.0.1:8000/](http://127.0.0.1:8000/).

Go to the DynamoDB browser tab and verify that the **gsg-signup-table** table contains the new records that the web app should have created. If all the above works correctly, you are almost ready to transfer the web app to AWS Beanstalk.

<a name="Tasks45"/>

## Task 4.5: Create the AWS Beanstalk environment and deploy the sample web app

### Prepare some configuration for AWS Beanstalk

At the repository, you already have a `requirements.txt` file that will let AWS Beanstalk know which Python modules your web app needs. As you advance in this hands-on, you will install more Python modules, and you will need to update `requirements.txt`. Please note that you first need to switch to the virtual environment to update the file.

```
_$ source ../eb-virt/bin/activate
(eb-virt)_$ pip freeze > requirements.txt
(eb-virt)_$ deactivate
```

The Beanstalk environment will use this command to install the exact version of packages that our web app needs.

```
(eb-virt)_$ pip install -r requirements.txt
```

Another important set of files are the ones at the `.ebextensions` directory. It contains instructions for AWS Beanstalk to start the web app and set up the timezone.

### Launch your new Elastic Beanstalk environment

Open the Elastic Beanstalk console using this preconfigured link: [https://console.aws.amazon.com/elasticbeanstalk/home#/newApplication?applicationName=gsg-signup&environmentType=LoadBalanced](https://console.aws.amazon.com/elasticbeanstalk/home#/newApplication?applicationName=gsg-signup&environmentType=LoadBalanced).

1. See the name at its place and choose **Next**.

2. For New Environment, choose **Create webserver**.

3. For Environment Type, select **Python**, **Load balancing, auto-scaling** and choose **Next**.

4. For Application version, select **Sample application** and choose **Next**.

5. For Environment Info, accept the proposed name (i.e., gsgSignup-4dxpj-env). Alternatively, you can find one name that you like better, and it is available. Continue by choosing **Next**.

    Please note that `gsgSignup-4dxpj-env` is also part of the URL we will use to access the project: `http://gsgSignup-4dxpj-env.eu-west-1.elasticbeanstalk.com/`.

6. For Additional Resources, select **Create this environment inside a VPC** and choose **Next**.

7. For Configuration Details, select a `t2.micro` Instance type (the smallest EC2 instance that you can pick) and a key pair that you have access to. Type your e-mail address to receive notifications regarding the environment that you are launching and finally choose **Next**.

8. For Environment Tags, need to create the same environment variables that you created for the local test. Once having entered the data, choose **Next**.
    <p align="center"><img src="./images/Lab04-4.png " alt="Lab04-4" title="Environment"/></p>

9. For VPC Configuration, select the proposed VPC (if you haven't created one yet) and also select `eu-west-1a` for `ELB` and `EC2`. Select a VPC security group that has the set of Internet ports that are open for inbound and outbound connections. Make sure that ELB visibility is External and choose **Next**.
    <p align="center"><img src="./images/Lab04-5.png " alt="Lab04-5" title="VPC Configuration"/></p>

10. For Permissions, choose the Instance profile that you created earlier (**gsg-signup-role**) and choose **Next**.
    <p align="center"><img src="./images/Lab04-6.png " alt="Lab04-6" title="gsg-signup-role"/></p>

11. Review all the settings and choose **Launch**.

<p align="center"><img src="./images/Lab04-7.png " alt="Lab04-7" title="Launching"/></p>

In just a few minutes, Elastic Beanstalk provisions the networking, storage, compute and monitoring infrastructure required to run a scalable web application in AWS. Once the site is up and running, at any time, you can deploy a new version of your application code to the cloud environment.

Once you see the following status, you can click on the URL field (next to the `Actions` button at the top right corner of the browser window).

<p align="center"><img src="./images/Lab04-8.png " alt="Lab04-8" title="OK"/></p>

A new tab will open showing:

<p align="center"><img src="./images/Lab04-9.png " alt="Lab04-9" title="Sample web app"/></p>

Good job! We are almost there.

<a name="Tasks46"/>

## Task 4.6: Configure Elastic Beanstalk CLI and deploy the target web app

At this point, we have the sample web app deployed. AWS EB CLI will help us to transfer and install our web app to the cloud. Go to your terminal window and write:

```
_$ eb init
Select a default region
...
4) eu-west-1 : EU (Ireland)
...
(default is 3): 4

Select an application to use
1) [ Create new Application ]
(default is 1): 1

Enter Application Name
(default is "eb-django-express-signup"):
Application eb-django-express-signup has been created.

It appears you are using Python. Is this correct?
(Y/n): y

Select a platform version.
1) Python 3.6
...
(default is 1): 1
Note: Elastic Beanstalk now supports AWS CodeCommit; a fully-managed source control service. To learn more, see Docs: https://aws.amazon.com/codecommit/
Do you wish to continue with CodeCommit? (y/N) (default is n): n
Do you want to set up SSH for your instances?
(Y/n): y

Select a key pair.
1) ccbda_upc
2) [ Create new KeyPair ]
(default is 2): 1
_$ eb use gsgSignup-4dxpj-env
```
Please note that `gsgSignup-4dxpj-env` is the environment name that we created previously and it also part of the URL we will use to access the project: `http://gsgSignup-4dxpj-env.eu-west-1.elasticbeanstalk.com/`.

Running eb init creates a configuration file at `.elasticbeanstalk/config.yml`. You can edit it if necessary.

```
branch-defaults:
  master:
    environment: gsgSignup-4dxpj-env
    group_suffix: null
global:
  application_name: eb-django-express-signup
  branch: null
  default_ec2_keyname: ccbda_upc
  default_platform: Python 3.6
  default_region: eu-west-1
  include_git_submodules: true
  instance_profile: null
  platform_name: null
  platform_version: null
  profile: null
  repository: null
  sc: git
  workspace_type: Application

```

Finally, you only need to type `deploy` to transfer the code to AWS.

Please note that if you haven't committed all the changes in your local repository eb deploy will deploy the last committed changes.
```
_$ eb deploy
Creating application version archive "app-b2f2-180205_205630".
Uploading eb-django-express-signup/app-b2f2-180205_205630.zip to S3. This may take a while.
Upload Complete.
INFO: Environment update is starting.
INFO: Deploying new version to instance(s).
INFO: New application version was deployed to running EC2 instances.
INFO: Environment update completed successfully.

```

### Test the Web App

Your EBS console will show changes, as seen below.

<p align="center"><img src="./images/Lab04-10.png " alt="Lab04-10" title="Sample web app"/></p>

When the deployment has finished, and the environment health is listed as "Green", open the URL of the app, and you should see the screen capture shown below.

<p align="center"><img src="./images/Lab04-11.png " alt="Lab04-11" title="Sample web app"/></p>

Interact with the web app and check that the new records inserted really are stored in DynamoDB.

### Troubleshoot Deployment Issues

If you followed all the steps, opened the URL, and got no app, there's a deployment problem. To troubleshoot a deployment issue, you may need to use the logs that are provided by Elastic Beanstalk.

Of course, you'd try to catch such an error in development. But if an error does get through to production, or you just want to update your app, Elastic Beanstalk makes it fast and easy to redeploy. Just modify your code, commit the changes, and issue deploy again.

```
_$ eb deploy
```

In the Elastic Beanstalk console, in the navigation pane for your environment, choose Logs, download them and try to find the error. For instance one of the environment variables are not set correctly.

### One final step

Before ending this session, please go to your Elastic Beanstalk console, unfold the **Actions** button and **Save Configuration**. It will be useful for you to continue with the environment in the next Lab session.

<p align="center"><img src="./images/Lab04-12.png " alt="Lab04-12" title="Save configuration"/></p>

Go to your EC2 console and check the EC2 instance that AWS uses for the Elastic Beanstalk environment. Terminate the instance. Check what happens in your EBS console. Wait a couple of minutes and check again your EC2 console. What has happened? Why do you think that has happened? Add your responses to `README.md`.

<p align="center"><img src="./images/Lab04-13.png " alt="Lab04-13" title="EC instances"/></p>

Now, to save expenses, you can terminate your environment, this time from the EBS console.  What has happened? Why do you think that has happened? Check both EC2 and EBS consoles. Add your responses to `README.md`.

# How to Submit this Assignment:

Make some screen captures of your:
- DyanmoDB table with the data of the new leads.
- Make sure you have written your responses to the above questions in `README.md`.
- Commit the files to the repository.


Submit **before the deadline** to the *RACO Practicals section* a "Lab4.txt" file including:

1. Group number
2. Name and email of the members of this group
3. Address of the GitHub repository with your solution
4. Add any comment that you consider necessary