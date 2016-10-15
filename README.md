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
