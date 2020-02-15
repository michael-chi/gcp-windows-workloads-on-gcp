From python:latest

RUN mkdir /app

COPY . /app
WORKDIR /app
RUN apt-get install python-
RUN pip install -r requirements.txt

CMD ["python", "main.py"]