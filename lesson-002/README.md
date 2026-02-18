# Lesson 002 - Hosting HTML with Nginx and Docker

Hello dockers,

## 1. Open Play with Docker

Open a new browser and access:

<https://labs.play-with-docker.com/>

1. Click **Login**.
2. After login, click **Start**.
3. On the **Instances** page, click **+ Add new instance** to start a new playground.

---

## 2. Serve an HTML file with Nginx (volume mapping)

At this point, we will no longer be only Docker CLI operators.
We are going to run HTML files inside the Nginx application server.

Create your first HTML file using the template available in the `templates` folder:

```bash
mkdir html
cd html
curl https://raw.githubusercontent.com/renatomatos79/playground/master/templates/blankpage.html >> index.html
```

Inspect the HTML file:

```bash
cat index.html
```

Expected content:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-giJF6kkoqNQ00vy+HMDP7azOuL0xtbfIcaT9wjKHr8RbDVddVHyTfAAsrekwKmP1" crossorigin="anonymous">
    <title>Hello, world!</title>
  </head>
  <body>
    <h1>Hello, world!</h1>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta1/dist/js/bootstrap.bundle.min.js" integrity="sha384-ygbV9kiqUc6oa4msXn9868pTtWMgiQaeYH7/t7LECLbyPA2x65Kgf80OJFdroafW" crossorigin="anonymous"></script>
  </body>
</html>
```

### What are we going to do?

Map the host folder `~/html` into the container folder `/usr/share/nginx/html`.

What does it mean?

- If we create a file in the host folder, it appears in the container.
- If we delete a file in the host folder, it is also deleted in the container.
- If we create a file in the container folder, it appears in the host.
- If we delete a file in the container folder, it is also deleted in the host.

Run Nginx with the mapped volume:

```bash
docker container run -d --name nginxsrv -p 8080:80 -v ~/html:/usr/share/nginx/html nginx
```

Check whether the container is running:

```bash
docker ps
```

Check container logs:

```bash
docker container logs nginxsrv
```

Test the page:

```bash
curl http://localhost:8080/index.html
```

---

## 3. Alternative approach: use a Dockerfile

Now let us host the same page in Nginx using a custom image.

### Dockerfile explanation

1. `FROM`: base image and version.
2. `LABEL`: image metadata (maintainer).
3. `WORKDIR`: default working folder inside container.
4. `COPY`: copies `index.html` from host to container.

> Attention: for this example, the `Dockerfile` must be in the same folder as `index.html`.

Example Dockerfile:

```dockerfile
FROM nginx:latest
LABEL maintainer='Renato Matos'
WORKDIR /usr/share/nginx/html
COPY ./index.html .
```

Create files:

```bash
mkdir html
cd html
curl https://raw.githubusercontent.com/renatomatos79/playground/master/templates/DockerfileBlankPage.yaml >> Dockerfile
cat Dockerfile
```

Build and tag the image:

```bash
docker image build -t blankpage:1.0.1 .
```

Run the new container:

```bash
docker container run -d --name nginxsrv2 -p 8081:80 blankpage:1.0.1
```

Test it:

```bash
curl http://localhost:8081
```
