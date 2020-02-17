From python:latest

RUN mkdir /app

COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt


# 生成私鑰，按照提示填寫內容
RUN openssl genrsa -des3 -out server.key 1024

# 生成csr檔案 ，按照提示填寫內容
RUN openssl req -new -key server.key -out server.csr

# Remove Passphrase from key
RUN cp server.key server.key.org 
RUN openssl rsa -in server.key.org -out server.key

# 生成crt檔案，有效期1年（365天）
RUN openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

CMD ["python", "main.py"]