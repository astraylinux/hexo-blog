title: Use Mongo C Driver To Upsert Data.
tags:
  - server
  - mongo
  - C
  - python
categories:
  - server
date: 2015-07-03 17:28:01
keywords: c server mongo update so
---

[https://github.com/astraylinux/study/tree/master/tools/mongo_c](https://github.com/astraylinux/study/tree/master/tools/mongo_c)

Recently, I have tried to optimize a python module which accepts json string from redis-server and update-insert to Mongodb. The performance of **pymongo** is terrible, and because of the [GIL](https://en.wikipedia.org/wiki/Global_Interpreter_Lock), I even can't improve the performance through multi thread. So, I try to figure out by use C language.

The Mongodb development library of 'C' is [**mongo-c-driver**](https://github.com/mongodb/mongo-c-driver). Install it.

In 'mongo-c-driver', the main data structure is not 'json', It uses 'Bson'([libbson](https://github.com/mongodb/libbson)) to instead. If you want to study libbson through the doc, you could use the command 'yelp' to open it(If you are using a Linux with GUI) or covert the [Mallard](http://projectmallard.org/) to Html.
```bash
yelp-build html *.page
```
<!--more-->

##bson.c
Everything ready, I write a simple program [bson.c](https://github.com/astraylinux/study/blob/master/tools/mongo_c/bson.c) to study libbson first. Focus to the Makefile, we should compile it with the 'mongo-c-driver'.
```bash
MONGO_LIBS = $(shell pkg-config --cflags --libs libmongoc-1.0)
BSON_LIBS = $(shell pkg-config --cflags --libs libbson-1.0)

#include
INCLUDE = /usr/include/:/usr/local/include/libmongoc-1.0/:/usr/local/include/libbson-1.0/

#compiler
CC = gcc

bson.bin:
    $(CC) bson.c -o bson.bin -I $(INCLUDE) $(BSON_LIBS)
```

##libupdate.c
This is the main program that I will use in my project. I compile it to a '.so' file, then I can call the functions by 'C', 'python' or other program languages.

The key is using the 'mongo_updater' pointer as a 'void' pointer,  in this way, I can call all the functions in python without defining any class to fit the 'mongo_updater' structure(I really no idear about it, how can I define a class to fit a structure that contain a mongc_client_t pointer and mongoc_collection_t pointer.).
```c
//This struction include data array and mongo client.collection for
//each thread.(use the struct pointer.)
typedef struct {
	bson_t **rows;
	mongoc_collection_t *collection;
}INTO_PACKAGE;

//Save this struction to avoid connect to server every time.
typedef struct _updater{
	mongoc_client_t *client_pool[THREAD_NUM];
	mongoc_collection_t *collection_pool[THREAD_NUM];
	unsigned thread_num;
}mongo_updater;

//new a mongo_updater
void *new_mongo_updater();

void destroy_mongo_updater(void *v_updater);

//destroy mongo client
void destroy_mongo(void *v_updater);

//init mongo client, collection and  appoint the thread number.
bool init_mongo(void *v_updater, const char *mongo_url, const char *database,\
	const char *collection, const unsigned thread_num);

//main function to upset data to mongo.
//v_updater: is the mongo_updater pointer, make it as 'void *', when it
//	reference by python or other language, define data type will
//	be convenient.
//json_str: json string.
int update2mongo(void *v_updater, const char *json_str);
```

##test.c and test.py
In 'test.c', I have to write a function to achieve 'urlencode'. In 'mongo-c-driver', connect to Mongodb is use a url that contain user name, password, host and auth source. It's url, so some special characters have to be handled by urlencode. I was connecting with Mongodb, then, I read some data from 'test.data' and converted to a Bson object, then update to Mongodb at last.

In test.py, The 'ctypes' module is necessary, then I load the 'libupdate.so' and redefine the functions by python, then call them. It's easy.
```python
#!/usr/bin/env python
# encoding: utf-8
import urllib
import json
import fileinput
from ctypes import CDLL, c_void_p, c_char_p, c_uint, c_void_p, c_bool, c_int

libupdater = CDLL("./libupdate.so")

new_mongo_updater = libupdater.new_mongo_updater
new_mongo_updater.argtypes = []
new_mongo_updater.restype = c_void_p

destroy_mongo_updater = libupdater.destroy_mongo_updater
destroy_mongo_updater.argtypes = [c_void_p]
destroy_mongo_updater.restype = c_void_p

init_mongo = libupdater.init_mongo
init_mongo.argtypes = [c_void_p, c_char_p, c_char_p, c_char_p, c_uint]
init_mongo.restype = c_bool

update2mongo = libupdater.update2mongo
update2mongo.argtypes = [c_void_p, c_char_p]
update2mongo.restype = c_int

destroy_mongo = libupdater.destroy_mongo
destroy_mongo.argtypes = [c_void_p]
destroy_mongo.restype = c_void_p
```






