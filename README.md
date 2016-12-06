# Machine Learning Experimenter

## Introduction

The Machine Learning Experimenter is a web application to more easily gain
experience with machine learning techniques. It has been developed with
deep neural networks and Tensorflow in mind. It let you launch cloud compute
instances (currently AWS is supported), execute code on them, and see the
results as they are produced. By easily running variations of the same model
in parallell on several machines, a human learner can quickly gain experience
with the effects of hyper parameters and nuances of machine learning techniques.

The system is composed of

1. a backend web server, written in NodeJS, that hosts the frontend client, orchestrates
all running machines, and serves the API for the frontend client and the machines. (https://github.com/rrostt/ml-experimenter-server)
2. a frontend web application written in React that talks to the backend API. (https://github.com/rrostt/ml-experimenter-frontend)
3. workers written in NodeJS that communicate with the backend, syncs files,
and executes commands. (https://github.com/rrostt/ml-experimenter-worker)

This particular repo (where you read this README) consists of
a docker-compose project for running the backend, host the backend, serve
a mongodb-store.


## Quick Start

```
git clone https://github.com/rrostt/ml-experimenter-containers
cd ml-experimenter-containers
git clone https://github.com/rrostt/ml-experimenter-frontend frontend
git clone https://github.com/rrostt/ml-experimenter-server server
docker exec mlexperimentercontainers_app_1 sh -c "/src/node add-user.js user password"
```

If successful, use browser to go to https://localhost:8443/.


## Get Started

*Note:* Workers connect to the backend, and thus the backend server must be
addressable by the workers. This typically means that if you want to run
workers on AWS instances, the server must have a public IP. There are plans to
maybe change this restriction.

To launch the project using `docker-compose up` you must first make sure 3
folders are setup:

- frontend/ pointing to the frontend repo folder
- server/ pointing or containing the server repo
- ssl/ containing domain.crt and domain.key

Edit frontend/app/config.js and server/config.js to set the correct host address.

Once setup run `docker-compose up` and you should be able to connect using
a web browser to the host on port 8443 (e.g. https://localhost:8443).

Before being able to login, you must create a user in the db. Easiest way to
create user *user* with password *password* is by running

`docker exec mlexperimentercontainers_app_1 sh -c "/src/node add-user.js user password"`

This assumes that docker-compose named the app container mlexperimentercontainers_app_1.
Run `docker ps` to verify.

You can now login using the new user.


### To create ssl keys

`openssl req \
       -newkey rsa:2048 -nodes -keyout domain.key \
       -x509 -days 365 -out domain.crt`


## Usage

Logged in for the first time, you have an empty project and no running machines.
You have 4 tabs and a Logout button at the top of the screen. The Code tab let
you work with code. The Machine tab let you work with and inspect machines. The
graph tab will show graphs drawn based on data printed by running machines.

To get started create a new file using the '+ Add File' at the bottom of the
Files column on the Code tab. Name it e.g. run.py. Write `print "hello world"`
in the code editor.

Next we need to add a machine to run this code. If you want to run it on an AWS
instance then go to settings and add your AWS Access Key ID and Secret Key ID.
To get those go to the AWS Management Console and choose My Security Credentials
under your name at the top right. If you create a new user be sure to add the
proper Permissions to the user.

To launch an AWS instance once you have saved the settings, go to the Machines
tab and click '+ Launch Instance'. This will ask you to pick an AMI and an
instance type. Go with default to get started as this will be easiest and
cheapest (free if you're still on your free tier allowance). This will launch
the instance. The interface won't update automatically so you will need to click
the refresh button next to 'AWS Instances' to know when it is ready. It will be
when the 'run machine' button appears. Click 'run machine' and wait for the
machine to start. (When AWS instances launch they may take some time to get
  ready. Before they are ready 'run instance' may fail to connect and give
  you an error. Try again until it succeeds.).

To run a machine locally check the ml-experimenter-worker repo.

Now we have a machine running where we can run the code. Go back to the Code
tab and click the 'go' button on the machine on the right, with the 'run.py'
file selected. This will copy all files in the project to the machine, and run
the code you had selected in the code editor when you clicked 'go'. Under the
Machines tab you can now select the machine and see what the code prints.
Hopefully you will see

```
Running run.py
hello world
```

You have now created a program and run it on a machine. Now, create more
machines, write more code, and go crazy with experimentation and learning
about machine learning.


## Architecture

A web interface allows you to write code, and to run the code on separate *machines*.
A *machine* is a running instance of *ml-experimenter-worker* and is typically
run on a cloud instance such as AWS EC2. A server backend handles the
communication between the machines and the web interface through an API.

A machines start it makes a websocket connection to the server and presents itself. Each
worker maintains its own state and communicates its change to the server through
events. The worker listens for events and responds with events. The main
events are `run` and `sync`. `sync` is accompanied with the list of files
to sync together with the modified time. The worker compares this list with its
own content of CWD, to determine what files to request. It then emits a `fetch`
event on the socket and waits for files. Files are received through `file` event.
The `run` event starts executing the given command through `spawn`. When the
command is running output to stdout results in an `stdout` event, and stderr in
a `stderr` event. An error spawning results in an stderr event.
