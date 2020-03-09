# Beego Auto Generated API Docs
[![Build Status](https://travis-ci.org/astaxie/beego.svg?branch=master)](https://travis-ci.org/astaxie/beego) 
[![GoDoc](http://godoc.org/github.com/astaxie/beego?status.svg)](http://godoc.org/github.com/astaxie/beego) 
[![Foundation](https://img.shields.io/badge/Golang-Foundation-green.svg)](http://golangfoundation.org) 
[![Go Report Card](https://goreportcard.com/badge/github.com/astaxie/beego)](https://goreportcard.com/report/github.com/astaxie/beego)

Swagger™ is a specification and complete framework implementation for describing, producing, consuming, and visualizing RESTful web services.

With the auto generated document by integrating Swagger into beego we can have these benifits:
1. Standard declarative comments for API functions which makes
2. The API code becomes more maintainable with the comments 
3. The Swagger style API document will be generated automacally based on the comments.

## Creating the Beego API aplication

### Install Bee & Beego
```bash
$ go get -u github.com/beego/bee
$ go get -u github.com/astaxie/beego
```

### Generate API Project
Go to `GOPATH/src` directory and execute command: 
```
$ bee api bapi
``` 

### Run Project
Then go to the application directory `cd bapi` and run: 
```bash
$ bee run -downdoc=true -gendoc=true.
```
`You can only use 127.0.0.1 here other than loalhost to prevent the Ajax CORS issue.`

## Directory structure
You’ll see the following directory structure created by `bee api`
```bash
|-- bapi
|-- conf
|   `-- app.conf
|-- controllers
|   |-- object.go
|   `-- user.go
|-- docs
|   |-- doc.go
|   `-- docs.go
|-- lastupdate.tmp
|-- main.go
|-- models
|   |-- object.go
|   `-- user.go
|-- routers
|   |-- commentsRouter.go
|   `-- router.go
|-- swagger
`-- tests
    `-- default_test.go
```
* `main.go` Our bootstrap file.
* `bapi` The compiled binary file.
* `conf` The directory for configuration file.
* `controllers` Our controllers
* `models` Our models
* `docs` Our auto generated document
* `lastupdate.tmp` Cache file for Annotation Router
* `routers` Routers
* `swagger` A static directory for Swagger API related files.
* `test` The testcase for the application. You can run the testcases without run the application.

## The bootstrap main file
The following is the main.go file:
```go
package main

import (
    _ "bapi/docs"
    _ "bapi/routers"

    "github.com/astaxie/beego"
)

func main() {
    if beego.RunMode == "dev" {
        beego.DirectoryIndex = true
        beego.StaticDir["/swagger"] = "swagger"
    }
    beego.Run()
}
```

## Namespace Router
The router is using the namespace router. Two points two notice:
1. The API with auto generated document only support namespace router now.
2. It only support NSNamespace and NSInclude within two levels.

The following is our routers:
```go
func init() {
    ns := beego.NewNamespace("/v1",
        beego.NSNamespace("/object",
            beego.NSInclude(
                &controllers.ObjectController{},
            ),
        ),
        beego.NSNamespace("/user",
            beego.NSInclude(
                &controllers.UserController{},
            ),
        ),
    )
    beego.AddNamespace(ns)
}
```

## Annotation Router
`Annotation Router` are some comments set for Controller that Beego will parse them into the real routers automatically. Let’s see what do we do to for it in `ObjectController` and `UserController`:
```go
// Operations about object
type ObjectController struct {
    beego.Controller
}

// @Title create
// @Description create object
// @Param   body        body    models.Object   true        "The object content"
// @Success 200 {string} models.Object.Id
// @Failure 403 body is empty
// @router / [post]
func (this *ObjectController) Post() {
    var ob models.Object
    json.Unmarshal(this.Ctx.Input.RequestBody, &ob)
    objectid := models.AddOne(ob)
    this.Data["json"] = map[string]string{"ObjectId": objectid}
    this.ServeJson()
}
```

`The router analysis will only happens in dev mode. So if you updated your code related to Annotation Router, you need to run dev mode at least once to generate the routers.`

## Auto Generated API Document

### API Document
We mentioned it before in `router.go` there are bunch of annotaions. It’s the description of this API application:
```go
// @APIVersion 1.0.0
// @Title beego Test API
// @Description beego has a very cool tools to autogenerate documents for your API
// @Contact astaxie@gmail.com
// @TermsOfServiceUrl http://beego.me/
// @License Apache 2.0
// @LicenseUrl http://www.apache.org/licenses/LICENSE-2.0.html
```

All of them are optional followed by a string. The following is the doc we generated: http://127.0.0.1:8080/docs:
```json
{
  "apiVersion": "1.0.0",
  "swaggerVersion": "1.2",
  "apis": [
    {
      "path": "/object",
      "description": "Operations about object\n"
    },
    {
      "path": "/user",
      "description": "Operations about Users\n"
    }
  ],
  "info": {
    "title": "beego Test API",
    "description": "beego has a very cool tools to autogenerate documents for your API",
    "contact": "astaxie@gmail.com",
    "termsOfServiceUrl": "http://beego.me/",
    "license": "Url http://www.apache.org/licenses/LICENSE-2.0.html"
  }
}
```

### Annotation for Controller
We add the annotation for controller and it’s methods to describe the controller.

```go
// Operations about object
type ObjectController struct {
    beego.Controller
}
```

The comment above describes the controller. The comment following for the methods will describe the router, parameters and the return value.

```go
// @Title Get
// @Description find object by objectid
// @Param   objectId        path    string  true        "the objectid you want to get"
// @Success 200 {object} models.Object
// @Failure 403 :objectId is empty
// @router /:objectId [get]
func (this *ObjectController) Get() {
}
```
Following code is the JSON data we generated from these annotations.
```json
{
  "path": "/object/{objectId}",
  "description": "",
  "operations": [
    {
      "httpMethod": "GET",
      "nickname": "Get",
      "type": "",
      "summary": "find object by objectid",
      "parameters": [
        {
          "paramType": "path",
          "name": "objectId",
          "description": "\"the objectid you want to get\"",
          "dataType": "string",
          "type": "",
          "format": "",
          "allowMultiple": false,
          "required": true,
          "minimum": 0,
          "maximum": 0
        }
      ],
      "responseMessages": [
        {
          "code": 200,
          "message": "models.Object",
          "responseModel": "Object"
        },
        {
          "code": 403,
          "message": ":objectId is empty",
          "responseModel": ""
        }
      ]
    }
  ]
}
```

### Customized Object Annotation
Following is the definition of our object:

```go
type Object struct {
    ObjectId   string
    Score      int64
    PlayerName string
}
```
The auto generated object description for document is:
```go
Object {
ObjectId (string, optional): ,
PlayerName (string, optional): ,
Score (int64, optional):
}
```

We can see all the fields are optional, we can change it by adding tag as showing below:
```go
type Object struct {
    ObjectId   string   `required:"true" description:"object id"`
    Score      int64    `required:"true" description:"players's scores"`
    PlayerName string   `required:"true" description:"plaers name, used in system"`
}
```

To change the returned fields name, you can add tag with name json or thrife as showing below:
```go
type Object struct {
    ObjectId   string   `json:"object_id"`
    Score      int64    `json:"player_score"`
    PlayerName string   `json:"player_name"`
}
```

The auto generated object description for document is:
`Object { object_id (string, optional): , player_score (string, optional): , player_name (int64, optional): }`

source: https://beego.me/blog/beego_api