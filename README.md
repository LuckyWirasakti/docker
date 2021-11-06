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
##### DOCKERFILE
```
FROM python:alpine

ENV PYTHONUNBUFFERED 1

RUN mkdir /code
WORKDIR /code

COPY . .

RUN apk add --no-cache --virtual .build-deps \
    ca-certificates gcc postgresql-dev linux-headers musl-dev \
    libffi-dev jpeg-dev zlib-dev \
    && pip install -r requirements.txt \
    && find /usr/local \
        \( -type d -a -name test -o -name tests \) \
        -o \( -type f -a -name '*.pyc' -o -name '*.pyo' \) \
        -exec rm -rf '{}' + \
    && runDeps="$( \
        scanelf --needed --nobanner --recursive /usr/local \
                | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
                | sort -u \
                | xargs -r apk info --installed \
                | sort -u \
    )" \
    && apk add --virtual .rundeps $runDeps \
    && apk del .build-deps

EXPOSE 8000
ENTRYPOINT ["sh", "entrypoint.sh"]
```
##### ENTRYPOINT
```
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
