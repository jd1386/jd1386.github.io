---
layout: post
title: Mongo Shell에서 자주 쓰이는 명령어
date: 2018-07-30 14:00:00 +0900
categories: blog
---

```shell
$mongo
show dbs // => list databases
use db_name // => connect to db_name
show collections // => list collections
db.collection_name.find() // => list all records in the collection
db.collection_name.insertMany([
  { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "P" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
]) // => bulk insert
db.collection_name.deleteMany({}) // => delete all records in collection_name
db.collection_name.deleteMany({ status: "Delete" }) // => delete all records that match a condition in the collection
db.collection_name.deleteOne({ status: "Delete"}) // => delete one record that matches a condition in the collection
db.dropDatabase() // => drop whole database
```