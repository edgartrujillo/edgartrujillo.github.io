## Edgar Trujillo's ePortfolio

**How can I develop this idea?**

**How can I break this API?**

**What more can I add?** 

**How can I secure this better?**


These are some of the questions that I have begun asking myself more frequently ever since I started pursuing a degree in Computer Science. I have always loved a good challenge and using my creativeness to solve a problem in a unique way. While pursuing my degree, not only have I showcased my development strengths, but I have also demonstrated how quickly I can turn a weakness into a strength. Desiring a career in the Computer Science field usually means you will hold a position that requires you to know how to develop error-free code and perhaps use multiple different languages. I have since put into practice the belief that anyone can quickly learn a new programing language, because the only differentiating factor is usually the code syntax; The underlying algorithms, concepts of variables, data structures, and functions is still the same. Therefore, I suggest people should develop a better understanding on the Foundations of Computer Science before writing code. Having a good understanding of the foundations of Computer Science can transform anyone into a collaborative team member, who can communicate to stakeholders, assist in the implementation of data structures and algorithms, and assist in the design of software and databases. 


Throughout my program, I have not only gained a strong knowledge in the foundations of Computer Science, but I have implemented and developed applications that demonstrate my ability to think long term since I always ensure the code I write is well commented, reusable, and understandable incase someone else decides to expand on it. 


For my capstone project, I was tasked with showcasing my skills and abilities in three key categories: Software design and engineering, Algorithms and data structure, and Databases. I decided to develop a project that showed the iteration and progression from each key category based on the final project from a previous class, CS-340 Advanced Programing. This final project was a RESTful API written in Python that provided an interface to a MongoDB containing detailed information on various stocks. Firstly, I decided to refactor the code from Python to Node.js to expand its usability. Second, after the code was refactored, I iterated the API by adding an authorization and authentication feature to ensure only approved users could utilize the API and thus increased the security of it. Lastly, I decided to expand on the API capabilities by adding more API methods and endpoints. 

### Original Python API 

```python
import json
from json import dumps
from bson import json_util
from pymongo import MongoClient
import sys
import bottle
from bottle import route, run, request, abort, post, get, put, delete ,response

connection = MongoClient('localhost', 27017)
db = connection['market']
collection = db['stocks']

# Beginnning of Support functions
def getMovingAverageCount(min, max):
  result = collection.find({ "50-Day Simple Moving Average":
    {
      '$gte': min ,
      '$lte': max
    }
  }).count()
  return result

def industryMatch(string):
  result = collection.find(
  {
    "Industry" : string
  }  ).limit(5)
  return result 

def getSector(string):
  pipline = [
    { '$match': { "Sector": string}},
    { '$group': { "_id" : "$Industry" , "Shares Outstanding" :  {"$sum": "$Shares Outstanding"}} }
  ]
  result = collection.aggregate(pipline)
  return result 
  
def delete_document(key, value):
  result = collection.delete_one({key:value})
  if not result:
    abort(404, 'No document with %s : %s' % key,value) 
  return result

def update_document(key, value,document):
  result = collection.update({key:value},{ '$set': document }, upsert=False,multi=False) 
  if not result:
    abort(404, 'No document with %s : %s' % key,value)
  return json.loads(json.dumps(result, indent=4, default=json_util.default))

def get_document(key, value):
  document = collection.find_one({key:value})
  if not document:
    abort(404, 'No document with %s : %s' % key,value)
  return document

def insert_document(document):
  try:
    result = collection.save(document)
  except (ValidationError) as ve:
    abort(400, str(ve))
#  return result
# End of support functions

# Begin of API def


@put('/stocks/api/v1.0/updateStock/<sym>')
def update_stock(sym):
  try:
    result = request.json
    update_document("Ticker", sym, result)
  except error:
    abort(404,'Error')   
  print "Complete"

@post('/stocks/api/v1.0/createStock/<sym>')
def post_stock(sym):
    try:
      result = request.json
      data = insert_document(result)
      print(data)
    except error:
      abort(404,'Error')
#    return result

@post('/stocks/api/v1.0/stockReport')
def post_stockReport():
    list = []
    try:
      result = request.body
      for i in result:
        if len(i) > 0:
          string = i
      print(string)
      string = string.replace('[',"")
      string = string.replace(']',"")
      symbols = string.split(',')
      for i in symbols:
        print(i)
        list.append(get_document("Ticker",i))
        
    except error:
      abort(404,'Error')
    return str(list)

@delete('/stocks/api/v1.0/deleteStock/<sym>')
def delete_stock(sym):
  try:  
    result = delete_document("Ticker", sym)
  except NameError:
    abort(404, 'No parameter')
  return bottle.HTTPResponse(status=200)


@route('/stocks/api/v1.0/getStock/<sym>')
def get_stock(sym):  
    try:
      string = get_document("Ticker", sym)
    except NameError:
        abort(404, 'No parameter')
    return json.loads(json.dumps(string, indent=4, default=json_util.default))



@route('/stocks/api/v1.0/portfolio/<sym>')
def get_portfolio(sym):  
    list = []
    try:
      result = collection.find( {'$text': {'$search':sym }})
      for i in result:
        list.append(i)
    except NameError:
        abort(404, 'No parameter')
    return str(list)#json.loads(json.dumps(string, indent=4, default=json_util.default))
  
@route('/stocks/api/v1.0/industryReport/<sym>')
def get_industryReport(sym):  
    list = []
    try:
      result = industryMatch(sym)
      for i in result:
        list.append(i)
    except NameError:
        abort(404, 'No parameter')
    return str(list)#json.loads(json.dumps(string, indent=4, default=json_util.default))
    
  
if __name__ == '__main__': #declare instance of request
    #app.run(debug=True)
    run(host='localhost', port=8080)
```

