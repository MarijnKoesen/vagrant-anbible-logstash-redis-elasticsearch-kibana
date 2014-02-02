### This is a Vagrant machine based on ansible for a centralised logging server with logstash, redis, elasticsearch and kibana.

The box is based on the centralised best practice from logstash:
http://logstash.net/docs/1.3.3/tutorials/getting-started-centralized

Basically the setup is:

```
+--------------------------------------------------------------+
|                       The logserver box                      |
+--------------------------------------------------------------+
|                            kibana                            |
|                              ^                               |
|                        elasticsearch                         |
|                              ^                               |
|                          logstash                            |
|                              ^                               |
|                            redis                             |
+--------------------------------------------------------------+
       ^                ^               ^                ^
       |                |               |                |
+------------+   +------------+  +------------+   +------------+  
| Some box 1 |   | Some box 2 |  | Some box 3 |   | Some box 4 |
|------------|   |------------|  |------------|   |------------|
|  logstash  |   |  logstash  |  |  logstash  |   |  logstash  |  
+------------+   +------------+  +------------+   +------------+
```

So all your boxes send their logs using logstash to the redis-server on the central server.

The central server uses logstash to fetch the logs from redis, process them and put them in the
final storage service, elasticsearch. 

Then we've got kibana on top for all the analysis and log viewing.



### Testing and hello world

```
$ git clone https://github.com/MarijnKoesen/vagrant-anbible-logstash-redis-elasticsearch-kibana.git
$ cd vagrant-anbible-logstash-redis-elasticsearch-kibana
$ git submodule update --init --recursive
$ vagrant up
```

Congratulations you now got a log server up and running.

To visit kibana go to: http://192.168.23.2/ and login, the default credentials are:

user: 'admin'
password: 'CHANGE ME!'




### Changing the default config

By default redis is bound to 127.0.0.1 for security purposes. 

If you want multiple servers to send logs to redis you can create your own ansible/vars/defaults.yml file in which
you can change the ip redis is bound to so you can send logs from multiple servers.



### Sending data to kibana

This is based on the logstash how-to mentioned above:

Just setup logstash on your servers and point them to the redis server on your centralised logging server.

To simply test this, create a shipper.conf:

    # shipper.conf:
    input {
      stdin {
        type => "example"
      }
    }

    output {
      stdout { codec => rubydebug }
      redis { host => "192.168.23.2" data_type => "list" key => "logstash" }
    }

Now start logstash:

    $ java -jar logstash-1.3.3-flatjar.jar agent -f shipper.conf

This will start a logstash process that listens to everything entered in stdin and 
sends it to the centralised redis server. So you can just type some messages in the 
shell and they will be sent to centralised redis.

Next they'll be processed by logstash in the VM who will send them further to elasticsearch 
after which they can be viewed in kibana.

