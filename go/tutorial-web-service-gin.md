# Tutorial: Developing a RESTful API with Go and Gin

Gin simplifies many coding tasks associated with building web applications, including web services. In this tutorial, you’ll use Gin to route requests, retrieve request details, and marshal JSON for responses.

## Design API endpoints

You’ll build an API that provides access to a store selling vintage recordings on vinyl. So you’ll need to provide endpoints through which a client can get and add albums for users.  
When developing an API, you typically begin by designing the endpoints. Your API’s users will have more success if the endpoints are easy to understand.

`/albums`
- GET – Get a list of all albums, returned as JSON.
- POST – Add a new album from request data sent as JSON.

`/albums/:id`
- GET – Get an album by its ID, returning the album data as JSON.

## Create a folder for your code

```sh
$ mkdir web-service-gin
$ cd web-service-gin
$ go mod init example/web-service-gin
```

## Create the data

To keep things simple for the tutorial, you’ll store data in memory.

web-service-gin/main.go
```go
package main

// album represents data about a record album.
type album struct {
    // Struct tags such as json:"artist" specify what a field’s name should be when the struct’s contents are serialized into JSON. Without them, the JSON would use the struct’s capitalized field names – a style not as common in JSON.
    ID     string  `json:"id"`
    Title  string  `json:"title"`
    Artist string  `json:"artist"`
    Price  float64 `json:"price"`
}

// albums slice to seed record album data.
var albums = []album{
    {ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
    {ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
    {ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}
```

## Write a handler to return all items

When the client makes a request at `GET` `/albums`, you want to return all the albums as JSON.

To do this, you’ll write the following:

- Logic to prepare a response
- Code to map the request path to your logic

__Note__ that this is the reverse of how they’ll be executed at runtime, but you’re adding dependencies first, then the code that depends on them.

web-service-gin/main.go
```go
// getAlbums responds with the list of all albums as JSON.
// gin.Context is the most important part of Gin. It carries request details, validates and serializes JSON, and more. 
func getAlbums(c *gin.Context) {
    // Call Context.IndentedJSON to serialize the struct into JSON and add it to the response.
    // The function’s first argument is the HTTP status code you want to send to the client. Here, you’re passing the StatusOK constant from the net/http package to indicate 200 OK.
    c.IndentedJSON(http.StatusOK, albums)
}
```
Near the top of main.go, just beneath the albums slice declaration, paste the code below to assign the handler function to an endpoint path.  
This sets up an association in which getAlbums handles requests to the /albums endpoint path.

web-service-gin/main.go
```go
func main() {
    // Initialize a Gin router using Default.
    router := gin.Default()
    // Use the GET function to associate the GET HTTP method and /albums path with a handler function.
    router.GET("/albums", getAlbums)
    // Use the Run function to attach the router to an http.Server and start the server.
    router.Run("localhost:8080")
}
```
Near the top of main.go, just beneath the package declaration, import the packages you’ll need to support the code you’ve just written.

web-service-gin/main.go
```go
package main

import (
    "net/http"

    "github.com/gin-gonic/gin"
)
```

### Run the code

Begin tracking the Gin module as a dependency.
```sh
$ go get .
```
Run the code.
```sh
$ go run .
```
From a new command line window, use curl to make a request to your running web service.
```sh
$ curl http://localhost:8080/albums
```

## Write a handler to add a new item

When the client makes a `POST` request at `/albums`, you want to add the album described in the request body to the existing albums data.

To do this, you’ll write the following:
- Logic to add the new album to the existing list.
- A bit of code to route the POST request to your logic.

web-service-gin/main.go
```go
// postAlbums adds an album from JSON received in the request body.
func postAlbums(c *gin.Context) {
    var newAlbum album

    // Call BindJSON to bind the received JSON to
    // newAlbum.
    if err := c.BindJSON(&newAlbum); err != nil {
        return
    }

    // Add the new album to the slice.
    albums = append(albums, newAlbum)
    // Add a 201 status code to the response, along with JSON representing the album you added.
    c.IndentedJSON(http.StatusCreated, newAlbum)
}
```
Change your main function so that it includes the router.POST function, as in the following:

web-service-gin/main.go
```go
func main() {
    router := gin.Default()
    router.GET("/albums", getAlbums)
    // Associate the POST method at the /albums path with the postAlbums function.
    router.POST("/albums", postAlbums)

    router.Run("localhost:8080")
}
```

### Run the code

```sh
$ go run .
```

```sh
curl http://localhost:8080/albums \
    --include \
    --header "Content-Type: application/json" \
    --request "POST" \
    --data '{"id": "4","title": "The Modern Sound of Betty Carter","artist": "Betty Carter","price": 49.99}'
```

```sh
$ curl http://localhost:8080/albums \
    --header "Content-Type: application/json" \
    --request "GET"
```

## Write a handler to return a specific item

When the client makes a request to `GET /albums/[id]`, you want to return the album whose ID matches the id path parameter.

This `getAlbumByID` function will extract the ID in the request path, then locate an album that matches.

web-service-gin/main.go
```go
// getAlbumByID locates the album whose ID value matches the id
// parameter sent by the client, then returns that album as a response.
func getAlbumByID(c *gin.Context) {
    // Use Context.Param to retrieve the id path parameter from the URL.
    id := c.Param("id")

    // Loop over the list of albums, looking for
    // an album whose ID value matches the parameter.
    for _, a := range albums {
        if a.ID == id {
            c.IndentedJSON(http.StatusOK, a)
            return
        }
    }
    c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}
```
Finally, change your main so that it includes a new call to router.GET, where the path is now /albums/:id, as shown in the following example.
```go
func main() {
    router := gin.Default()
    router.GET("/albums", getAlbums)
    // In Gin, the colon preceding an item in the path signifies that the item is a path parameter.
    router.GET("/albums/:id", getAlbumByID)
    router.POST("/albums", postAlbums)

    router.Run("localhost:8080")
}
```