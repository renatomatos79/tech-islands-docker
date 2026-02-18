# Lesson 004 - Push Image to Docker Hub and Run from Playground

Hello dockers,

In lesson 003 we built our first .NET Core image. Now we will:

- publish it to Docker Hub
- run it from Docker Playground

## 1. Create Docker Hub Repository

1. Create a free Docker Hub account:  
   <https://hub.docker.com/>
2. Create a public repository named `apis`:  
   <https://hub.docker.com/repositories>

---

## 2. Tag Local Image

Tag the image created in lesson 003:

```powershell
docker tag core-docker-api:1.0 <your-user>/apis:core-docker-api-1.0
```

Example:

```powershell
docker tag core-docker-api:1.0 renatomatos79/apis:core-docker-api-1.0
```

Confirm tags:

```powershell
docker image ls | grep core-docker-api
```

---

## 3. Login and Push

```powershell
docker login
docker image push <your-user>/apis:core-docker-api-1.0
```

Check uploaded tag:

<https://hub.docker.com/repository/docker/renatomatos79/apis/tags?page=1&ordering=last_updated>

---

## 4. Run from Docker Playground

Open Playground:

<https://labs.play-with-docker.com>

Run container from Docker Hub image:

```bash
docker run -d -p 8081:80 --name core_docker_api <your-user>/apis:core-docker-api-1.0
docker ps
curl http://localhost:8081/products
```

Congratulations.
