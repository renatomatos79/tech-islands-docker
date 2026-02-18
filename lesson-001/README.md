# Lesson 001 - Docker Basics

Hello dockers,

## 1. Open Docker Playground

Open your browser and access:

<https://labs.play-with-docker.com/>

1. Click **Login**.
2. Click **Start**.
3. On the **Instances** page, click **+ Add new instance**.

---

## 2. Check Docker Version

```bash
docker --version
```

---

## 3. Run Your First Container

```bash
docker run hello-world
```

Run it again to confirm the image is now local:

```bash
docker run hello-world
```

---

## 4. Run Alpine in Interactive Mode

```bash
docker container run -it alpine
```

Inside Alpine:

```bash
ls
```

List all containers (running and stopped):

```bash
docker container ls -a
```

Start and attach to a stopped container:

```bash
docker container start -ai funny_brattain
```

Create a named container:

```bash
docker container run --name myalpine -it alpine
```

Trying to reuse the same name shows a conflict:

```bash
docker container run --name myalpine -it alpine
```

---

## 5. Run Nginx with Port Mapping

Expose container port `80` on host port `8080`:

```bash
docker container run -d -p 8080:80 nginx
```

Find your Nginx container:

```bash
docker container ls -a
docker container ls -a | grep nginx
```

Test Nginx from localhost:

```bash
curl http://localhost:8080
```

---

## 6. Monitor and Inspect Containers

Container resource usage:

```bash
docker container stats zealous_lichterman
```

Inspect detailed metadata:

```bash
docker container inspect zealous_lichterman
```

Filter inspect output:

```bash
docker container inspect zealous_lichterman | grep IP
```

> The container IP is internal. Use host IP/localhost for external access.

---

## 7. List and Remove Images

Detailed list:

```bash
docker image ls --no-trunc
```

Short list:

```bash
docker image ls
```

Remove an image (force if needed):

```bash
docker image rm bf756fb1ae65
docker image rm -f bf756fb1ae65
```

---

## 8. Bulk Cleanup

List container IDs:

```bash
docker container ls -a -q
```

Remove all containers:

```bash
docker container rm -f $(docker container ls -a -q)
```

Remove all images:

```bash
docker image rm -f $(docker image ls -q)
```

Thanks.
