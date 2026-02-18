# Lesson 007 - Docker Compose with API + Redis

Hello dockers,

In lesson 006 we connected API and Redis by IP. In this lesson we will use Docker Compose to manage service dependency in a cleaner way.

Compose file reference:  
<https://github.com/renatomatos79/playground/blob/master/lesson-007/docker-compose.yaml>

## 1. Prepare Playground

1. Open Docker Playground: <https://labs.play-with-docker.com/>
2. Create folder and download compose file:

```bash
mkdir dk-compose
cd dk-compose
curl https://raw.githubusercontent.com/renatomatos79/playground/master/lesson-007/docker-compose.yaml >> docker-compose.yaml
cat docker-compose.yaml
```

Expected structure:

```yaml
version: '3'
networks:
  ntw_redis:
services:
  redissrv:
    image: redis:latest
    ports:
      - 6379:6379
    networks:
      - ntw_redis
  api:
    image: renatomatos79/apis:core-docker-api-1.1
    ports:
      - 8001:80
    environment:
      - REDIS_HOST=redissrv:6379
    networks:
      - ntw_redis
    depends_on:
      - redissrv
```

---

## 2. Start Compose

```bash
docker-compose up -d
```

Check running containers:

```bash
docker ps
```

---

## 3. Test API and Cache

Get products:

```bash
curl http://localhost:8001/products
```

Update cache using route `/products/{cache-item}`:

```bash
curl -X PUT http://localhost:8001/products/products -H "content-type: application/json" -d "[{\"Id\":1,\"Name\":\"From cache\",\"Price\":1.5}]"
```

Check response again:

```bash
curl http://localhost:8001/products
```

---

## 4. Stop Compose

```bash
docker-compose down
```

Everything should be working properly.
