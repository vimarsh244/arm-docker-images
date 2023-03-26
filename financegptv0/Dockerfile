# syntax=docker/dockerfile:1

FROM python:3.9-slim-bullseye

RUN mkdir -p /files
WORKDIR /files

# copy source files
COPY ./backend/flask /files

RUN pip3 install flask pydantic openai langchain gunicorn faiss-cpu


CMD [ "python3 main.py"]
