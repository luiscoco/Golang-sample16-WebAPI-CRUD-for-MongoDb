# Golang: How to create a WebAPI CRUD for MongoDB Microservice

## 1. Prerequisite

- Install Golang: https://go.dev/dl/

![image](https://github.com/luiscoco/Golang-sample16-WebAPI-CRUD-for-MongoDb/assets/32194879/2c71fd1f-95b1-4740-8877-b0d7ce46998e)

We download the file "**go1.21.6.windows-amd64.msi**" and execute it

- Install and Run **VSCode**

- Load the **Go extension** in VSCode

  ![image](https://github.com/luiscoco/Golang-sample16-WebAPI-CRUD-for-MongoDb/assets/32194879/9b4b5d29-2a7d-47a3-b266-96273e74eb7f)

- Install **Docker Desktop**

- Pull **mongodb docker image** and run it

We first pull the mongodb image

```
docker pull mongo
```

We run the mongo docker container

```
docker run --name mongodb -d -p 27017:27017 --restart unless-stopped mongo
```

![image](https://github.com/luiscoco/Golang-sample16-WebAPI-CRUD-for-MongoDb/assets/32194879/60d27a6f-edbb-4116-90c3-3ac8346fd813)

- Verify the image and container in **Docker Desktop**

![image](https://github.com/luiscoco/Golang-sample16-WebAPI-CRUD-for-MongoDb/assets/32194879/5a959223-0fbe-46d8-be07-6d2136f99807)

![image](https://github.com/luiscoco/Golang-sample16-WebAPI-CRUD-for-MongoDb/assets/32194879/cda014ad-a77c-4fd1-a96b-2ab4770bbf12)


## 2. Run VSCode and creaet a Golang application

Create a project directory and open VSCode

Create the **main.go** file

![image](https://github.com/luiscoco/Golang-sample16-WebAPI-CRUD-for-MongoDb/assets/32194879/02924aab-9566-427a-97fa-98e0c7e35e4b)

```
package main

import (
    "context"
    "encoding/json"
    "net/http"
    "github.com/gorilla/mux"
    "go.mongodb.org/mongo-driver/bson"
    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
    "go.mongodb.org/mongo-driver/bson/primitive"
    "log"
)

// Book struct to map with MongoDB documents
type Book struct {
    ID     primitive.ObjectID `json:"id,omitempty" bson:"_id,omitempty"`
    Title  string             `json:"title,omitempty" bson:"title,omitempty"`
    Author string             `json:"author,omitempty" bson:"author,omitempty"`
    ISBN   string             `json:"isbn,omitempty" bson:"isbn,omitempty"`
}

var collection *mongo.Collection

func main() {
    connectDB()
    r := mux.NewRouter()

    r.HandleFunc("/books", createBook).Methods("POST")
    r.HandleFunc("/books", getBooks).Methods("GET")
    r.HandleFunc("/books/{id}", getBook).Methods("GET")
    r.HandleFunc("/books/{id}", updateBook).Methods("PUT")
    r.HandleFunc("/books/{id}", deleteBook).Methods("DELETE")

    log.Fatal(http.ListenAndServe(":8080", r))
}

func connectDB() {
    clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")
    client, err := mongo.Connect(context.TODO(), clientOptions)
    if err != nil {
        log.Fatal(err)
    }

    collection = client.Database("bookstore").Collection("books")
}

func createBook(w http.ResponseWriter, r *http.Request) {
    var book Book
    json.NewDecoder(r.Body).Decode(&book)
    
    result, err := collection.InsertOne(context.TODO(), book)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    json.NewEncoder(w).Encode(result)
}

func getBooks(w http.ResponseWriter, r *http.Request) {
    var books []Book
    cursor, err := collection.Find(context.TODO(), bson.M{})
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    defer cursor.Close(context.TODO())

    for cursor.Next(context.TODO()) {
        var book Book
        cursor.Decode(&book)
        books = append(books, book)
    }

    json.NewEncoder(w).Encode(books)
}

func getBook(w http.ResponseWriter, r *http.Request) {
    id, err := primitive.ObjectIDFromHex(mux.Vars(r)["id"])
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    var book Book
    err = collection.FindOne(context.TODO(), bson.M{"_id": id}).Decode(&book)
    if err != nil {
        http.Error(w, err.Error(), http.StatusNotFound)
        return
    }

    json.NewEncoder(w).Encode(book)
}

func updateBook(w http.ResponseWriter, r *http.Request) {
    id, err := primitive.ObjectIDFromHex(mux.Vars(r)["id"])
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    var book Book
    json.NewDecoder(r.Body).Decode(&book)

    _, err = collection.UpdateOne(
        context.TODO(),
        bson.M{"_id": id},
        bson.D{
            {"$set", bson.D{
                {"title", book.Title},
                {"author", book.Author},
                {"isbn", book.ISBN},
            }},
        },
    )
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    json.NewEncoder(w).Encode(book)
}

func deleteBook(w http.ResponseWriter, r *http.Request) {
    id, err := primitive.ObjectIDFromHex(mux.Vars(r)["id"])
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    _, err = collection.DeleteOne(context.TODO(), bson.M{"_id": id})
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.WriteHeader(http.StatusNoContent)
}
```

Now we need to initialize a module with this command. A **main.mod** file will be created

```
go mod init main
```

This is the **main.mod** file content

```
module yourmodule/bookstore

go 1.21.6

require (
	github.com/gorilla/mux v1.8.0
	go.mongodb.org/mongo-driver v1.7.0
)

require (
	github.com/go-stack/stack v1.8.0 // indirect
	github.com/golang/snappy v0.0.1 // indirect
	github.com/klauspost/compress v1.9.5 // indirect
	github.com/pkg/errors v0.9.1 // indirect
	github.com/xdg-go/pbkdf2 v1.0.0 // indirect
	github.com/xdg-go/scram v1.0.2 // indirect
	github.com/xdg-go/stringprep v1.0.2 // indirect
	github.com/youmark/pkcs8 v0.0.0-20181117223130-1be2e3e5546d // indirect
	golang.org/x/crypto v0.0.0-20200302210943-78000ba7a073 // indirect
	golang.org/x/sync v0.0.0-20190911185100-cd5d95a43a6e // indirect
	golang.org/x/text v0.3.5 // indirect
)
```

We generate the **go.sum** file with this command

```
go mod tidy
```

Now we can run the application

```
go run main.go
```

## 3. Test the application with Postman

We send two POST request for creting two books

![image](https://github.com/luiscoco/Golang-sample16-WebAPI-CRUD-for-MongoDb/assets/32194879/4e8759bf-2cab-4cd3-bea3-10d517ede47a)

This is the POST request body JSON

```
{
    "title": "The Go Programming Language",
    "author": "Alan A. A. Donovan and Brian W. Kernighan",
    "isbn": "0134190440"
}
```

We verify the database entries running the GET request

![image](https://github.com/luiscoco/Golang-sample16-WebAPI-CRUD-for-MongoDb/assets/32194879/ffa78c15-30d4-480c-81da-dea87450d0b2)

## 4. Verify the database with Studio 3T free for MongoDB

We connect to Mongo database with this connection string: mongodb://localhost:27017

![image](https://github.com/luiscoco/Golang-sample16-WebAPI-CRUD-for-MongoDb/assets/32194879/415646ee-1d9b-4b0b-951c-6c7464dbaf29)
