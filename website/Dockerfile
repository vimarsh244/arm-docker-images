# Dockerfile

# base image
FROM node:lts-alpine

# FROM satantime/puppeteer-node:16-bullseye

#dependency packages
# RUN sudo apt install chromium -y

# create & set working directory
RUN mkdir -p /files
WORKDIR /files

# copy source files
COPY ./web /files
# RUN cd web

RUN npm i puppeteer

# install dependencies
RUN npm install --force

# start app
RUN npm run build
EXPOSE 3000
# CMD npm run build && npm run start
CMD npm run start
