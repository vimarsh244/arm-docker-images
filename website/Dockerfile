# Dockerfile

# base image
FROM node:lts-alpine

# FROM satantime/puppeteer-node:16-bullseye

#dependency packages
# RUN sudo apt install chromium -y



# Installs latest Chromium package.
RUN echo "http://dl-cdn.alpinelinux.org/alpine/edge/main" > /etc/apk/repositories \
    && echo "http://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories \
    && echo "http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories \
    && echo "http://dl-cdn.alpinelinux.org/alpine/v3.12/main" >> /etc/apk/repositories \
    && apk upgrade -U -a \
    && apk add \
    libstdc++ \
    chromium \
    harfbuzz \
    nss \
    freetype \
    ttf-freefont \
    font-noto-emoji \
    wqy-zenhei \
    && rm -rf /var/cache/* \
    && mkdir /var/cache/apk




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
