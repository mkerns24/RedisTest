# RedisTest


## Step3 - Setting up local Redis

### Installation

Followed http://redis.io/topics/quickstart, but make test didn't work under Installing Redis.
Problem was using more than one cpu according to http://serverfault.com/questions/747476/redis-installation-make-test-errors.
I followed the stack overflow response and ran "taskset -c 1 make test".
Tests passed, Installation complete.

### Trying out Redis real quick

cli-command line interface

I went in the src folder and started the server via redis-server.
I opened up another terminal window and ran redis-cli to access the server I had just started in the previous terminal window.

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


