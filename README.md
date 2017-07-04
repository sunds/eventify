[![Build Status](https://travis-ci.org/eventifyio/eventify.svg?branch=master)](https://travis-ci.org/eventifyio/eventify)
[![Coverage Status](https://coveralls.io/repos/github/morissette/eventify/badge.svg?branch=master)](https://coveralls.io/github/morissette/eventify?branch=master)
[![Documentation Status](https://readthedocs.org/projects/eventify/badge/?version=latest)](http://eventify.readthedocs.io/en/latest/?badge=latest)


# Eventify
A lightweight python module for building event driven distributed systems.

## Installation
```bash
pip install eventify
```

## Problem
Developers need a easy and fast way to implement business logic in a modern asynchronous way. Developers should not have to worry about service communication.

## Contributing
TBD

## Architecture
![ArchDiagram](https://raw.githubusercontent.com/eventifyio/eventify/master/docs/eventify.png)

## Tutorial
Below is everything you need to get started using the eventify project.

### Minimum Viable Service
Create an Event handler function to receive events.  To publish events, use session.emit_event() within the Event handler function. To start your service (and begin listening for events), pass the Event handler function as a callback to run a Service.

```python
#!/usr/bin/env python
"""
Minimal viable microservice
"""
import logging
import sys

from eventify.event import Event
from eventify.service import Service


FORMAT = '%(asctime)-15s %(name)s %(levelname)s %(message)s'
logging.basicConfig(stream=sys.stdout, format=FORMAT, level=logging.DEBUG)
logger = logging.getLogger('example.service')

def my_example_event_handler(event, session=None):
    """
    Example event handler - This function is implemented as
    a callback method to the eventify service
    :param event: An event dictionary
    :param session: Access to the underlying methods of eventify
    """
    # Parse Incoming Event from Dict to Object
    event = Event(event)

    # See the event that was received
    logger.debug("received event %s", event.name)


    # Create and Publish an Event
    # you can also use the publish method directly
    new_event = Event({
        "name": "ReceivedEvent",
        "message": "Event received by consumer",
        "trace_id": event.trace_id
    })
    session.emit_event(new_event)


def run():
    logger.debug('service started')
    service = Service(
        config_file='config.json',
        callback=my_example_event_handler
    ).start()

if __name__ == '__main__':
    run()
```

### Minimum Configuration - config.json
Each application (service, microservice, etc.) can publish to 0 or 1 topic and subscribe to 0 or more topics. Thus each application requires its own configuration file which defines its publications & subscriptions. Each configuration file also specifies the location of the transport host.

```json
{
    "_comment": "service configuration",
    "name": "example",
    "image": "example/service",
    "driver": "crossbar",
    "transport_host": "ws://events-server:8080/ws",
    "pub_options": {
        "acknowledge": true,
        "retain": true
    },
    "publish_topic": {
        "topic": "example",
        "timeout": 20,
        "reply_in": 0
    },
    "subscribed_topics": [
        "ui-events"
    ],
    "replay_events": true,
    "replay_type": "event_store"
}
```

### Event History Support
To add event history support add the @event_tracker decorator to any event handler your define.

### Environment variables
* EVENT_DB_HOST

Hostname of database server
* EVENT_DB_USER

Username for database server
* EVENT_DB_PASS

Password for username
* EVENT_DB_TYPE

Database driver type - defaults to postgres
* EVENT_DB_POOL_SIZE

Database connection pool size - defaults to 5

* EVENT_TRACKING_HOST

For use with replay functionality - host of in memory cache

## Supported Drivers
* Crossbar

## Crossbar
Quickest way to get started with crossbar is to use docker; both redis and postgresql are required for a minimal working service.

### Running Crossbar
```bash
docker run -d -p 8080:8080 -it crossbario/crossbar:latest
```
### Running Redis
```bash
docker run -d -p 6379:6379 -t redis:latest
```

### Running Postgresql
```bash
docker run --name postgres-container -p 5432:5432 -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```
Note: This does not have a volume or persistant storage so in a production environment you would want to configure a volume

### See More
http://crossbar.io

## Pypi
https://pypi.python.org/pypi/eventify

## Read The Docs
http://eventify.readthedocs.io/en/latest/
