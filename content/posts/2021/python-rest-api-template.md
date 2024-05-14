---
slug: 'Python Rest Api Template'
title: 'A simple Python REST API-Template on Azure App Services'
date: 2021-05-30T15:36:10+02:00
draft: false
#author: ["mightyjens"]
categories: ["coding"]
tags: ["Coding","Python"]
series: [""]
categories_weight: 10
comments: true
showToc: true
TocOpen: false
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
UseHugoToc: true
editPost:
    URL: "https://github.com/mightyjens/mightyjens.github.io/tree/main/content"
    Text: "Suggest Changes" 
    appendFilePath: true  
---

In the last few days, I finally found the time to get in touch with Python a bit. The programming language is known to be very widespread and offers almost infinite possibilities besides PowerShell-style automation, data science and web development…. In the area of Business Central, some people might know it in the context of Docker.

But I always find it difficult to learn things according to instructions or literature. The best way for me is to take a concrete problem and try to solve it. In this case, I did learn some of the absolute basics from the book *“Python Crash Course – A Hands on”*, but then switched to solving my defined problem… a Python REST API-Template.

So, here is my approach to learning a little bit of python. Feel free to comment on it as I am absolutely sure it‘s far away from being optimal 🙂

> You can find the complete Python REST API-Template on my [GitHub Page](https://github.com/mightyjens/python-flask-rest-api).

## The Problem
In the course of my work and the evolution of Business Central, especially in the cloud, microservices, among other technologies, are becoming increasingly important… i.e. the encapsulation of specific (or unsolvable in AL) tasks into APIs, which can then be consumed via REST calls. I have already written a couple of productively used microservices for our Addons based on Azure Functions in C# as well as Node.JS. However, the elegance and readability of Python has always appealed to me.

## The Goal
My goal for the learning session here was to create a reusable template for a REST API, which I can “fill with life” at will afterwards by extending it with the concrete functions. It should then be running as an Azure App Service, at best with an automatic build-pipeline. The template should have the following properties and requirements:

- Build on the versatile and easy-to-use Flask Framework
- Basic Authentication using Azure Cosmos DB (MongoDB)
- API versioning using Flask Blueprints
- Settings saved in env variables

## Implementing the Requirements
### Settings

An uncomplicated way to store your settings data is using environmental variables. In your Visual Studio Code environment, you simply create an .env file and add your settings to is. Afterwards, you’re able to access them wherever you want using the “os.environ.get(‘TEST_ENV_VAR’)” method:

```
TEST_ENV_VAR = local

ENCRYPTION_HASH_PREFIX = 'pbkdf2:sha256:260000$'

API_USER_TEST = 'hQyv2OGDNGrhucyV$4ee08644c70e5e974e50655abfaa3d0d18f0cf46b27223af9811df1ba25efec8'

COSMOS_ACCOUNT_HOST = ''
COSMOS_ACCOUNT_USER = ''
COSMOS_ACCOUNT_KEY = ''
COSMOS_DATABASE_NAME = ''
COSMOS_COLLECTION_NAME = ''
```

And the most charming thing with these environmental variables is, that we could access the Azure App Service settings the same way without changing anything:

![Python Rest API](https://www.j3ns.de/wp-content/uploads/2021/05/Azure-App-Settings-1024x239.png)

### API versioning using Blueprints
Although I know it’s usually not a best practise in API development, I wanted to implement versioning in the template. I want to be able to serve i.e. newer or breaking changes through new versions. Fortunately, Flask gives you the Blueprint approach. With these blueprints, you could register your different versions to your routes.

To achieve this, the structure of your project should be setup properly. Simplified, my project looks like this:

- api
  - common (used by all versions)
    - functions.py
    - authy.py
    - mongo.py
  - v1 (blueprint & version specific)
    - functions.py
    - routes.py
  - v2-n (blueprint & version specific)
    - functions.py
    - routes.py
  - app.py (register blueprints and initiate flask)

First, we create the blueprint for every version needed. Here we could now implement all our functionality we need in the API:

```python
# routes.py

from common.authy import auth
from flask import Blueprint

# Create a blueprint
api = Blueprint('api_v1', __name__)

# Implement your methods
@api.route('/mymethod', methods=["GET"])
def mymethod():
    return "Hello World"
```

After that, in our app.py, we import all blueprints/versions and register them with a URL prefix:

```python
# app.py

from flask import Flask

# Define routes
# https://realpython.com/flask-blueprint/#what-a-flask-application-looks-like

# Import the blueprint from n-versions
from v1.routes import api as api_v1
from v2.routes import api as api_v2

app = Flask(__name__)

# Now register the blueprint with url prefix
app.register_blueprint(api_v1, url_prefix='/v1')
app.register_blueprint(api_v2, url_prefix='/v2')
```

Access to the API is then possible through the defined URLs:

- https://myapi.azurewebsites.net/v1/mymethod
- https://myapi.azurewebsites.net/v2/mymethod

### Basic Authentication
For our goal to use basic authentication with our API we use the build-in HTTPBasicAuth() class. And within this implementation we override verify_password() with our own procedures. As we would like to get the credentials from an Azure Cosmos DB, we fetch it from there and check it against the given, hashed credentials.

```python 
# authy.py 

import os, common.mongo, common.functions
from flask_httpauth import HTTPBasicAuth
from werkzeug.security import generate_password_hash, check_password_hash

auth = HTTPBasicAuth()

@auth.verify_password
def verify_password(username, password) -> str:
    # Check username is filled
    if not username:
        return None
    
    # Check for environmental logins
    passwordResponse = common.functions.getValueFromEnv(username)

    if not passwordResponse:
        # Fetch data from cosmos db
        passwordResponse = common.mongo.getUserFromDatabase(username)
        if not passwordResponse:
            return None

    encryptionHashPrefix = os.environ["ENCRYPTION_HASH_PREFIX"]

    if check_password_hash(f"{encryptionHashPrefix}{passwordResponse}", password):
        return username
    else:
        return None
```

Routes to be authenticated could then be easily protected with one line of code.
The methods that require authentication can be decorated with the login_required decorator:

```python
# routes.py

from common.authy import auth
from flask import Blueprint

@api.route('/', methods=["GET"])
@auth.login_required
def index():
    return "Hello World"
```

### Accessing Azure Cosmos DB
As we already mentioned in the authentication part, we use an Azure Cosmos DB for storing data… in this case our users’ credentials. I didn’t really get along with Microsoft’s azure-cosmos library. So I use the pymongo implementation for that I found some nice examples:

```python
# mongo.py

import os
from pymongo import MongoClient

def getUserFromDatabase(username):

    cosmos_user_name = os.environ["COSMOS_ACCOUNT_USER"]
    cosmos_password = os.environ["COSMOS_ACCOUNT_KEY"]
    cosmos_url = os.environ["COSMOS_ACCOUNT_HOST"]
    cosmos_database_name = os.environ["COSMOS_DATABASE_NAME"]
    cosmos_collection_name = os.environ["COSMOS_COLLECTION_NAME"]

    # Create Mongo client
    mongoUri = f'mongodb://{cosmos_user_name}:{cosmos_password}@{cosmos_url}'
    mongoClient = MongoClient(mongoUri)
    
    # This is the name of the Mongo compatible database
    mongoDatabase = mongoClient[cosmos_database_name]
    
    # The name of the collection you are querying
    mongoCollection = mongoDatabase[cosmos_collection_name]
    mongoDocument = mongoCollection.find_one({ "username": username})
    
    # Get password from the collection
    
    if mongoDocument:
        return mongoDocument['password']
    else: 
        return None
```

## Deployment of the Python REST API-Template
For this example, I looked at the deployment functions in Azure App Services. Here you have the option of choosing a GitHub repository and a branch. As soon as the branch is updated, the app service automatically updates itself from this repository:

![Python Rest API](https://www.j3ns.de/wp-content/uploads/2021/05/2021-05-30-12_17_37-mightysftp-Microsoft-Azure-und-3-weitere-Seiten-Personlich-%E2%80%93-Microsoft%E2%80%8B-Edge.png)

In your GitHub repo’s actions, you could now watch the build pipeline run. Crazy simple and intuitive:


![Python Rest API](https://www.j3ns.de/wp-content/uploads/2021/05/Build-Pipeline-1-1024x295.png)

![Python Rest API](https://www.j3ns.de/wp-content/uploads/2021/05/Build-Pipeline-2-1024x437.png)

I’m really looking forward for the first productive requirements where I could use it. I find it far more readable and intuitive than, for example, Node.JS. However, one must be aware that the latter is more suitable for certain requirements due to its speed and performance.

You can find the complete Python REST API-Template on my [GitHub Page](https://github.com/mightyjens/python-flask-rest-api).