# syntax=docker/dockerfile:1

FROM python:3.9

RUN mkdir -p /files

# copy source files
COPY ./backend/flask /files

WORKDIR /files


RUN pip install -r requirements.txt

# ENTRYPOINT [ "python" ]


CMD [ "gunicorn -b 0.0.0.0 main:app"]
