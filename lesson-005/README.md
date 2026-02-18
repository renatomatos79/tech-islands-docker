# Lesson 005 - Redis Basics in Docker

Hello dockers,

In this lesson we will use Redis in a Docker container and learn to:

- create cache items
- remove cache items
- configure cache expiration (TTL)

## 1. Open Docker Playground

<https://labs.play-with-docker.com/>

1. Create a new instance.
2. Start Redis:

```bash
docker run --name redisserver -d -p 6379:6379 redis
```

---

## 2. Access Redis CLI

Attach to container:

```bash
docker exec -it redisserver bash
```

Inside the container:

```bash
cd /usr/local/bin
./redis-cli
```

---

## 3. Create and Read Cache Data

Set value:

```redis
mset list "a, b, c, d, e"
```

Check TTL:

```redis
ttl list
```

> `-1` means the key does not expire.

Get value:

```redis
mget list
```

---

## 4. Set Expiration

Set key expiration to 20 seconds:

```redis
expire list 20
```

After 20 seconds, retrieve again:

```redis
mget list
```

Expected: `nil` (expired key).
