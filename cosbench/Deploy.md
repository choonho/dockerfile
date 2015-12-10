# Environment

Keyword | Value
----    | -----
MARATHON| 127.0.0.1:8080
SHARED_DIR | /root/user/choonho
NUM_OF_DRIVERS | 10


# Deploy Cosbench Driver nodes

## Create cosbench-driver.json

edit /tmp/cosbench-driver.json
~~~text
{
  "id": "cosbench-driver",
  "cpus": 1,
  "mem": 512,
  "instances": ${NUM_OF_DRIVERS},
  "cmd": "/usr/bin/supervisord",
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "sunshout/cosbench",
      "network": "BRIDGE",
      "portMappings": [
        { "containerPort": 18088, "hostPort": 0 }
      ]
    }
  }
}
~~~

## Deploy drivers

~~~bash
curl -X POST http://${MARATHON}/v2/apps -d @/tmp/cosbench-driver.json -H "Content-type: application/json"
~~~

## Check deployed instances

To make the controller.conf which includes all drivers,
we have to find drivers URIs

~~~bash
curl -X GET http://${MARATHON}/v2/apps/cosbench-driver/tasks
~~~

edit /tmp/cosbench-driver.py

~~~text
URL='http://${MARATHON}/v2/apps/cosbench-driver/tasks'
NUM_OF_INSTANCES=${NUM_OF_DRIVERS}
import urllib2
import json


f = urllib2.urlopen(URL)
j = json.load(f)

print """
[controller]
drivers = %d
log_level = INFO
log_file = log/syslog.log
archive_dir = archive

""" % NUM_OF_INSTANCES

tasks = j['tasks']
count = 1
for task in tasks:
    print """
[driver%d]
name = driver%d
url = http://%s:%s/driver
""" % (count, count, task['host'],task['ports'][0])
    count = count + 1
~~~

### create cosbench.conf

~~~bash
python /tmp/cosbench-driver.py > ${SHARED_DIR}/controller.conf
~~~ 


# Deploy COSBench Controller

## Create cosbench-controller.json

edit /tmp/cosbench-controller.json
~~~text
{
  "id": "cosbench-controller",
  "cpus": 1,
  "mem": 512,
  "cmd": "/usr/bin/supervisord",
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "sunshout/cosbench",
      "network": "BRIDGE",
      "portMappings": [
        { "containerPort": 19088, "hostPort": 0 }
      ]
    },
    "volumes": [
       {
            "containerPath": "/usr/local/bin/cosbench/conf/controller.conf",
            "hostPath": "/var/lib/mesos/user/choonho/controller.conf",
            "mode": "RO"
        }
    ]
  }
}
~~~

## Deploy controller

~~~bash
curl -X POST http://${MARATHON}/v2/apps -d @/tmp/cosbench-controller.json -H "Content-type: application/json"
~~~