### Code Review on Initial Python Code 
![Code Review Powerpoint](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Screenshot_10.png)
You can find the powerpoint presentation at [Code Review Powerpoint](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Code%20Reviews.pptx).



### Preparing 
Before beginning development on any of the enhancements, I first needed to get familiar with Node.js
I decided to follow the recommeneded approach of utilizing the **MVC model**
```
M - Models
V - Views
C - Controllers 

I also created a different folder that defined the routes
stocks.routes.js - shows the possible routes the API handles
app.js - main server file 
```

### Running API and Enhancements

> Be sure to have a MongoDB sever installed locally 

> I have included the stocks.json as well 

> Import the stocks.json into the db
> with Mongoimport 
> Use the command line in the directory containing stocks.json 
```
mongoimport --db market --collection stocks --file stocks.json 
```

> An index will be needed on the text as well 
> in Mongo shell after json is imported

> In Mongo Shell
``` 
use market
db.stocks.createIndex( { Company: "text" } )
``` 
> End Mongo Shell 

> Afterwards to run API
```
node app.js
```




### Enhancement One
#### Refactoring the Python code into JavaScript and using Node.js as the backend  
You can find the code at [Enhancement One Bitbucket](https://bitbucket.org/edgarr_t/enhancementone/src/master/).

![Enhancement One Routes](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Screenshot_7.png "Enhancement One Routes")


In cases where image doesn't load [Enhancement One Routes](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Screenshot_7.png "Enhancement One Routes")




![Creating a Stock](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Screenshot_9.png "Creating a Stock")




In cases where image doesn't load [Creating a Stock](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Screenshot_9.png "Creating a Stock")



The artifact I decided to use for my first enhancement was a RESTful API I wrote in Python for my CS-340 class and it was our final project. The purpose of the API was to quickly get stock information from a Mongo Database. The API provided a simple interface to accomplish this and allows a developer to integrate this database in a program of their choice since they can easily use the REST API. For this enhancement I decided to rewrite the python API to work with Node.js as well. I was really interested in this artifact since I first started working on it for my CS-340 class. In the past few years, the popularity of RESTful APIs has gone through the roof. Companies are now breaking up their monolithic programs into different microservices to increase efficiency and speed. Python and Node.js are popular languages that are used to accomplish this and since I had already created the API in Python, I wanted to challenge myself and accomplish the same with the JavaScript language. I think this will be beneficial for others as well because if we considered this project opened to others, then having both a Python and Node.js API available to developers will ensure they can keep the same language throughout their full project instead of having a piece written in Python. I believe this demonstrates an ability to use well-founded and innovative techniques, skills, and tools in computing practices for the purpose of implementing computer solutions that deliver value and accomplish industry-specific goals. Perhaps the most challenging part while working on this enhancement was my constant mix up of wanting to write the application with a Python syntax style when the JavaScript syntax is very different. I also learned how to set up a simple server in node.js that listened for API calls and then correctly responded. I definitely learned that I enjoyed working with Node.js as a backend language as well.          
         




### Enhancement Two
#### Securing the API
You can find the code at [Enhancement Two Bitbucket](https://bitbucket.org/edgarr_t/enhancementtwo/src/master/).

![Register for a Token](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Screenshot_3.png "Register for a Token")


In cases where image doesn't load [Register for a Token](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Screenshot_3.png "Register for a Token")



![No Token](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Screenshot_4.png "No Token")


In cases where image doesn't load [No Token](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Screenshot_4.png "No Token")



![Correct Token Supplied](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Screenshot_5.png "Correct Token Supplied")


In cases where image doesn't load [Correct Token Supplied](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Screenshot_5.png "Correct Token Supplied")



The artifact I decided to use for my second enhancement was to continue improving and iterating on my enhancement one Node.js RESTful API I wrote, but now with a focus on improving the security of the whole application. Again, as mentioned before this RESTful API was originally written in Python for my CS-340 class and it was our final project. The purpose of the API was to quickly get stock information from a Mongo Database. The API provided a simple interface to accomplish this and allows a developer to integrate this database in a program of their choice since they can easily use the REST API. For this enhancement, I decided to add a form of authorization and authentication to the API to ensure that only people authorized could use the API. 
I felt this artifact was important to include in my ePortfolio because often times many developers fail to consider security when developing new apps, programs, or applications. Very often our first ideas and effort are spent on how to develop this new program to fulfill our needs and then only after are we close to done do we consider how to secure it. I think moving forward in the future, developers should start any development with security as a foundational piece. This will not only make the application more secure but will help developers think differently in a positive way. I think this artifact demonstrates the course objective of designing and evaluating computing solutions that solve a given problem using algorithmic principles and computer science practices and standards appropriate to its solution while managing the trade-offs involved in design choices. The added security also shows the use of a security mindset that anticipates adversarial exploits in software architecture and designs to expose potential vulnerabilities, mitigate design flaws, and ensure privacy and enhanced security of data and resources. Perhaps, the hardest part of this artifact was getting the authorization and authentication to work together correctly. In a previous internship as an IOT engineer intern, we often worked with edge devices through some interface and used the concept of Bearer Tokens to establish authorization. I decided to follow the same principle in this artifact but didn’t anticipate the difficulty of getting it to work together with my API that was already written. This is also why, moving forward I will consider the implementation of security before developing any code and this will save development time and efforts.



### Enhancement Three
#### Expanding API Routes
You can find the code at [Enhancement Three Bitbucket](https://bitbucket.org/edgarr_t/enhancementthree/src/master/).


![Enhancement Three Routes](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Screenshot_8.png "Enhancement Three Routes")




In cases where image doesn't load [Enhancement Three Routes](https://github.com/edgartrujillo/edgartrujillo.github.io/blob/master/Screenshot_8.png "Enhancement Three Routes")





Following the iteration process, for my third enhancement I decided to continue improving and iterating on my enhancement one Node.js RESTful API I wrote, but now focusing on expanding the API’s capabilities through data mining and adding more advanced concepts of MySQL. Again, as mentioned before this RESTful API was originally written in Python for my CS-340 class and it was our final project. The purpose of the API was to quickly get stock information from a Mongo Database. The API provided a simple interface to accomplish this and allows a developer to integrate this database in a program of their choice since they can easily use the REST API. 
I included this artifact because it shows the progression this REST API has had through the different enhancements and from where it first was before any improvements were made. It also was an opportunity to expand on the API’s capabilities. For my final project, we had some predefined routes and results desired, but now with no restrictions, I had an opportunity to create new routes (endpoints) that accomplished different tasks. I believe this will demonstrate an ability to use well-founded and innovative techniques, skills, and tools in computing practices for the purpose of implementing computer solutions that deliver value and accomplish industry-specific goals and also demonstrate design and evaluate computing solutions that solve a given problem using algorithmic principles and computer science practices and standards appropriate to its solution, while managing the trade-offs involved in design choices.
Through the development of this enhancement, there were a few issues I ran into. Since the Node.js language is still relatively new to me, the concept of callbacks and the way that Node.js code runs were troublesome at times. Since JavaScript runs in an asynchronous way, it could be hard at times to get your algorithm to run correctly since at times you could be waiting for data from a callback. 



