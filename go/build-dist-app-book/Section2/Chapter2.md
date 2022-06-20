# Chapter 2. Settin Up API Endpoints

## Exploring API functionality

To illustrate how to build a RESTful API, we will build a cooking application.

The application will do the following:

- Display the recipes that are submitted by the users, along with their ingredients and
instructions.
- Allow anyone to post a new recipe.

## Defining the data model

```go
type Recipe struct {
	ID           string    `json:"id"`
	Name         string    `json:"name"`
	Tags         []string  `json:"tags"`
	Ingredients  []string  `json:"ingredients"`
	Instructions []string  `json:"instructions"`
	PublishedAt  time.Time `json:"publishedAt"`
}
```

## HTTP Endpoints

| HTTP Method | Resource              | Description                 |
|-------------|-----------------------|-----------------------------|
| GET         | /recipes              | Returns a list of recipes   |
| POST        | /recipes              | Create a new recipe         |
| PUT         | /recipes/{id}         | Update an existing recipe   |
| DELETE      | /recipes/{id}         | Delete an existing recipe   |
| GET         | /recipes/search?tag=X | Searches for recipes by tag |

## Implemneting HTTP routes

_Pretty simple_

## Writing the OpenAPI Specification

The OpenAPI Specification (formerly known as the _Swagger Specification_) is an API
description format or API definition language.

### Swagger metadata

| **Property** | **Description**                                                 |
|--------------|-----------------------------------------------------------------|
| Schemes      | The transfer protocols supported by your API are HTTP and HTTPS |
| Host         | The host where API is served (for instance, localhost:8080)     |
| BasePath     | The default base path for the API (e.g. /v1)                    |
| Version      | The current version of the API                                  |
| Contact      | The API owner or author                                         |
| Consumes     | A list of default MIME type values (API receives)               |
| Producers    | A list of default MIME type values (API sends)                  |

```go
// Recipes API
//
// This is a sample recipes API. You can find out more about the API at https://github.com/PacktPublishing/Building-Distributed-Applications-in-Gin.
//
//	Schemes: http
//  Host: localhost:8080
//	BasePath: /
//	Version: 1.0.0
//	Contact: Mohamed Labouardy <mohamed@labouardy.com> https://labouardy.com
//
//	Consumes:
//	- application/json
//
//	Produces:
//	- application/json
// swagger:meta
package main
```

__VERI INITRESTING VERI INTRESTING__