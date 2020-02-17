From python:latest

RUN mkdir /app

COPY . /app
WORKDIR /app

RUN pip install -r requirements.txt
RUN apt-get install build-essential
RUN make
RUN openssl req \
    -new \
    -newkey rsa:4096 \
    -days 365 \
    -nodes \
    -x509 \
    -subj "/C=TW/ST=Taiwan/L=Taipei/O=GCP/CN=domainjoinapi.c.PROJECT_ID.internal" \
    -keyout server.key \
    -out server.crt

CMD ["python", "main.py"]