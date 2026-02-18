# Lesson 003 - Build and Run a .NET Core API in Docker

Hello dockers,

Until now we built local images only. In this lesson we will:

- set up Docker locally
- build and run a .NET Core API in a container

## 1. Prepare Local Environment (Windows)

1. Enable **Hyper-V** in Control Panel.
2. Check **Hyper-V Management Tools** and **Hyper-V Platform**.
3. Install Docker Desktop for Windows:  
   <https://hub.docker.com/editions/community/docker-ce-desktop-windows/>
4. Sign in to Docker Desktop.
5. Validate installation:

```powershell
docker run hello-world
```

---

## 2. Create the API Project

Reference project:  
<https://github.com/renatomatos79/playground/tree/master/vs/CoreDockerApi>

1. Open Visual Studio 2019.
2. Create a **.NET Core Web Application** named `CoreDockerApi`.
3. Location: `C:\Temp\docker-k8s\vs`.
4. Enable **Keep the solution in the same project folder**.
5. Use **Empty project**, **.NET Core**, **ASP.NET Core 3.1**.
6. Uncheck **Configure for HTTPS** and **Enable Docker support** (we do this manually).

Create `ProductModel`:

```csharp
public class ProductModel
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

Create `ProductController`:

```csharp
public class ProductController : Controller
{
    [HttpGet("/products")]
    public async Task<IEnumerable<ProductModel>> GetProductsAsync()
    {
        return await Task.FromResult(new List<ProductModel>
        {
            new ProductModel { Id = 1, Name = "Eggs (box with 12 units)", Price = (decimal)1.50 },
            new ProductModel { Id = 2, Name = "Chocolate", Price = (decimal)1.99 }
        });
    }
}
```

Update `Startup.ConfigureServices`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
}
```

Update `Startup.Configure`:

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseHttpsRedirection();
    app.UseRouting();
    app.UseAuthorization();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

Test locally:

```powershell
curl http://localhost:56879/products
```

---

## 3. Build Docker Image

Use this Dockerfile reference:  
<https://raw.githubusercontent.com/renatomatos79/playground/master/vs/CoreDockerApi/Dockerfile>

Build the image:

```powershell
docker build -t core-docker-api:1.0 .
```

Run container exposing port `80` as `8001`:

```powershell
docker run -d -p 8001:80 --name core_docker_api core-docker-api:1.0
```

Validate result:

```powershell
docker ps
curl http://localhost:8001/products
```
