# RedisTest


## Step3 - Setting up local Redis

### Installation

Followed http://redis.io/topics/quickstart, but make test didn't work under Installing Redis.
Problem was using more than one cpu according to http://serverfault.com/questions/747476/redis-installation-make-test-errors.
I followed the stack overflow response and "taskset -c 20 make test".
Tests passed, Installation complete.

###
