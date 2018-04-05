# Redis playground 

I create this project to learn and get exp about redis

## Environment and tools:
OS: Ubuntu 16
Tools: Redis + Redis Commander

## How to install Redis

These instructions will guide how to install Redis in Linux (Ubuntu)

In order to get the latest version of Redis, we will be compiling and installing the software from source. Before we download the code, we need to satisfy the build dependencies so that we can compile the software.
To do this, we can install the build-essential meta-package from the Ubuntu repositories. We will also be downloading the tcl package, which we can use to test our binaries.
We can update our local apt package cache and install the dependencies by typing:

```
sudo apt-get update
sudo apt-get install build-essential tcl
```

Since we won't need to keep the source code that we'll compile long term (we can always re-download it), 
we will build in the /tmp directory. Let's move there now:
```
cd /tmp
```
Now, download the latest stable version of Redis. This is always available at a stable download URL:
```
curl -O http://download.redis.io/redis-stable.tar.gz
```
Unpack the tarball by typing:
```
tar xzvf redis-stable.tar.gz
```
Move into the Redis source directory structure that was just extracted:
```
cd redis-stable
```
Build and Install Redis
Now, we can compile the Redis binaries by typing: make

After the binaries are compiled, run the test suite to make sure everything was built correctly. You can do this by typing:
```
make test
```
This will typically take a few minutes to run. Once it is complete, you can install the binaries onto the system by typing:
```
sudo make install
```

Configure Redis

Now that Redis is installed, we can begin to configure it.

To start off, we need to create a configuration directory. We will use the conventional /etc/redis directory, which can be created by typing:
```
sudo mkdir /etc/redis
```
Now, copy over the sample Redis configuration file included in the Redis source archive:
```
sudo cp /tmp/redis-stable/redis.conf /etc/redis
```
Next, we can open the file to adjust a few items in the configuration:
```
sudo nano /etc/redis/redis.conf
```
In the file, find the supervised directive. Currently, this is set to no. Since we are running an operating system that uses the systemd init system, we can change this to systemd:
```
/etc/redis/redis.conf
```
```
. . .

# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.


remove this line"supervised systemd"
replace by"supervised systemd"

. . .
```

Next, find the dir directory. This option specifies the directory that Redis will use to dump persistent data. We need to pick a location that Redis will have write permission and that isn't viewable by normal users.

We will use the /var/lib/redis directory for this, which we will create in a moment:

/etc/redis/redis.conf
```
. . .

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.


change this line: 
dir /var/lib/redis

. . .
```
Save and close the file when you are finished.

----------------------------------------
Create a Redis systemd Unit File
Next, we can create a systemd unit file so that the init system can manage the Redis process.

Create and open the /etc/systemd/system/redis.service file to get started:
```
sudo nano /etc/systemd/system/redis.service
```
```
[Unit]
Description=Redis In-Memory Data Store
After=network.target

[Service]
User=redis
Group=redis
ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf
ExecStop=/usr/local/bin/redis-cli shutdown
Restart=always

[Install]
WantedBy=multi-user.target
```
Save and close the file when you are finished.
------------------------------
Create the Redis User, Group and Directories
Now, we just have to create the user, group, and directory that we referenced in the previous two files.

Begin by creating the redis user and group. This can be done in a single command by typing:
```
sudo adduser --system --group --no-create-home redis
```
Now, we can create the /var/lib/redis directory by typing:
```
sudo mkdir /var/lib/redis
```
We should give the redis user and group ownership over this directory:
```
sudo chown redis:redis /var/lib/redis
```
Adjust the permissions so that regular users cannot access this location:
```
sudo chmod 770 /var/lib/redis
```
Start and Test Redis
Now, we are ready to start the Redis server.

Start the Redis Service
Start up the systemd service by typing:
```
sudo systemctl start redis
sudo systemctl status redis
```
You should see something that looks like this:
```
Output
● redis.service - Redis Server
   Loaded: loaded (/etc/systemd/system/redis.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2016-05-11 14:38:08 EDT; 1min 43s ago
  Process: 3115 ExecStop=/usr/local/bin/redis-cli shutdown (code=exited, status=0/SUCCESS)
 Main PID: 3124 (redis-server)
    Tasks: 3 (limit: 512)
   Memory: 864.0K
      CPU: 179ms
   CGroup: /system.slice/redis.service
           └─3124 /usr/local/bin/redis-server 127.0.0.1:6379       

. . .
```
Test the Redis Instance Functionality
To test that your service is functioning correctly, connect to the Redis server with the command-line client:
```
redis-cli

set test CHICKEN
Output
OK
Now, retrieve the value by typing:

get test
You should be able to retrieve the value we stored:

Output
CHICKEN
Exit the Redis prompt to get back to the shell:

exit
As a final test, let's restart the Redis instance:
```

#References:
* https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-redis-on-ubuntu-16-04

---------------------------------------------------------------------------------------

## Redis Commander

https://joeferner.github.io/redis-commander/
Redis Commander is a Node.js web application that can be used to view, edit and manage your Redis databases from the comfort of your browser. It allows you to directly manipulate all of Redis’ data types. It’s freely available (although it doesn’t specify under which license) and can be easily installed via npm, provided you have a working node.js installation.

Pros: it’s free, powerful, in your browser and runs wherever Node.js is.
Cons: requires direct connectivity, only runs where Node.js is.

You should install npm before take the installation
```
npm install -g redis-commander
```
Run by
```
redis-commander
```
If you haven't install node or got problem 
```
/usr/bin/env: ‘node’: No such file or directory
```
Run this
```
apt-get install nodejs-legacy
```
--> Now your redis commander run on http://localhost:8081/
