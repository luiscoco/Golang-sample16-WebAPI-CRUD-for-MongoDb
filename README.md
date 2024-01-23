# Golang: How to create a WebAPI CRUD for MongoDB Microservice

## 1. Prerequisite

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

Create the main.go file

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

## 3. 

