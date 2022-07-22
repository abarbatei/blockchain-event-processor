# blockchain-event-processor

## General description

_blockchain-event-processor_ is designed as a system that indexes events for a given contract. 
The indexed data is then retrievable via a web API for normal use cases. 
If more complex queries are required, the data consumer will have the option, under certain conditions, to directly query the database.

## System structure

The project will be split into the following components:

- [bevents-scraper](https://github.com/abarbatei/bevents-scraper) - component will handle reading events from a specific smart contract and passing them along to whoever is listening
- [bevents-index-processor](https://github.com/abarbatei/bevents-index-processor) - receives the events and saves plus indexes them into a persistence mechanism for later use
- [bevents-server-api](https://github.com/abarbatei/bevents-server-api) - a webserver API for general use

Communication between components is done via RabbitMQ queues. 
Data persistence is obtained using a MongoDB database.

The following diagram indicates how system components interact between each other.
![Component interaction](res/blockchain-event-processor-architecture.jpg)

By constructing the system in this manner the _bevent-scraper_ component is the only one that needs to interact with the blockchain.
Also, by using a message broker topic type publish combined with the component ability to be configured to process multiple events from multiple contracts  
we can achieve both horizontal scaling (more scrapers can be deployed each with their own event to filter if needed)
or vertical scaling (more events for one scraper that are handler by multiple consumers).

Any consumer (in our case _bevents-index-processor_) can choose to subscribe to that routing key, having its own queue and processing what events it desires.
Other consumers can be added to the same queue if need be, to accelerate processing.
Also, each consumer can choose how it processes the events without interaction with other consumers if they choose to use a different queue (which in our case acts like a buffer).

The _bevents-server-api_ and the MongoDB is mostly just used as a POC to showcase how a simple storing/indexing can be achieved and provided to users.

## Setup

In order for the system to fully work each setup for each individual component must be done.
Additionally, this also implies having a RabbitMQ server and a MongoDB running and accessible in their respective networks.

#### Setting up RabbitMQ 

The easiest way to host a RabbiMQ would be via Docker.
See here for a [RabbitMQ docker container](https://hub.docker.com/_/rabbitmq/). 

I recommend using the tags with the management console that allows user logging into a console to inspect queue and server state.
- these are the `X-managenet` tag versions.
- management terminal is opened on port *15672* by default
- default RabbitMQ port for reading/receiving messages is *5672*
- remember to forward them when running the docker

Example rabbit server start
```
docker run -d --hostname mng-rabbit --name mngmt-active-rabbit -e RABBITMQ_DEFAULT_USER=user -e RABBITMQ_DEFAULT_PASS=password -p 15672:15672 -p 5672:5672 rabbitmq:3-management
```

When running the [bevents-scraper](https://github.com/abarbatei/bevents-scraper) project will publish messages on 
a specific routing key linked to a `topic` type exchange. 
If the user provided to _bevents-scraper_ cannot create exchanges, please create them manually as an admin.

### Setting up MongoDB 

For a quick MongoDB server, again, we can use a docker container.
You can get MongoDB containers from https://hub.docker.com/_/mongo

Example starting a mongo container
```
docker run --name events-mongo -p 27017:27017 -d mongo:4.4
```
The exact schema, databases, collections and indexes to be setup are somewhat custom to each implementation.
Our case would need the setup indicate in [bevents-index-processor MongoDB setup](https://github.com/abarbatei/bevents-index-processor#mongodb-setup-and-interaction)

#Credits

The 2 custom images used in the system component interaction diagram are royalty free provided by 
[Shubham Dhage](https://unsplash.com/@theshubhamdhage?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText") on [Unsplash](https://unsplash.com/s/photos/blockchain?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
and by [Gerd Altmann](https://pixabay.com/users/geralt-9301/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=3938432) from [Pixabay](https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=3938432).

The small, orange image is from the RabbitMQ project.