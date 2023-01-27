# Golang Microservices
A Microservices Architecture consists of a collection of small, autonomous services. Each service is self-contained and should implement a single business capability.

Below is an example of designing and implementing Microservices using:

1) Gin Gonic

2)Traefik

3)MongoDB
       
       
 ![Screenshot (84)](https://user-images.githubusercontent.com/93249038/215036753-14cf112d-352e-430f-bb18-d37f86cefc2d.png)



#  Movie Service Build & Run
# 2.1. Setup development environement
--> Install Golang

--> Install MongoDB

https://docs.mongodb.com/manual/installation

# Start MongoDB

$ mongod --dbpath="[your_database_path]"

#Install neccessary Golang packages
 
$ go get -u github.com/swaggo/swag/cmd/swag github.com/swaggo/gin-swagger github.com/swaggo/gin-swagger/swaggerFiles github.com/alecthomas/template github.com/gin-gonic/gin github.com/sirupsen/logrus gopkg.in/mgo.v2/bson github.com/natefinch/lumberjack

#  Compile & run services
- Generate API documentation using Swag    
 
Guideline: https://github.com/swaggo/swag

# Change configuration file
Update values of file ./config/config.json

{
    "port": ":8808",
    "enableGinConsoleLog": true,
    "enableGinFileLog": false,

    "logFilename": "logs/server.log",
     "logMaxSize": 10,
    "logMaxBackups": 10,
    "logMaxAge": 30,

    "mgAddrs": "127.0.0.1:27017",
    "mgDbName": "go-microservices",
    "mgDbUsername": "",
    "mgDbPassword": "",

    "jwtSecretPassword": "Mithun",
    "issuer": ""
}


# Run services
Run the Authentication service
$ cd [go-microservices]/src/user-microservice
$ go run main.go
>> [GIN-debug] Listening and serving HTTP on :8808
Authentication Swagger
http://localhost:8808/swagger/index.html

# Run the Movie service
$ cd [go-microservices]/src/movie-microservice
$ go run main.go
>> [GIN-debug] Listening and serving HTTP on :8809
Movie Swagger
http://localhost:8809/swagger/index.html

# Setup API Gateway
3.1. Download the latest traefik

https://github.com/containous/traefik/releases/latest

# Launch traefik

Copy traefik.toml from [go-microservices]/traefik to traefik execution folder and update parameters.
defaultEntryPoints = ["http", "https"]
[entryPoints]


    [entryPoints.http]
    
    address = ":7777"

[api]

    entryPoint = "http"
    
    dashboard = true

[file]

[frontends]


    [frontends.usermanagement]
    
        entrypoints = ["http"]	
	
	
        backend="usermanagement"
	
	
        [frontends.usermanagement.routes.matchUrl]
	
	
            rule="PathPrefixStrip:/seedotech.usermanagement"

    [frontends.moviemanagement]
    
        entrypoints = ["http"]	
	
	
	
        backend="moviemanagement"
	
        [frontends.moviemanagement.routes.matchUrl]
	
	
            rule="PathPrefixStrip:/seedotech.moviemanagement"

[backends]




    [backends.usermanagement]
    
    
        [backends.usermanagement.servers.main1]
	
	
            url = "http://192.168.1.9:8808"
	    
            weight = 3

        [backends.usermanagement.servers.main2]
	
            url = "http://192.168.1.10:8808"
	    
            weight = 1

    [backends.moviemanagement]
    
        [backends.moviemanagement.servers.main1]
	
            url = "http://192.168.1.9:8809"
	    
            weight = 3

        [backends.moviemanagement.servers.main2]
	
	
            url = "http://192.168.1.10:8809"
	    
            weight = 1

        [backends.moviemanagement.servers.main3]
	
            url = "http://192.168.1.12:8809"
	    
	    
            weight = 2				            
# Run traefik
$ ./traefik -c traefik.toml
# Traefik dashboard

Traefik Dashboard

Now, assume that the Gateway IP is 192.168.1.8, then, when sending a request to the gateway like:

GET http://192.168.1.8:7777/seedotech.usermanagement/api/v1/movies/list

Gateway will route request to

GET http://192.168.1.9:8808/api/v1/movies/list

OR

GET http://192.168.1.10:8808/api/v1/movies/list

#  REST API Response Format
4.1. Successful response returns the application data through HTTP 200 OK Message

Example
// User information
type User struct {
	ID       bson.ObjectId `bson:"_id" json:"id" example:"5bbdadf782ebac06a695a8e7"`
	
	
	Name     string        `bson:"name" json:"name" example:"Mithun1508"`
	
	Password string        `bson:"password" json:"password" example:"Mitbhun1508"`
}

// Token string
type Token struct {
	Token string `json:"token" 
	example:"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoicmF5Y2FkIiwicm9sZSI6IiIsImV4cCI6MTUzOTI0OTc3OSwiaXNzIjoic2VlZG90ZWNoIn0.lVHq8J_0vfvECbplogAOCDCNh63ivTBOhya8KE6Ew_E"`
	
	
}

// Returning user array (Code 200)
# @Success 200 {array} models.User

[


  {
  
    "id": "5bbc4dd782ebac0a8d2c02fe",
    
    "name": "Mithun",
    
    "password": "user password"
  },
  {
    "id": "5bbdadf782ebac06a695a8e7",
    
    "name": "Mithun1508",
    
     "password": "Mithun1508"
  }
]

// Returning token string (Code 200)

# @Success 200 {object} models.Token
{


    "token":
    "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoicmF5Y2FkIiwicm9sZSI6IiIsImV4cCI6MTUzOTI0OTc3OSwiaXNzIjoic2VlZG90ZWNoIn0.lVHq8J_0vfvECbplogAOCDCNh63ivTBOhya8KE6Ew_E"
    
}
4.2. Failed response returns the application error information (NOT HTTP 200 OK Message)

type Error struct {
	Code    int    `json:"code" example:"27"`
	
	
	Message string `json:"message" example:"Error message"`
}

Example:
// Returning HTTP StatusBadRequest (Code 400)
# @Failure 400 {object} models.Error
{
    "code": -1,
    
    "message": "User name is empty"
}


// Returning HTTP StatusNotFound (Code 404)
# @Failure 404 {object} models.Error
{
    "code": -1,
    
    "message": "User not found"
}
5. Coding Convention
5.1. MongoDB Naming Convention
# COLLECTION NAMES
  
  The name should be a plural of the types of documents to be saved.
Use camelCase. Normally you shouldn’t have to because the collection name will be one word (plural of the types of documents saved).
# DATABASE NAMES
Try to have the database named after the project and one database per project.
Use camelCase.

# FIELD NAMES
Use camelCase.
Don’t use _ underscore as the starting character of the field name. The only field with _ undescore should be _id.
# Golang Coding Convention

1)Foler naming: lowercase with the hyphens seperated between words. e.g. user-microservice

2) Package naming: no underscore between words, everything is lowercase. e.g. controller

3) Go file naming: no underscore between words, everything is lowercase. e.g. moviegenre.go

4)Files with the suffix _test.go are only compiled and run by the go test tool

5) Files with os and architecture specific suffixes automatically follow those same constraints, e.g. name_linux.go will only build on linux, name_amd64.go will only build on amd64. This is the same as having a //+build amd64 line at the top of the file


# Terminologies Concepts
1)Discovery Service e.g. Eureka

2) Load Balancing e.g. Ribbon

3) Circuit Breaker e.g. Hystrix

4) Service Registry


References: https://awesomeopensource.com/project/raycad/go-microservices (Credits to the Author. Lots Learning and really enjoyed implementing this Much needed Topic for go beginners) 

