# RxDB Databae Server
=====================

Welcome to the RxDB Database Server source code tree!
RxDB is simple Key-Value store. It's supports basic Key Value operations
via wellknown protocols, and something more. 


Build and run
-------------

	$ git clone git@github.com:dvarkin/rxdb.git

	$ cd rxdb

	$ make

	$ make tests

	$ make run

Generate souce code Docs
----------

Edocs available in Markup format. To generate source code documenation type in shell:

      $ make edoc

By default RxDB aquire 3 ports:

   - 8080 for HTTP/REST

   - 4444 for UDP

   - 5555 for TCP

TODO: Move port configuration to .config file

## Protocol Overview

RxDB binary protocol is JSON based. 

### Actions

You can send commands to RxDB via TCP/UDP protocols or use HTTP/REST. 

#### GET

Get uniqe key from store:


      {
      "action": "get", "key": "myKey"
      }     		

     
#### PUT

Put unique key and value to store. If key is already exists in the store, it will be replaced with new value.


      {
      "action": "put", "key": "myKey", "value": "newValue"
      }     		


#### PUT with expire

Put key and value to store with expire in **seconds**. You can get this key after expiration.


      {
      "action": "put", "key": "myKey", "value": "newValue", "expire": 5
      }     		

     
#### DEL

Delete key from store


      {
      "action": "del", "key": "myKey"
      }     		


#### SUB (TCP only)

Subscribe to changes of particular key. 


      {
      "action": "sub", "key": "myKey"
      }     		


#### UNSUB (TCP only)

Unsubscribe to changes of particular key

    
      {
      "action": "unsub", "key": "myKey"
      }     		


## HTTP/Rest Protocol


RxDB supports **Swagger** notation for API. Swagger documentation and REST demo available on *http://localhost:8080/api-docs/index.html*.

### Get Key


   curl -X GET --header "Accept: text/plain" "http://localhost:8080/rest/MyKey"


### Put Key and Value



   curl -X PUT --header "Content-Type: text/plain" --header "Accept: text/plain" -d "KeyValueExample" "http://localhost:8080/rest/MyKey"


### Put Key and Value with expiration.

Expiration is an optional argument.


   curl -X PUT --header "Content-Type: application/json" --header "Accept: text/plain" -d "asd" "http://localhost:8080/rest/a?expire=5"

   
### Delete Key


   curl -X DELETE --header "Accept: text/plain" "http://localhost:8080/rest/MyKey"



## UDP Protocol

For test propouses, RxDB contain simple UDP client. You can try to play with RxDB via test UDP client:

### Get Key
   

   (rxdb@127.0.0.1)1> rxdb_raw_client:udp(<<"{\"key\":\"a\",\"action\":\"get\"}">>).
   <<"[]">>


### Put Key


    (rxdb@127.0.0.1)2> rxdb_raw_client:udp(<<"{\"value\":\"Val\",\"key\":\"a\",\"action\":\"put\"}">>).
    <<"\"ok\"">>


### Put with expiration



    (rxdb@127.0.0.1)2> rxdb_raw_client:udp(<<"{\"expire\":1,\"value\":\"Val\",\"key\":\"a\",\"action\":\"put\"}">>).


### Delete Key


    (rxdb@127.0.0.1)4> rxdb_raw_client:udp(<<"{\"key\":\"a\",\"action\":\"del\"}">>).
    <<"\"ok\"">>


## TCP Protocol
   
You can try to insert Rx commands via telnet:

### Put Key

```
$ telnet 127.0.0.1 5555
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
 

{"action":"put","key":"MyKey","value":"MyValue"}
"ok"

```

### Get Key

```
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
{"action":"get","key":"MyKey"}
{"value":"MyValue","key":"MyKey"}
```

### Subscribtion 

RxDB supports receiving updates of current Key via subscribtions.

**Subscribtions supprots only with TCP protocol**

```
Connected to localhost.
Escape character is '^]'.
{"action":"sub","key":"MyKey"}
"ok"
```

Now you can open another terminal, and Update "MyKey" value.

```
$ telnet localhost 5555
Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
{"action":"put","key":"MyKey","value":"Receive via subscribtion"}
"ok"
```

In first terminal you will get:

```
{"value":"Receive via subscribtion","key":"MyKey","action":"upd"}
```

And now you can unsubscribe:

```
{"action":"unsub","key":"MyKey"}
"ok"
```

TODO: monitor TCP sockets, and cleaning subscribtion table from dead ports. 