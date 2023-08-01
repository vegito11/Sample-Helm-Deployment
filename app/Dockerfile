FROM python:3.9-slim-buster

ENV PYTHONPATH /app

WORKDIR /app

RUN apt-get update && apt-get install gcc -y

COPY requirements.txt .

RUN pip install -r requirements.txt

RUN export 

COPY . .

CMD ["python", "userapi.py"]