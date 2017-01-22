+++
draft = false
date = "2017-01-19T09:15:18+08:00"
title = "Mongo Introduction"

+++

# Introdution

1. [NoSQL](https://zh.wikipedia.org/wiki/NoSQL)
2. Open Source
3. No Need For ORM( Object Relational Mapping)
4. Document(Record)

```javascript
{
   "_id" : ObjectId("54c955492b7c8eb21818bd09"),
   "address" : {
      "street" : "2 Avenue",
      "zipcode" : "10075",
      "building" : "1480",
      "coord" : [ -73.9557413, 40.7720266 ]
   },
   "borough" : "Manhattan",
   "cuisine" : "Italian",
   "grades" : [
      {
         "date" : ISODate("2014-10-01T00:00:00Z"),
         "grade" : "A",
         "score" : 11
      },
      {
         "date" : ISODate("2014-01-16T00:00:00Z"),
         "grade" : "B",
         "score" : 17
      }
   ],
   "name" : "Vella",
   "restaurant_id" : "41704620"
}
```

5. Collection(Table)

> primary key: _id

6. Database


# Connect Database

```javascript
mongo --host 192.168.0.238 --port 27017 -u demo -p 123456 --authenticationMechanism MONGODB-CR demo
```

# Insert

```javascript
# use database
use demo

# show tables
show tables

show collections

# insert
db.restaurants.insert(
   {
      "address" : {
         "street" : "2 Avenue",
         "zipcode" : "10075",
         "building" : "1480",
         "coord" : [ -73.9557413, 40.7720266 ]
      },
      "borough" : "Manhattan",
      "cuisine" : "Italian",
      "grades" : [
         {
            "date" : ISODate("2014-10-01T00:00:00Z"),
            "grade" : "A",
            "score" : 11
         },
         {
            "date" : ISODate("2014-01-16T00:00:00Z"),
            "grade" : "B",
            "score" : 17
         }
      ],
      "name" : "Vella",
      "restaurant_id" : "41704620"
   }
)

# after insert, return  WriteResult
WriteResult({ "nInserted" : 1 })

# or
var r = db.restaurants.insert(...)

# Three methods to insert
insert  insertOne  insertMany
```

# Find or Query

```javascript
db.restaurants.find( { "address.zipcode": "10075", "borough": "Manhattan"  } )
```

### Array Query

#### Exact Match
```javascript
db.users.find( { badges: [ "blue", "black" ] } )

{
   "_id" : 1,
   "name" : "sue",
   "age" : 19,
   "type" : 1,
   "status" : "P",
   "favorites" : { "artist" : "Picasso", "food" : "pizza" },
   "finished" : [ 17, 3 ]
   "badges" : [ "blue", "black" ],
   "points" : [ { "points" : 85, "bonus" : 20 }, { "points" : 85, "bonus" : 10 } ]
}
```

#### Match an Array Element

```javascript
db.users.find( { badges: "black" } )

{
   "_id" : 1,
   "name" : "sue",
   "age" : 19,
   "type" : 1,
   "status" : "P",
   "favorites" : { "artist" : "Picasso", "food" : "pizza" },
   "finished" : [ 17, 3 ]
   "badges" : [ "blue", "black" ],
   "points" : [ { "points" : 85, "bonus" : 20 }, { "points" : 85, "bonus" : 10 } ]
}
{
   "_id" : 4,
   "name" : "xi",
   "age" : 34,
   "type" : 2,
   "status" : "D",
   "favorites" : { "artist" : "Chagall", "food" : "chocolate" },
   "finished" : [ 5, 11 ],
   "badges" : [ "red", "black" ],
   "points" : [ { "points" : 53, "bonus" : 15 }, { "points" : 51, "bonus" : 15 } ]
}
{
   "_id" : 6,
   "name" : "abc",
   "age" : 43,
   "type" : 1,
   "status" : "A",
   "favorites" : { "food" : "pizza", "artist" : "Picasso" },
   "finished" : [ 18, 12 ],
   "badges" : [ "black", "blue" ],
   "points" : [ { "points" : 78, "bonus" : 8 }, { "points" : 57, "bonus" : 7 } ]
}
```

#### Match a Specific Element of an Array

```javascript
db.users.find( { "badges.0": "black" } )

{
   "_id" : 6,
   "name" : "abc",
   "age" : 43,
   "type" : 1,
   "status" : "A",
   "favorites" : { "food" : "pizza", "artist" : "Picasso" },
   "finished" : [ 18, 12 ],
   "badges" : [ "black", "blue" ],
   "points" : [ { "points" : 78, "bonus" : 8 }, { "points" : 57, "bonus" : 7 } ]
}
```

[More](https://docs.mongodb.com/manual/tutorial/query-documents/#read-operations-arrays)

#### Specify Conditions with Operators

```javascript
db.restaurants.find( { "grades.score": { $gt: 30 } } )
db.restaurants.find( { "grades.score": { $lt: 10 } } )
db.restaurants.find( { "cuisine": "Italian", "address.zipcode": "10075" } )
db.restaurants.find(
   { $or: [ { "cuisine": "Italian" }, { "address.zipcode": "10075" } ] }
)
```

[more](https://docs.mongodb.com/manual/reference/operator/query/)

#### Sort Query Results

```javascript
db.restaurants.find().sort( { "borough": 1, "address.zipcode": 1 } )
```

#### Show partial fields

```javascript
db.restaurants.find({}, {borough: 1})
```

#### Pretty results

```javascript
db.restaurants.find().pretty()
```

# Update

```javascript
db.restaurants.update(
    { "name" : "Juni" },
    {
      $set: { "cuisine": "American (New)" },
      $currentDate: { "lastModified": true }
    }
)

db.restaurants.update(
  { "restaurant_id" : "41156888" },
  { $set: { "address.street": "East 31st Street" } }
)

db.restaurants.update(
  { "address.zipcode": "10016", cuisine: "Other" },
  {
    $set: { cuisine: "Category To Be Determined" },
    $currentDate: { "lastModified": true }
  },
  { multi: true, upsert: true}
)

db.restaurants.update(
   { "restaurant_id" : "41704620" },
   {
     "name" : "Vella 2",
     "address" : {
              "coord" : [ -73.9557413, 40.7720266 ],
              "building" : "1480",
              "street" : "2 Avenue",
              "zipcode" : "10075"
     }
   }
)
```

## 自增ID的实现

1. collection store id
2. $inc operator

```javascript
db.ids.insert({name: 'test', value: NumberInt(0)})
db.ids.update({name: 'test'}, {$inc: {value: 1}})
```

[more](https://docs.mongodb.com/manual/reference/operator/update/)

# Remove

```javascript
db.restaurants.remove( { "borough": "Manhattan" } )
db.restaurants.remove( { "borough": "Queens" }, { justOne: true } )
db.restaurants.remove( { } )
```

## Drop collection

```javascript
db.restaurants.drop()
```

# Data Aggregation

```javascript
db.collection.aggregate( [ <stage1>, <stage2>, ... ] )
```

## Group

```javascript
db.restaurants.aggregate(
   [
     { $group: { "_id": "$borough", "count": { $sum: 1 } } }
   ]
)

{ "_id" : "Staten Island", "count" : 969 }
{ "_id" : "Brooklyn", "count" : 6086 }
{ "_id" : "Manhattan", "count" : 10259 }
{ "_id" : "Queens", "count" : 5656 }
{ "_id" : "Bronx", "count" : 2338 }
{ "_id" : "Missing", "count" : 51 }
```

## Filter and Group

```javascript
db.restaurants.aggregate(
   [
     { $match: { "borough": "Queens", "cuisine": "Brazilian" } },
     { $group: { "_id": "$address.zipcode" , "count": { $sum: 1 } } }
   ]
)

{ "_id" : "11368", "count" : 1 }
{ "_id" : "11106", "count" : 3 }
{ "_id" : "11377", "count" : 1 }
{ "_id" : "11103", "count" : 1 }
{ "_id" : "11101", "count" : 2 }
```

# MapReduce

```javascript
db.collection.mapReduce(
                         <map>,
                         <reduce>,
                         {
                           out: <collection>,
                           query: <document>,
                           sort: <document>,
                           limit: <number>,
                           finalize: <function>,
                           scope: <document>,
                           jsMode: <boolean>,
                           verbose: <boolean>,
                           bypassDocumentValidation: <boolean>
                         }
                       )
```

## Map Function

```javascript
function() {
   ...
   emit(key, value);
}
```

The map function has the following requirements:

- In the map function, reference the current document as this within the function.
- The map function should not access the database for any reason.
- The map function should be pure, or have no impact outside of the function (i.e. side effects.)
- A single emit can only hold half of MongoDB’s maximum BSON document size.
- The map function may optionally call emit(key,value) any number of times to create an output document associating key with value.

The following map function will call emit(key,value) either 0 or 1 times depending on the value of the input document’s status field:

```javascript
function() {
    if (this.status == 'A')
        emit(this.cust_id, 1);
}
```

The following map function may call emit(key,value) multiple times depending on the number of elements in the input document’s items field:

```javascript
function() {
    this.items.forEach(function(item){ emit(item.sku, 1); });
}
```

## Reduce Function

```javascript
function(key, values) {
   ...
   return result;
}
```

The reduce function exhibits the following behaviors:

- The reduce function should not access the database, even to perform read operations.
- The reduce function should not affect the outside system.
- MongoDB will not call the reduce function for a key that has only a single value. The values argument is an array whose elements are the value objects that are “mapped” to the key.
- MongoDB can invoke the reduce function more than once for the same key. In this case, the previous output from the reduce function for that key will become one of the input values to the next reduce function invocation for that key.
- The reduce function can access the variables defined in the scope parameter.
- The inputs to reduce must not be larger than half of MongoDB’s maximum BSON document size. This requirement may be violated when large documents are returned and then joined together in subsequent reduce steps.



```javascript
reduce(key, [ C, reduce(key, [ A, B ]) ] ) == reduce( key, [ C, A, B ] )
reduce( key, [ reduce(key, valuesArray) ] ) == reduce( key, valuesArray )
reduce( key, [ A, B ] ) == reduce( key, [ B, A ] )
```



## Example



```javascript
{
     _id: ObjectId("50a8240b927d5d8b5891743c"),
     cust_id: "abc123",
     ord_date: new Date("Oct 04, 2012"),
     status: 'A',
     price: 25,
     items: [ { sku: "mmm", qty: 5, price: 2.5 },
              { sku: "nnn", qty: 5, price: 2.5 } ]
}

var mapFunction1 = function() {
                       emit(this.cust_id, this.price);
                   };
var reduceFunction1 = function(keyCustId, valuesPrices) {
                          return Array.sum(valuesPrices);
                      };
db.orders.mapReduce(
                     mapFunction1,
                     reduceFunction1,
                     { out: "map_reduce_example" }
                   )
```
