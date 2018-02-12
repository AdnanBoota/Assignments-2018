# Lab session #5: Enhancing your web app using additional cloud services

This hands-on session continues the previous lab session where you deployed a basic web app that interacts with students interested in joining your new Cloud Computing course.

For that web app, we use a **DynamoDB table** that stores the list of leads, a **Elastic Beanstalk instance** that runs the Python code that implements the web app functionality.

You had previously set a **IAM Policy and Role** to grant correct access to the resources involved.

The code used sits in a **private** repository of your GitHub account named `eb-django-express-signup`.

Remember that in your EBS console you have the settings of the previously created environment and that you can *unfreeze* the web app by clicking at *Restore terminated environment*.

<p align="center"><img src="./images/Lab05-1.png " alt="Lab05-1" title="Restore environment"/></p>

### Amazon Simple Notification Service
We want to know when customers submit a form, so we will use **Amazon Simple Notification Service** (Amazon SNS), a push messaging service that can deliver notifications over various protocols. For our web app, we will push notifications to an email address.

### Amazon CloudFront CDN

A content delivery network or content distribution network (CDN) is a geographically distributed network of proxy servers that disseminate a service spatially, as close to end-users as possible, to provide high availability, low latency, and high performance.

The information that flows every day on the Internet can be classified as "static" and "dynamic" content. The "dynamic" part is the one that changes depending on the user's input. It is distributed by, for instance, PaaS servers with load balancers. The "static" part does not change based on the user's input and it can be taken as close to the user as possible to improve the "user experience".

Nowadays, CDNs serve a substantial portion of the "static" content of the Internet: text, graphics, scripts, downloadable media files (documents, software products, videos, etc.), live streaming media, on-demand streaming media, social networks and so much more.

Content owners pay CDN operators to deliver the content they produce to their end users. In turn, a CDN pays ISPs (Internet Service Providers), carriers, and network operators for hosting its servers in their data centers.

**AWS CloudFront CDN** is a global CDN service that securely delivers static content with low latency and high transfer speeds. CloudFront CDN works seamlessly with other AWS services including **AWS Shield** for DDoS mitigation, **Amazon S3**, **Elastic Load Balancing** or **Amazon EC2** as origins for your applications, and **AWS Lambda** to run custom code close to final viewers.


#  Tasks for Lab session #5

