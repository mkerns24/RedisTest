# RedisTest


## Step3 - Setting up local Redis

### Installation

Followed http://redis.io/topics/quickstart, but make test didn't work under Installing Redis.
Problem was using more than one cpu according to http://serverfault.com/questions/747476/redis-installation-make-test-errors.
I followed the stack overflow response and ran "taskset -c 1 make test".
Tests passed, Installation complete.

### Trying out Redis real quick

cli means command line interface

I went in the src folder and started the server via

```
$ ./redis-server
```
I opened up another terminal window and ran
```
$ redis-cli 
```
to access the server I had just started in the previous terminal window.

I ran "set TestKey TestValue" to set the value of Testvalue to the key TestKey".

I ran "get TestKey" and I got "TestValue" as expected.

I then shutdown redis via shutdown in the cli. The server in terminal window 1 shutdown.

I then exited the cli via "exit".

I restarted the server in the first terminal window via "redis-server" and started the cli again in the second terminal window.

I found when I ran "get TestKey" after having restarted the server, the value I got back was still "TestValue", meaning the server has persistence storage as mentioned in the redis installation guide.

I ran "DEL TestKey" and deleted the key.


## Step4 - Messing with Redis

Going through this http://openmymind.net/2011/11/8/Redis-Zero-To-Master-In-30-Minutes-Part-1/

The commands I executed are under the step4 directory in the log.

### Universal Helpful Commands

FLUSHALL - Delete everything stored
SHUTDOWN - turn off the server
EXIT - leave the CLI


### Strings/Set/Get

set and get seem straight forward.

you set the key to a specific value. "SET Key Value"

Then specify the key to get the value. "GET Key" -> Value

Strings are just a data type that Redis can hold as a value

### Hashes

With hashes we can apparently search and extra amount inside a value.
So we can have the key goku and have some type of property associated with it and give it a value.

"HSET goku power 9001" 

So to get this value we need both the key-goku and the property-power.

"HGET goku power" -> 9001


This next part "SET users:goku {race: 'sayan', power: 9001}" is stupid.

It makes it sound like we could run that and it would work, but it doesn't. We need to convert "{race: 'sayan', power: 9001}" into a byte array so that we can throw that in as a value. 

So if we have a "Key-Property-Value" type of situation its probably easier just to use HSET instead of SET.

### Lists

Yeah, I don't get this part. I don't really see how to run scripts without using something other than Redis.

Will get back to this when we have some other server running.

### Sets 

I didn't know Redis even had this functionality. We can make sets!

#### SADD
SADD - So SADD is like Set-ADD, adds somethign to a set.

```
SADD group1 elementA
// group1{elementA}

SADD group1 elementB
// group1{elementA.elementB}

SADD group2 elementB
// group1{elementA.elementB} group2{elementB}

SADD group2 elementC
// group1{elementA.elementB} group2{elementB,elementC}
```

#### SINTER
SINTER - is like the interpolation of the two sets

```
SINTER group1 group2
// elementB
```

#### SUNION
SUNION - is the union of the two sets

```
SUNION group1 group2
// elementA elementB elementC
```

#### SBlahSTORE

So there is a way to store the answers from the interpolation and the union of the sets.

SUNIONSTORE stores into a set the answer for the SUNION
Similarly SINTERSTORE stores into a set the answer for the SINTER

```
SUNIONSTORE uniongroup group1 group2
// uniongroup{elementA, elementB, elementC}
SINTERSTORE intergroup group1 group2
// intergroup{elementB}
```

#### SMEMBERS

Tells you the members of the group

```
SMEMBERS group1
-elementA, elementB
```

#### SDIFF

Sdiff is really weird. Its like I got group1 with elementA and elementB and then im like whats different about it than group2 which has element B and elementC. Then i see that Oh Group1 has elementA, but both groups have elementB. So then the answer is elementA

```
// group1{elementA.elementB} group2{elementB,elementC}
SDIFF group1 group2
-elementA
SDIFF group2 group1
-elementC
```

#### Sorted Sets

Seems like how all the set stuff started with S----, all the sorted set stuff starts like Z---.

#### ZADD

Adds something into a sorted set with a specific value associated with it.

```
ZADD friends:leto 1000 ghanima
```

#### ZSCORE

Tells you the value associated with a specific key.

```
ZSCORE friends:leto ghanima
-"1000"
```

#### ZRANGE

This one lists out the elements of the set based on their indices

```
ZRANGE friends:leto 0 -1 WITHSCORES
-1) "faradn"
-2) "2"
-3) "duncan"
-4) "994"
-5) "ghanima"
-6) "1000"
```

"0 -1" means index value 0 to index value -1, which means all elements.
Not sure if bounds are inclusive/exclusive.

#### ZRANGEBYSCORE

This one lists out the elements of the set based on their values

```
ZRANGEBYSCORE friends:leto 500 1000
-1) "duncan"
-2) "ghanima"
```

"500 1000" means values held by the elements between 500 and 1000.
Not sure if bounds are inclusive/exclusive.

#### ZINCRBY

Increases the value of an existing element

```
// faradn holds the initial value of 2
ZINCRBY friends:leto 121 faradn
-"123"
```

## Step 5 - MAKING OUR OWN MODULE!!!!

So first thing first, we can't actually make the module with the stable redis version we have.

We need to use the unstable redis version to be able to load modules.

### Part A - Getting that unstable redis version going

I followed the same steps as stated in step3 on http://redis.io/topics/quickstart , but instead of downloading redis-stable with the wget. I just manually downloaded the unstable version here, http://redis.io/download .

I did the whole make, make test.

### Part B - Directory organization

Make the directory like

```
Step4
|- redis-unstable
|- testmodule 
```
where the redis-unstable has the server we just unpacked and made, and testmodule is where we are going to make our module.


### Part C - Making that module

make a file in testmodule called module.c which has

```
#include "redismodule.h"
/* ECHO <string> - Echo back a string sent from the client */
int EchoCommand(RedisModuleCtx *ctx, RedisModuleString **argv, int argc) {
  if (argc < 2) return RedisModule_WrongArity(ctx);
  return RedisModule_ReplyWithString(ctx, argv[1]);
}

/* Registering the module */
int RedisModule_OnLoad(RedisModuleCtx *ctx) {
  if (RedisModule_Init(ctx, "example", 1, REDISMODULE_APIVER_1) == REDISMODULE_ERR) {
    return REDISMODULE_ERR;
  }
  if (RedisModule_CreateCommand(ctx, "example.echo", EchoCommand, "readonly", 1,1,1) == REDISMODULE_ERR) {
    return REDISMODULE_ERR;
  }
}
```

grab the needed header file from redis-unstable/src/

```
//from the testmodule directory
$ cp ../redis-unstable/src/redismodule.h .
```

compile it all

```
$ gcc -fPIC -std=gnu99 -c -o module.o module.c
$ ld -o module.so module.o -shared -Bsymbolic -lc
```
for MAC OS X the ld command is
```
$ ld -o module.so module.o -bundle -undefined dynamic_lookup -lc
```

module.so has what we need now

### Part D - Run the server with the module

get to the src folder for our redis-unstable

Start the server with the module

```
$ ./redis-server --loadmodule ../../testmodule/module.so
```

Start the cli in another terminal window
```
$ ./redis-cli
```

Try it out the module with the cli

```
127.0.0.1:6379> example.echo test
"test"
```

Success!
