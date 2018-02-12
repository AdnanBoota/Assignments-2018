# Lab session #6: Interacting with users and services in the Cloud

* [Task 6.1: How to provide your services through a REST API?](#Tasks61)
* [Task 6.2: How to provide our service combined with third-party services?](#Tasks62)  
* [Task 6.3: Using Advanced Analytics as a Service in the Cloud](#Tasks64)  

<a name="Tasks61"/>

#  Tasks of Lab 6
<a name="Tasks61"/>

## Task 6.1:  How to provide your services through a REST API?
As we have previously seen, we can have plots in Python using libraries such as matplotlib.  However, how to provide our results through an API to consumers?

If you want to use python to build a prototype server (of your service) and you want to provide web-based API with minimal effort, you can take advantage of the web app that we have previously developed and deployed using AWS Elastic Beanstalk.

As a simplistic and quick implementation example, we are going to create a new view "chart" that will provide the service of visualizing how many e-mails we have gathered in our web app. Consequently, when a "client" invokes the URL *http://127.0.0.1:8000/chart*, we will provide them a chart with our results.

In your web app, you can have as many URLs as necessary. Such URLs receive parameters and produce results in different formats: Images, XML, JSON, etc.

If you are interested in building a "real REST server", there are many excellent Python frameworks for building a RESTful API: [Flask](http://flask.pocoo.org/), [Falcon](http://falconframework.org/), [Bottle](http://bottlepy.org/docs/dev/index.html) and even [Django REST framework](http://www.django-rest-framework.org/) that follows the Django framework that we are now using.

We have different options to create the visualization. However, we suggest using [D3.js](https://d3js.org) because it plays very well with web standards such as CSS and SVG, and allows anybody to create some fantastic interactive visualizations on the web. Many people who work in data science think that it is one of the coolest libraries for data visualization.

[D3.js](https://d3js.org) is, as the name suggests, based on Javascript. We will present a simple option to offer data D3 visualization with Python using the Python library [Vincent](https://github.com/wrobstory/vincent), that bridges the gap between a Python back-end and a front-end that supports D3.js visualization. Vincent takes our data in Python format and translates it into [Vega](https://github.com/vega/vega), a JSON-based visualization grammar that will be used on top of D3.

First, we need to install Vincent:

```
(eb-virt)_$ pip install vincent
```

Then, we add a new view at *form/urls.py*.

```python
urlpatterns = [
    # ex: /
    path('', views.home, name='home'),
    # ex: /signup
    path('signup', views.signup, name='signup'),
    # ex: /search
    path('search', views.search, name='search'),
    # ex: /chart
    path('chart', views.chart, name='chart'),
]
```

We will add the following lines at the end of the file *form/views.py* :

```python
import vincent
from django.conf import settings

BASE_DIR = getattr(settings, "BASE_DIR", None)


def chart(request):
    leads = Leads()
    items = leads.get_leads()
    domain_count = Counter()
    domain_count.update([item['email'].split('@')[1] for item in items])
    domain_freq = domain_count.most_common(15)
    labels, freq = zip(*domain_freq)
    data = {'data': freq, 'x': labels}
    bar = vincent.Bar(data, iter_idx='x')
    bar.to_json(os.path.join(BASE_DIR, 'static', 'domain_freq.json'))
    return render(request, 'chart.html', {'items': items})
```

At this point, the file `domain_freq.json` is written as *static/domain_freq.json* inside the project's directory. The JSON file will contain a description of the plot that can be handed over to D3.js and Vega.


To visualize the plot you can save at *form/templates/chart.html* a simple template (taken from [Vincent resources](https://github.com/wrobstory/vincent)).

```html
{% load static %}
<!DOCTYPE html>
<html>
<head>
    <title>Vega Scaffold</title>
    <script src="http://d3js.org/d3.v3.min.js" charset="utf-8"></script>
    <script src="http://d3js.org/topojson.v1.min.js"></script>
    <script src="http://d3js.org/d3.geo.projection.v0.min.js" charset="utf-8"></script>
    <script src="http://trifacta.github.com/vega/vega.js"></script>
</head>
<body>
<div id="vis"></div>
</body>
<script type="text/javascript">
    // parse a spec and create a visualization view
    function parse(spec) {
        vg.parse.spec(spec, function (chart) {
            chart({el: "#vis"}).update();
        });
    }
    parse('{% static 'domain_freq.json' %}');
</script>
</html>
```

Now you can open your browser at http://localhost:8000/chart and obtain access to the chart in SVG format.

<p align="center"><img src="./images/Lab06-1.png " alt="Lab06-1" title="Chart"/></p>

With the above procedure, we can plot many different types of charts using `Vincent`. [Explore Vincent by yourself](http://vincent.readthedocs.io/en/latest/).

Now you know how to deploy your services/web apps on the cloud using Elastic Beanstalk. Elastic Beanstalk, as you have already experimented, is an orchestration service that automatically builds your EC2, auto-scaling groups, ELB's, Cloudwatch metrics and S3 buckets so that you can focus on just deploying applications to AWS and not worry about infrastructure tasks.

There is also an AWS service called [AWS Lambda](https://aws.amazon.com/lambda/) that, as well as EBS, allows you to not deal with over/under capacity, deployments, scaling and fault tolerance, OS or language updates, metrics, and logging. AWS Lambda is considered a **Function as a Service**, whereby you create a piece of code and AWS will execute that portion of code, and only that piece of code, as many times as you tell it to.

AWS Lambda introduced a new serverless world. AWS community started working on a complete solution to develop AWS-powered **microservices** or backends for the web, mobile, and IoT applications. The migration required facilitation because of the building-block nature of AWS Lambda and its complex symbiosis with Amazon API Gateway. One example is [Serverless Framework](https://serverless.com) that allows building auto-scaling, pay-per-execution, event-driven apps on AWS Lambda. **(optional project)**

To sum up: AWS Lambda runs your code while Elastic Beanstalk runs your applications.

#### Questions

Do you think that the file  *domain_freq.json* written as static content is the best way to distribute it? Why? What will you change in the above example? How will the changes look like? Write your answers in the `README.md` file for this session.

<a name="Tasks62"/>

## Task 6.2: How to provide our service combined with third-party services?

To augment the value of our service, we can build it upon other services. As an example of combining our service with third-party services, I suggest to plotting tweets on a map. For this purpose, we will use [GeoJSON](http://geojson.org), a format for encoding a variety of geographic data structures and [Leaflet.js](http://leafletjs.com), a Javascript library for interactive maps.

**GeoJSON** supports a variety of geometric types of formats that can be used to visualize the desired shapes onto a map. A GeoJSON object can represent a geometry, feature, or collection of features. Geometries only contain the information about the shape; its examples include Point, LineString, Polygon, and more complex shapes. Features extend this concept as they contain a geometry plus additional (custom) properties. Finally, a collection of features is just a list of features. A GeoJSON data structure is always a JSON object. The following snippet shows an example (taken from https://github.com/bonzanini/Book-SocialMediaMiningPython) of GeoJSON that represents a collection with two different points, each point used to pin a particular city:

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [
          -0.12
          51.5
        ]
      },
      "properties": {
        "name": "London"
      }
    },
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [
          -74,
          40.71
        ]
      },
      "properties": {
        "name": "New York City"
      }
    }
  ]
}
```

In this GeoJSON instance, the first key is the `type` of the represented object. This field is
mandatory and its value must be one of the following:
* `Point`: This is used to represent a single position
* `MultiPoint`: This represents multiple positions
* `LineString`: This specifies a string of lines that go through two or more positions
* `MultiLineString`: This is equivalent to multiple strings of lines
* `Polygon`: This represents a closed string of lines, that is, the first and the last positions are the same
* `GeometryCollection`: This is a list of different geometries
* `Feature`: This is one of the preceding items (excluding GeometryCollection) with additional custom properties
* `FeatureCollection`: This is used to represent a list of features

Given that `type` in the preceding example has the `FeatureCollection` value, we will expect the `features` field to be a list of objects (each of which is a `Feature`).

The two features shown in the example are simple points, so in both cases, the
`coordinates` field is an array of two elements: longitude and latitude. This field also
allows for a third element to be there, representing altitude (when omitted, the altitude is
assumed to be zero).

For our example, we just need the smallest structure: a `Point` identified by its coordinates (latitude and longitude). To generate this GeoJSON data structure we only need to iterate all the tweets looking for the coordinates field.

Twitter allows its users to provide their geolocation when they publish a tweet, in the form of latitude and longitude coordinates. With this information, we are ready to create some friendly visualization for our data as interactive maps. Unfortunately, the details about the geographic localization of the user's device are available in only in a small portion of tweets: many users disable this functionality on their mobile.

Let us create a listener program, *TwitterListener.py*, based on the code we made in previous lab sessions. We are going to execute the following code and let it listen to tweets. If the tweet contains geo localization we save it in our new NoSQL table `twitter-geo`.

```python
from tweepy import OAuthHandler
from tweepy import Stream
from tweepy.streaming import StreamListener
import json
import sys
import boto3

GEO_TABLE = 'twitter-geo'
AWS_REGION = 'eu-west-1'

class MyListener(StreamListener):

    def __init__(self):
        dynamodb = boto3.resource('dynamodb', region_name=AWS_REGION)
        try:
            self.table = dynamodb.Table(GEO_TABLE)
        except Exception as e:
            print('\nError connecting to database table: ' + (e.fmt if hasattr(e, 'fmt') else '') + ','.join(e.args))
            sys.exit(-1)

    def on_data(self, data):
        tweet = json.loads(data)
        if not tweet['coordinates']:
            sys.stdout.write('.')
            sys.stdout.flush()
            return True
        try:
            response = self.table.put_item(
                Item={
                    'id': tweet['id_str'],
                    'c0': str(tweet['coordinates']['coordinates'][0]),
                    'c1': str(tweet['coordinates']['coordinates'][1]),
                    'text': tweet['text'],
                    "created_at": tweet['created_at'],
                }
            )
        except Exception as e:
            print('\nError adding item to database: ' + (e.fmt if hasattr(e, 'fmt') else '') + ','.join(e.args))
        else:
            status = response['ResponseMetadata']['HTTPStatusCode']
            if status == 200:
                sys.stdout.write('x')
                sys.stdout.flush()

    def on_error(self, status):
        print('status:%d' % status)
        return True

consumer_key = 'YOUR-CONSUMER-KEY'
consumer_secret = 'YOUR-CONSUMER-SECRET'
access_token = 'YOUR-ACCESS-TOKEN'
access_secret = 'YOUR-ACCESS-SECRET'

auth = OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_secret)

twitter_stream = Stream(auth, MyListener())
twitter_stream.filter(locations=[-2.5756, 39.0147, 5.5982, 43.957])
```

In the above program have used a different way of filtering tweets: a geo bonding box. You can [select your favorite geo bounding box](http://boundingbox.klokantech.com/) using a web app made by [Klokan Technologies](https://www.klokantech.com/): choose CSV format, draw your bounding box, copy the coordinates and start listening.

<p align="center"><img src="./images/Lab6-BoundingBox.png" alt="Bounding box for Barcelona Area" title="Bounding box for Barcelona Area"/></p>

Going back to our dear Django web app, we will add a new view 'map'.

```python
urlpatterns = [
...
    # ex: /map
    path('map', views.map, name='map'),
]
```

The controller for the new view uses *get_tweets()* to scan *'twitter-geo'* and *map()* builds the GeoJSON file to plot the tweets. Edit the file *form/views.py* and paste the following Python code at the end.

```python
import json

def map(request):
    geo_data = {
        "type": "FeatureCollection",
        "features": []
    }
    tweets = Tweets()
    for tweet in tweets.get_tweets():
        geo_json_feature = {
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [tweet['c0'], tweet['c1']]
            },
            "properties": {
                "text": tweet['text'],
                "created_at": tweet['created_at']
            }
        }
        geo_data['features'].append(geo_json_feature)

    with open(os.path.join(BASE_DIR, 'static', 'geo_data.json'), 'w') as fout:
        fout.write(json.dumps(geo_data, indent=4))
    return render(request, 'map.html')
```

At the *form/models.py* file we will add a new model with the function *get_tweets*. Create this model based on the code for the model Leads().

```python
class Tweets(models.Model):

    def get_tweets(self):
        #
        # Add your code here to return the 'tweets' stored in your new dynamoDB table
        #
        logger.error('Unknown error retrieving tweets from database.')
        return None
```

Now, using the Leaflet.js Javascript library for interactive maps, we can create our `.html` view containing the map. Copy the contents of this file at *forms/templates/map.html*

```html
{% load static %}
<!DOCTYPE html>
<html>
<head>
    <title>Quick Start - Leaflet</title>
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="shortcut icon" type="image/x-icon" href="docs/images/favicon.ico"/>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.0.3/dist/leaflet.css"/>
    <script src="https://unpkg.com/leaflet@1.0.3/dist/leaflet.js"></script>
    <script src="http://code.jquery.com/jquery-2.1.0.min.js"></script>
    <style>
        #map {
            height: 600px;
        }
    </style>
</head>
<body>
<!-- this goes in the <body> -->
<div id="map"></div>
<script>
    // Load the tile images from OpenStreetMap
    var mytiles = L.tileLayer('http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
        attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
    });
    // Initialise an empty map
    var map = L.map('map');
    // Read the GeoJSON data with jQuery, and create a circleMarker element for each tweet
    $.getJSON("{%  static 'geo_data.json' %}", function (data) {
        var myStyle = {
            radius: 2,
            fillColor: "red",
            color: "red",
            weight: 1,
            opacity: 1,
            fillOpacity: 1
        };
        var geojson = L.geoJson(data, {
            pointToLayer: function (feature, latlng) {
                return L.circleMarker(latlng, myStyle);
            }
        });
        geojson.addTo(map)
    });
    map.addLayer(mytiles).setView([40.5, 5.0], 6);
</script>
</body>
</html>
```
Just execute the web app locally http://127.0.0.1:8000/map, and you will see something like:

<p align="center"><img src="./images/Lab6map1.png" alt="Tweets in Barcelona City" title="Tweets in Barcelona City"/></p>

<p align="center"><img src="./images/Lab6map2.png" alt="Tweets in Europe" title="Tweets in Europe"/></p>

#### Questions

Now we are showing all the collected tweets on the map. Can you think of a way of restricting the tweets plotted using some constraints? For instance, the user could invoke http://127.0.0.1:8000/map?from=2018-02-01-05-20&to=2018-02-03-00-00. Implement that functionality or any other functionality that you think it could be interesting for the users. Change the code to implement the new feature and explain what you have done and show the results in the *README.md* file for this lab session.

Do you think that the file *geo_data.json* written as static content is the best way to distribute it? Why? Write your answers in the `README.md` file for this session.

<a name="Tasks63"/>

## Task 6.3: Advanced Analytics as a Service in the Cloud (optional task)

In this section, we are going to experiment the power of the Cloud Vision API from Google to detect text within images as an example of Advanced Analytics as a Service in the Cloud.
This section follows the official [introduction to image classification]( https://cloud.google.com/vision/docs/detecting-labels).


This hands-on helps you to classify images using labels. [Cloud Vision API](https://cloud.google.com/vision/) enables developers to understand the content of an image by encapsulating powerful machine learning models in an easy to use REST API. It quickly classifies images into thousands of categories (e.g., "sailboat", "lion", "Eiffel Tower"), detects individual objects and faces within images and finds and reads printed words contained within images. You can build metadata on your image catalog, moderate offensive content, or enable new marketing scenarios through image sentiment analysis. Analyze images uploaded in the request or integrate with your image storage on Google Cloud Storage. Try the API [(Drag image file here or Browse from your computer)](https://cloud.google.com/vision/). Locate the area shown below and test the functionality offered:

<p align="center"><img src="./images/Lab06-GoogleVision.png" alt="GoogleVision" title="GoogleVision"/></p>

### 6.3.1 Cloud Platform sign up
The first step is to [sign up]( https://cloud.google.com/vision/). It exists a free trial for this service, to fully enjoy the benefits of Google’s Cloud platform the best option is to get a business trial, with $300 worth of free credit for this year. That amount is far more than enough for the expected needs on this subject’s scope and will give you time enough to explore other features.

Once the registration process is finished, we need to access the [management console]( https://console.cloud.google.com/home), where it is possible to create a new project, choosing a name and an ID for it. It is necessary to enable billing for this project to continue, but as long as we are spending from the free credit we have there is nothing to worry about.

The project creation process takes a few seconds, after finishing it will appear in our list. It is time to select the new project and enable the Cloud Vision API that is the tool we will use during this hands-on. Finally, we need to set up some credentials that will be the way we authenticate to enable the communication: This can be done selecting “Credentials” in the left menu and simply clicking over the blue button saying “Create credentials” and choosing the “service account key” option.  Select JSON as your key type. Once completed, your service account key is downloaded to your browser's default location.

### 6.3.2 Environment setup
The base language is Python and those packages and almost every needed module will be already installed. The scripts and demo files needed can be downloaded from the official Google Cloud Platform GitHub repository, and only a few more changes will be needed to be ready.

#### Download the Example Code
Download the code from this repository. You can do this by executing:

```bash
_$ git clone https://github.com/CCBDA-UPC/google-cloud-vision-example.git cloud-vision
```

This will download the repository of samples into the directory
`cloud-vision`.

This example has been tested with Python 2.7 and 3.4.

#### Set Up to Authenticate With Your Project's Credentials

Next, set up to authenticate with the Cloud Vision API using your project's
service account credentials. E.g., to authenticate locally, set
the `GOOGLE_APPLICATION_CREDENTIALS` environment variable to point to your
downloaded service account credentials before running this example:

```export GOOGLE_APPLICATION_CREDENTIALS=/path/to/your/credentials-key.json```

If you do not do this, you will see an error that looks something like this when
you run the script:
`HttpError 403 when requesting
https://vision.googleapis.com/v1/images:annotate?alt=json returned
"Project has not activated the vision.googleapis.com API. Please enable the API
for project ..."`


### 6.3.3 Quick Start: Running the Example

<p align="center"><img src="./images/Lab06-Tweet-MN.png" alt="Tweet" title="Tweet"/></p>


If you'd like to get the example up and running before we dive into the
details, here are the steps to analyze the image of this [tweet](https://twitter.com/angeltoribio/status/963048129569443841).

1. Enable your virtualenv and go to the repository folder:
```bash
$ source <path-to-virtualenv>/bin/activate
$ cd google-cloud-vision-example
```

2. Install the requirements:
```bash
$ pip install -r requirements.txt
```

2. Run the `label.py` passing the downloaded tweet image as argument:
```bash
$ python label.py <path-to-image>
```

3. Wait for the script to finish. It will show 5 possible classfications for your image:
```bash
Results for image Marenostrum.jpg:
electricity - 0.770
computer cluster - 0.745
display device - 0.735
electrical supply - 0.671
electrical wiring - 0.667
```


# How to Submit this Assignment:

Go to your responses repository, commit and push:
- the `README.md` file with your answers, including the results of the optional task 6.3
- the screenshots of your maps for task 6.2

Go to your **private** web app repository and commit the changes that you have made to implement task 6.2.

Submit **before the deadline** to the *RACO Practicals section* a "Lab6.txt" file including:

1. Group number
2. Name and email of the members of this group
3. Address of the GitHub repository with your solution
4. Add any comment that you consider necessary
