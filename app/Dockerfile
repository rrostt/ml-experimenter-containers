# FROM node:alpine
FROM ubuntu:16.04

RUN apt-get update
RUN apt-get install -y curl git
RUN curl -sL https://deb.nodesource.com/setup_7.x | bash -
RUN apt-get install -y nodejs node-gyp

# Create app directory
WORKDIR /src

EXPOSE 1234
CMD [ "npm", "start" ]
