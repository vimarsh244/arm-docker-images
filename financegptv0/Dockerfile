# syntax=docker/dockerfile:1

FROM python:3.9-slim-bullseye

RUN mkdir -p /files

# copy source files
COPY ./backend/flask /files

WORKDIR /files


RUN pip3 install flask pydantic openai langchain gunicorn faiss-cpu python-dotenv 


CMD [ "python main.py"]
