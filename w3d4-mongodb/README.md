# Intro to databases and MongoDB

The following lecture notes and example app were heavily based on [Fabio Neves' work](https://github.com/fzero/lhl-lectures/tree/master/w3d4-mongodb), which was originally based on [Khurram Virani's original notes](https://github.com/kvirani/express_mongo_todo_example).

Please also see [Vaz's lecture notes](https://github.com/vaz/express_mongo_todo_example/blob/master/Lecture-notes.md) for a very helpful reference guide on completing your exercises.

The code discussed in class can be found in [`/code`](code).

## Topics covered (Summary)

- Methods of Data Storage
  - In memory
  - In plain text
  - In databases

- MongoDB Basics
  - Document Oriented Database (document store)
  - Major Components
  - Mongo Shell
  - Primary Key

- MongoDB in a Node.js App
  - 

## Methods of Data Storage

Most apps need to store data in some way. There are different ways of doing this, let's discuss some of them.

### In Memory

Up to this point in the course, you have been simply declaring a variable in your Node.js app and then making changes to it as you go.

```js
const data = {
  "some": "data"
}

data["hello"] = "world";

console.log(data)  // Object {some: "data", hello: "world"}
```

And I'm sure all of you have experienced what happens when the server stops and starts again.

All changes are lost whenever you restart the server.

This is because the data is being stored in memory. More specifically, the server has no persistent memory of any kind when you turn it off and on again. It's as if it's starting from scratch each time.

Clearly, this is not okay for most apps. We need data that is able to persist regardless of whether the server has been turned off or not.

### In Plain Text

One way of getting around this would be to save your data into a file. Node's file stream library `fs` allows you to read and write directly to a file in plain text.

#### Some Challenges

Although this allows us to get persistent data, it is still not the most ideal way for us to store information for a webapp. Here are some of the challenges with using plain text files for application data storage:

- Too much work on the app developer to work with this file (Open, Read, Write, etc)
- Have to get the full file every time (this is really bad if you have a lot of data)
- No way to easily search data by criteria
- Concurrency: multiple apps/processes can't read/write the file without causing inconsistency. Web apps can have many users CRUDing data and so we need to be able to read/write concurrently without problems.

#### Practical Uses

Although plain text files are no good as persistent storage for application data, it works very well for configuration data since it is something that rarely changes. Many JSON files in web development are used for configuration purposes.

It's also helpful to use this kind of data storage when you are transferring a huge dump of data from one place to another. Comma-separated values (CSV) files are frequently used in this manner because they can be exported and imported into Excel with ease. This is why some of these formats are often called data serialization formats. They take a bunch of data, and serialize it into one long string into a single file. Below are some common data serialization formats.

##### CSV

The simplest format is probably the **[comma-separated-value (CSV)](https://en.wikipedia.org/wiki/Comma-separated_values)** file:

```csv
some,data
hello,world
```

Each line in the file is like a row in a table. CSV files can be read by many programs, such as Excel.

##### JSON

**[Javascript Object Notation](https://en.wikipedia.org/wiki/JSON)** is a popular format for HTTP transactions.

```json
{
  "some": "data",
  "hello": "world"
}
```

This format is very similar to Javascript objects (as the name implies) and is therefore very intuitive to work with in Javascript itself.

JSON has become a standard on the web for APIs sending and receiving information over the internet. You can also see it in action as a configuration file in node projects `package.json`.

##### XML

**[Extensible Markup Language (XML)](https://en.wikipedia.org/wiki/XML)** is used in a variety of places. The syntax is likely familiar to you because HTML is a loose cousin of XML.

```xml
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend!</body>
</note>
```

##### YAML

**[Yet Another Markup Language (YAML)](https://en.wikipedia.org/wiki/YAML)** is a lightweight data serialization format that is also easily human readable.

```yaml
 --- # Favorite movies
 - Casablanca
 - North by Northwest
 - The Man Who Wasn't There
```

### In Databases

For application data, the standard practice is to use a database. What is a database? It's simply an organized collection of data that persists over time and can have many features to make it easier to for querying data (reading/writing/etc.).

In the context of a webapp, you are already familiar with the concept of a client and server. The client requests something from the server, and the server provides the client with an appropriate response.

You can think of a database as another extension on this abstraction. The client asks the server for some information. Depending on what was requested, the server will then ask the database some information. The database will give the server what it requested. And subsequently, the server will then be able to provide an appropriate response to the client.

A rough diagram:

```
client <-> server <-> database
```

This way, if the server goes down, the data will still persist in the database.

## MongoDB Basics

Basic crud operations: https://docs.mongodb.com/manual/crud/

There are currently two popular models of databases in web development: the [relational model](https://en.wikipedia.org/wiki/Relational_model) and the [document store model](https://en.wikipedia.org/wiki/Document-oriented_database).

MongoDB is a document-oriented database. It has become one of the most popular databases used because of its simplicity. As such, it has a lot of support in the Node community, becoming a popular option for many projects.

MongoDB is open source (like everything we use/teach here). It is a non-relational db and is more of an object store (objects are referred to as "documents").

### Major Components

The major components of a MongoDB setup are: Databases, Collections, and Documents.

Most apps usually only have one **database**, but within each database, you can have different types of data. Each piece of data is represented as a **collection** of **documents**. Or in other words:

```
- Databases (Most apps usually have one)
  - Collections (think of it like an Array)
    - Documents (JS Objects with _id's)
```

----

It's best to illustrate this with an example of a [Reddit](http://www.reddit.com)-like app:

MyRedditApp has just **one database**, but inside this database, it has **two collections**: the **Posts** collection, and the **Comments** collection.

A typical **Post** document might look like this:
```json
{
  "_id": 2,
  "title": "An Awesome Search Engine",
  "url": "http://www.google.com"
}
```

A typical **Comment** document might look like this:

```json
{
  "_id": 1,
  "post_id": 2,
  "comment_text": "I love search engines!"
}
```

This is just a simplified example, since it's lacking a users collection and many other features. But nevertheless, I hope this serves to illustrate the basic structure of MongoDB collections and documents.

### Mongo Shell / REPL

This allows us to test out our MongoDB queries with an easy terminal interface.

We went over the mongo shell. Here are the commands we ran:

```javascript
show dbs
use todos_app
db
db.todos
db.todos.insert({ desc: 'Example TODO', completed: false })
db.todos.insert({ desc: 'Example Completed TODO', completed: true })
db.todos.find()
db.todos.find({completed: true})
db.todos.find({id: ObjectId("unique-mongo-id-here")})
```

We talked about how:

- Mongo is very javascript centric.
- Mongo has "transactional" functions on collections to allow us to CRUD (Create Read Update Delete) documents within those collections.
- There are many other functions that you can look up in the documentation (how to search, insert multiple, batch delete, batch update, etc, etc)
- Collections can just be viewed as properties of the `db` object (`db.todos` can also be written as `db.collection('todos')`) therefore their name is what uniquely identify them.
- Mongo assigns an `_id` key/property to any document so we can uniquely identify it. More on this below.

### Primary Key

The `_id` is a "Primary Key" that uniquely identifies each object (document) within our collections.

It is automatically added to any object we insert and we can use it to find single object from a collection (see example above).

## MongoDB in a Node.js App

The app is in the `/code` folder, so first we should navigate to it: `cd code`. Now we are ready to start:

1. Install the packages: `npm install`
2. Create an empty directory called data: `mkdir data`
3. Start the MongoDB database in the folder: `mongod --dbpath data`
4. Start the server with the NPM script (this basically runs `app.js`): `npm start`

