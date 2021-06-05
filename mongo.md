
# MongoDB

## MongoDB in Docker

To run a MongoDB in docker:

```text
docker run --net="host" --name mongo -d mongo 
```

## Installing a local MongoDB

If you want to do this you need to add another yum repo.  See:  https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/

## Mongo Express

Mongo Express is a web based tool for doing admin on Mongo databases.  

Run it in Docker using:

```text
docker run --net="host" -e ME_CONFIG_MONGODB_SERVER="localhost" mongo-express
```

Then visit: http://localhost:8081


## Mongo CLI

To access Mongo CLI in Docker:

```text
docker exec -it mongo /bin/bash
mongo
```

Start the client:

	mongo

Show collections:

	show collections

List rows in a collection:

	db.customer.find()

Insert a document:

	db.customer.insert( { "firstName" : "geoff" } )

Select documents where firstName is geoff:
	 
	db.customer.find( { "firstName" : "geoff" } )

Select lastName where firstName is geoff:

	db.customer.find( {"firstName": "Geoff"}, { "lastName": 1 } )

Delete a document where firstName is geoff:

	db.customer.remove( { "firstName" : "geoff"} )

Update lastName where firstName is geoff:

	db.customer.update( { "firstName": "geoff" }, {$set: { "lastName" : "Fish" } }, {multi : true} )

Replace document where ID=X.  This didn't work using firstName as the key.  Maybe the ID is special.

	db.customer.save( { "_id" : ObjectId("59155e6b9b319b0ae0e4d3a9"), "firstName" : "Geoff",  "lastName" : "Danger" } )

Find a document (case insensitive).  For this we need to create an index.  It works.

	db.customer.insert( { "firstName" : "geoff" } )
	db.customer.createIndex({firstName: 1}, {collation: {locale: "en", strength: 2}});
	db.customer.find({firstName: "GEOFF"}).collation({locale: "en", strength: 2});	 

Find document where first name contains "ff", using a regex.  Indexes don't currently support contains.

	db.customer.find( {"firstName" : { $regex : ".*ff.*" } } )

Find document where first name is Geoff using an index (faster):

	db.customer.createIndex( { "firstName" : "text" } )
	db.customer.find( { $text: { $search: "Geoff" } } )

This only works for whole words or phrases.  There's also only one text index allowed per collection.  (But it can span some or all of the fields as desired, so it may work for doing a fairly dumb full text-search)

Limit (paging):

	db.customer.find().limit(2).skip(2)

Sorting:

	db.customer.find().sort( {"firstName": 1, "lastName": 1} )

Count:

	db.customer.find().count()

	db.customer.find( {"firstName" : "Geoff" }).count()
	
Join

Join can be done like this: https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/

I think it is recommended to design database solutions that don't need joins.

References: https://www.tutorialspoint.com/mongodb/

## Spring integration

In [Spring](spring4.md) I put a section on how to use MongoDB.


## Robomongo

This tool gives you a bit of native UI for browsing the database.  

* Download from https://robomongo.org/download
* tar -xvzf <filename.tar.gz>
* run bin/robomongo
* Edit /home/ms/.3T/robomongo/1.0.0/robomongo.json and set textFontFamily=verdana, textFontPointSize=10


## Backup and restore

This is easily done with the mongodump and mongorestore commands.  See: https://www.tutorialspoint.com/mongodb/mongodb_create_backup.htm