* [Task 5.1: Use Amazon Simple Notification Service in your web app](#Tasks51)
* [Task 5.2: Create a new option to retrieve the list of leads](#Tasks52)
* [Task 5.3: Improve the web app transfer of information](#Tasks53)
* [Task 5.4: Deliver static content using a Content Delivery Network](#Tasks54)


<a name="Tasks51" />

## Task 5.1: Use Amazon Simple Notification Service in your web app


### Create an SNS Topic

Our signup web app wants to notify you each time a user signs up. When the data from the signup form is written to the DynamoDB table, the app will send you an SNS notification.

First, you need to create an SNS topic, which is a stream for notifications, and then you need to create a subscription that tells SNS where and how to send the notifications.

**To set up Amazon SNS notifications**

Open the Amazon SNS console at [https://console.aws.amazon.com/sns/v2/home](https://console.aws.amazon.com/sns/v2/home).

- Choose **Create topic**.
- For Topic name, type *gsg-signup-notifications*. Choose **Create topic**.
- Choose **Create subscription**.
- For **Protocol**, choose *Email*. For **Endpoint**, enter *your email address*. Choose **Create Subscription**.

To confirm the subscription, Amazon SNS sends you an email named *AWS Notification — Subscription Confirmation*. Open the link in the email to confirm your subscription.

Do not forget that before testing the new functionality you need to have the SNS subscription approved.

<p align="center"><img src="./images/Lab05-2.png " alt="Lab05-2" title="Confirmed"/></p>

Add the *unique identifier* for the SNS topic to the configuration environment of your local deployment.

```bash
_$ export NEW_SIGNUP_TOPIC="arn:aws:sns:eu-west-1:YOUR-INSTANCE-NUMBER:gsg-signup-notifications"
```

Before you forget, you can also add a new variable to the environment of the Elastic Beanstalk deployment: go to the configuration of your web app, select *Software configuration* and click on the gear.

<p align="center"><img src="./images/Lab05-3.png " alt="Lab05-3" title="Configuration"/></p>

Scroll down the page and find the environment variables. Add a new variable named *NEW_SIGNUP_TOPIC* containing *arn:aws:sns:eu-west-1:YOUR-INSTANCE-NUMBER:gsg-signup-notifications*. Choose **Apply**.

<p align="center"><img src="./images/Lab05-4.png " alt="Lab05-4" title="Environment"/></p>


### Modify the web app to send messages

Open the files *form/models.py* and *form/views.py* read and understand what the code does.

Add the code below to *form/models.py* as a new operation of the model *Leads()*.

```python
def send_notification(self, email):
    sns = boto3.client('sns', region_name=AWS_REGION)
    try:
        sns.publish(
            TopicArn=NEW_SIGNUP_TOPIC,
            Message='New signup: %s' % email,
            Subject='New signup',
        )
        logger.error('SNS message sent.')

    except Exception as e:
        logger.error(
            'Error sending SNS message: ' + (e.fmt if hasattr(e, 'fmt') else '') + ','.join(e.args))
```

You have probably noticed that there is a Python variable that needs to be instantiated. Scroll up that file and add *NEW_SIGNUP_TOPIC* next to the other two environment variables, as shown below:

```python
STARTUP_SIGNUP_TABLE = os.environ['STARTUP_SIGNUP_TABLE']
AWS_REGION = os.environ['AWS_REGION']
NEW_SIGNUP_TOPIC = os.environ['NEW_SIGNUP_TOPIC']
```

Go to *form/views.py* and modify the signup view: if the lead has been correctly inserted in our DynamoDB table we can send the notification.

```python
def signup(request):
    leads = Leads()
    status = leads.insert_lead(request.POST['name'], request.POST['email'], request.POST['previewAccess'])
    if status == 200:
        leads.send_notification(request.POST['email'])
    return HttpResponse('', status=status)
```

Close the file and execute the Django web app locally.

You will see that the new record appears inserted but when you try to add a new lead appears an error *User: ... is not authorized to perform: SNS:Publish on resource*:

```bash
(eb-virt)_$ python manage.py runserver
"GET / HTTP/1.1" 200 7456
New item added to database.
Error sending SNS message: An error occurred (AuthorizationError) when calling the Publish operation:
  User: arn:aws:iam::YOUR-USER-NUMBER:root is not authorized to perform: SNS:Publish on resource: arn:aws:sns:eu-west-1:YOUR-INSTANCE-NUMBER:gsg-signup-notifications
"POST /signup HTTP/1.1" 200 0
```

To fix that error we need to grant access to the IAM profile that we created in the previous session. Go to [https://console.aws.amazon.com/iam](https://console.aws.amazon.com/iam) and check the JSON contents of the *gsm-signup-policy*: it only grants access to DynamoDB  to put an item:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "dynamodb:PutItem",
            "Resource": "*"
        }
    ]
}
```

We can change the Action property to a list where we grant the sns:Publish action to the list of allowed actions for *gsm-signup-policy*.

```json
"Action":["sns:Publish","dynamodb:PutItem"]
```
Instead of editing the JSON policy definition by hand you can use the visual editor and add checkmarks to the actions of each resource that you want your web app to use.

<p align="center"><img src="./images/Lab05-5.png " alt="Lab05-5" title="IAM"/></p>

Once you have stored the changes in the IAM policy, you can post a new record. This time you will see no error and you will receive a notification in your e-mail.

```bash
Existing item updated to database.
SNS message sent.
"POST /signup HTTP/1.1" 409 0
```

Now that the web app is working in your computer commit the changes. Deploy the new version to your EBS environment and test that it works correctly.

#### Questions
Has everything gone alright? Add your answers to the `README.md` file in the responses repository.

<a name="Tasks52" />

## Task 5.2: Create a new option to retrieve the list of leads

Edit the file *form/urls.py* to add the new URL and associate it to the new view *search*.

```python
urlpatterns = [
    # ex: /
    path('', views.home, name='home'),
    # ex: /signup
    path('signup', views.signup, name='signup'),
    # ex: /search
    path('search', views.search, name='search'),
]
```

To create the controller for the new view edit *form/views.py* and include the following code.

```python
from collections import Counter

def search(request):
    domain = request.GET.get('domain')
    preview = request.GET.get('preview')
    leads = Leads()
    items = leads.get_leads(domain, preview)
    if domain or preview:
        return render(request, 'search.html', {'items': items})
    else:
        domain_count = Counter()
        domain_count.update([item['email'].split('@')[1] for item in items])
        return render(request, 'search.html', {'domains': sorted(domain_count.items())})
```
The search view gets two parameters:
- preview: (*values are Yes/No*) lists the leads that are interested, or not, in a preview.
- domain: (*value is the part right after the @ of an e-mail address*) will list only the leads from that domain.

Reading the code, we understand that the search view retrieves the value of the parameters, gets the complete list of leads and then:

- if any parameter is set, the program just lists all the records matching the search.
- if both parameters are empty the program extracts the domain from each e-mail address and counts how many addresses belong to each domain.

To access the records stored at the NoSQL table *gsg-signup-table* you need to add a method *get_leads* to the model *Leads()* file *form/models.py*. The [Scan](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Scan.html) operation allows us to filter values from the table.

```python
def get_leads(self, domain, preview):
    try:
        dynamodb = boto3.resource('dynamodb', region_name=AWS_REGION)
        table = dynamodb.Table('gsg-signup-table')
    except Exception as e:
        logger.error(
            'Error connecting to database table: ' + (e.fmt if hasattr(e, 'fmt') else '') + ','.join(e.args))
        return None
    expression_attribute_values = {}
    FilterExpression = []
    if preview:
        expression_attribute_values[':p'] = preview
        FilterExpression.append('preview = :p')
    if domain:
        expression_attribute_values[':d'] = '@' + domain
        FilterExpression.append('contains(email, :d)')
    if expression_attribute_values and FilterExpression:
        response = table.scan(
            FilterExpression=' and '.join(FilterExpression),
            ExpressionAttributeValues=expression_attribute_values,
        )
    else:
        response = table.scan(
            ReturnConsumedCapacity='TOTAL',
        )
    if response['ResponseMetadata']['HTTPStatusCode'] == 200:
        return response['Items']
    logger.error('Unknown error retrieving items from database.')
    return None
```
A final step is to move the file *extra-file/search.html* to *form/templates/search.html*. That file receives the data from the view controller and creates the HTML to show the results.

Save the changes and, before committing them, check that everything works fine by typing *http://127.0.0.1:8000/search* in your browser.

<p align="center"><img src="./images/Lab05-6.png " alt="Lab05-6" title="Search"/></p>

To add the new option to the menu bar, simply edit the file *form/templates/generic.html*, go to line 28 and add the second navbar as shown below. Save the file and, with no further delay, check that you have it added in the version that runs in your computer.

```html
 <div class="collapse navbar-collapse" id="navbarResponsive">
    <ul class="navbar-nav">
        <li class="nav-item active"><a class="nav-link active" href="{% url 'form:home' %}">Home</a></li>
        <li class="nav-item"><a class="nav-link" href="#">About</a></li>
        <li class="nav-item"><a class="nav-link" href="#">Blog</a></li>
        <li class="nav-item"><a class="nav-link" href="#">Press</a></li>
    </ul>
    <ul class="nav navbar-nav ml-auto">
        <li class="nav-item"><a class="nav-link" href="{% url 'form:search' %}">Admin search</a></li>
    </ul>
</div>
```

<p align="center"><img src="./images/Lab05-7.png " alt="Lab05-7" title="Search"/></p>

If the web app works correctly in your computer commit the changes and deploy the new version on the cloud. Change whatever is necessary to make it work.

#### Questions
Has everything gone alright? What have you changed? Add your answers to the `README.md` file in the responses repository.

<a name="Tasks53" />

## Task 5.3: Improve the web app transfer of information (optional)
If you analyze the new function added, probably a good thing to do will be to optimize the transfer of information: imagine that instead of a few records in your NoSQL table you have millions of records. Transferring millions of records to your web app just to count how many e-mail addresses match a domain doesn't seem to be a great idea.

Change the above code to obtain the count of e-mail addresses for each domain directly from DynamoDB. Test the changes locally, commit them to your GitHub repository and make the web app work in the cloud.

Now, to save expenses, you can terminate your environment from the EBS console.

#### Questions
Do you think that you need to save the environment configuration? What has changed? Add your responses to `README.md`.

<a name="Tasks54" />

## Task 5.4: Deliver static content using a Content Delivery Network


### The static content in our web app

If you check line 9 of the file *form/templates/generic.html* you will see that, instead of loading in our server Bootstrap 4 CSS, we are already using a CDN to retrieve the CSS and send it to the final users. Bootstrap uses *maxcdn.bootstrapcdn.com* as their CDN distribution point.

```html
    <!-- https://www.bootstrapcdn.com/ -->
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" rel="stylesheet"
          integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
    <!-- Bootstrap Lumen theme -->
    <link href="https://maxcdn.bootstrapcdn.com/bootswatch/4.0.0-beta.3/lumen/bootstrap.min.css" rel="stylesheet"
          integrity="sha384-lBO0+E/aIJhpRIYjP6dJ1mNYgo3hhUBPcF74XRfOM27g7WmDuitolvnUENdDG4QI" crossorigin="anonymous">
    <!-- Bootstrap fonts -->
    <link href="https://maxcdn.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css" rel="stylesheet"
          integrity="sha384-wvfXpqpZZVQGK6TAh5PVlGOfQNHSoD2xbE+QkPxCAFlNEevoEH3Sl0sibVcOQVnN" crossorigin="anonymous">
```

We can now add our  CSS code to customize the look and feel of our web app even more. In that same file, add the following line just before the </head> HTML tag:

```html
    <link href="{% static 'custom.css' %}" rel="stylesheet">
</head>
```

If you check contents of the file *static/custom.css* you will see that it includes some images, also available in the same folder. If you save the modifications to *form/templates/generic.html* and review your web app, http://127.0.0.1:8000, you will see that it appears slightly different.

### Upload your static content to Amazon S3 and grant object permissions

All the distributed static content overloads our server with requests. Moving it to a CDN will reduce our server's load and, at the same time, the users will experience a much lower latency while using our web app. We only have static files in this app, but a typical web app distributes hundreds of pieces of static content.

To configure our CDN, we are going to follow the steps at ["Getting Started with CloudFront CDN"](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/GettingStarted.html). Check that document if you need extra details.

 Review the QuickStart hands-on [Getting Started in the Cloud with AWS](../../../Cloud-Computing-QuickStart/blob/master/Quick-Start-AWS.md) and create a new bucket in 'eu-west-1' region to deposit the web app static content. Let us name this bucket *eb-django-express-signup*.

 <p align="center"><img src="./images/Lab05-8.png " alt="Lab05-8" title="S3 bucket"/></p>

 This time add the files manually and grant them public read permission.

  <p align="center"><img src="./images/Lab05-9.png " alt="Lab05-9" title="S3 bucket"/></p>

You can also use AWS CLI to sync the contents of your static folder with that bucket. [Synchronize with your S3 bucket](https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html) using the following command:

```bash
_$ aws s3 sync --acl public-read ./static s3://eb-django-express-signup/static
upload: ./static/custom.css to s3://eb-django-express-signup/static/custom.css
upload: ./static/CCBDA-Square.png to s3://eb-django-express-signup/static/CCBDA-Square.png
upload: ./static/startup-bg.png to s3://eb-django-express-signup/static/startup-bg.png
```

If you explore in your S3 console you will see that there is a URL available to retrieve the files. Verify that you can access the contents of that URL, making the file public if it was not already.

```
https://s3-eu-west-1.amazonaws.com/eb-django-express-signup/static/CCBDA-Square.png
```


### Create a CloudFront CDN Web Distribution

Following the steps at ["Getting Started with CloudFront CDN"](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/GettingStarted.html) we end up having to wait until the files are distributed. It takes five minutes or more, be patient. Once the first distribution is set up, whenever you resync your static contents it will take much less.

 <p align="center"><img src="./images/Lab05-10.png " alt="Lab05-10" title="CloudFront CDN distribution"/></p>

### Change the code and test your links

The HTML code of our web app has only one direct access to a static file; the images referenced (using a relative route) through the CSS stylesheet. We just need to change *form/templates/generic.html* and our web app is now retrieving all static content from our CDN distribution.

Consider that we are now borrowing a CloudFront URL (RANDOM-ID-FROM-CLOUDFRONT.cloudfront.net) but usually, in the setup, we will use a URL from our domain, something like *static.mydomain.com* to map the CDN distribution.

```html
    <link href="http://RANDOM-ID-FROM-CLOUDFRONT.cloudfront.net/custom.css" rel="stylesheet">
```

#### Questions

Take a couple of screenshots of you S3 and CloudFront consoles to demonstrate that everything worked all right. Commit the changes on your web app, deploy them on EBS and check that it also works fine from there: use Google Chrome and check the origin of the files that you are loading:

 <p align="center"><img src="./images/Lab05-11.png " alt="Lab05-11" title="Files loaded"/></p>

 Add all these files to your repository and comment what you think is relevant in your session's *README.md*.


# How to Submit this Assignment:

Commit the `README.md` file to your responses repository and commit all changes to the web app repository.

Submit **before the deadline** to the *RACO Practicals section* a "Lab5.txt" file including:

1. Group number
2. Name and email of the members of this group
3. Address of the GitHub repository with your solution
4. Add any comment that you consider necessary