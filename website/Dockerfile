# Dockerfile

# base image
FROM node:lts-alpine

# create & set working directory
RUN mkdir -p /files
WORKDIR /files

# copy source files
COPY ./web /files
# RUN cd web

# install dependencies
RUN npm install --force

# start app
RUN npm run build
EXPOSE 3000
CMD npm run start
