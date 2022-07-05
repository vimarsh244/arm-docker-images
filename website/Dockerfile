# Dockerfile

# base image
FROM node:lts-alpine


# create & set working directory
RUN mkdir -p /files
WORKDIR /files

# copy source files
COPY ./web2 /files
# RUN cd web

# install sharp for better image optimization
# RUN npm install sharp # doesnt work 

# install dependencies
RUN npm install --force

# start app
RUN npm run build

EXPOSE 3000
# CMD npm run build && npm run start
CMD npm run start
