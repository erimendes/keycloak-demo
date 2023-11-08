# syntax=docker/dockerfile:1

ARG NODE_VERSION=21.1.0

FROM node:${NODE_VERSION}-alpine as base

WORKDIR /app
COPY *.json .
COPY ./src ./src
COPY ./middleware ./middleware
RUN npm install
EXPOSE 3000
CMD ["npm", "start"]