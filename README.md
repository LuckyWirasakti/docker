##### BUILD IMAGE
```
docker build -t wirasakti .
```
##### RUN CONTAINER
```
docker run --rm -it --name wirasakti_site \
    -v "$PWD:/code/" \
    -p 80:8000 \
    wirasakti
```
---
##### DJANGO FRAMEWORK
##### DOCKERFILE
```
FROM python:alpine

WORKDIR /usr/src/app

ENV PYTHONUNBUFFERED 1
ENV PYTHONDONTWRITEBYTECODE 1

RUN apk update \
    && apk add postgresql-dev gcc python3-dev musl-dev
    
RUN pip install --upgrade pip
COPY ./requirements.txt .
RUN pip install -r requirements.txt

COPY ./entrypoint.sh .
RUN sed -i 's/\r$//g' /usr/src/app/entrypoint.sh
RUN chmod +x /usr/src/app/entrypoint.sh

COPY . .

EXPOSE 8000
ENTRYPOINT ["sh", "entrypoint.sh"]
```
##### ENTRYPOINT
```
while ! nc -z 127.0.0.1 5432; do
  sleep 0.1
done
python manage.py test --no-input
python manage.py migrate --no-input
python manage.py runserver 0.0.0.0:8000
```
---
##### POSTGRESQL
```
docker run --rm -it \
    --name wirasakti_db \
    -e POSTGRES_DB=wirasakti \
    -e POSTGRES_USER=wirasakti \
    -e POSTGRES_PASSWORD=mysecretpassword \
    -e PGDATA=/var/lib/postgresql/data/pgdata \
    -v "$PWD/data:/var/lib/postgresql/data" \
    -p 5432:5432 \
postgres:alpine
```
---
##### REDIS
```
docker run --rm -it \
    --name wirasakti_redis \
    -v "$PWD/redis:/data" \
    -p 6379:6379 \
redis:alpine
```
