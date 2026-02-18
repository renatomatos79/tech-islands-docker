# Lesson 006 - Add Redis Cache Support to the API

Hello dockers,

In lesson 005 we used Redis directly. Now we will update our .NET Core API to support Redis cache.

Reference solution:  
<https://github.com/renatomatos79/playground/tree/master/vs/CoreDockerApiWithRedis>

## 1. Prepare Project

1. Start from project `vs/CoreDockerApi` (without Redis).
2. Clone repository: <https://github.com/renatomatos79/playground.git>
3. Add NuGet packages:

```xml
<PackageReference Include="Newtonsoft.Json" Version="12.0.3" />
<PackageReference Include="ServiceStack.Redis" Version="5.10.2" />
```

4. Create folder `Cache`.

---

## 2. Add Cache Classes

Create `CacheObject`:

```csharp
public class CacheObject<T>
{
    public CacheObject(T item, int cacheLifeTimeInSeconds)
    {
        this.Item = item;
        this.CreatedOn = DateTime.Now;
        this.LifeTimeInSeconds = cacheLifeTimeInSeconds;
        this.ExpiresOn = this.CreatedOn.AddSeconds(cacheLifeTimeInSeconds);
    }

    public T Item { get; set; }
    public int LifeTimeInSeconds { get; set; }

    public long MaxAgeInSeconds
    {
        get
        {
            if (ExpiresOn > DateTime.Now)
            {
                return (long)ExpiresOn.Subtract(DateTime.Now).TotalSeconds;
            }
            return 0;
        }
    }

    public DateTime CreatedOn { get; set; }
    public DateTime ExpiresOn { get; set; }
}
```

Create `ICacheManager`:

```csharp
public interface ICacheManager
{
    void Add<T>(string key, T value, int expiresInSeconds);
    CacheObject<T> Get<T>(string key);
}
```

Create `CacheManager`:

```csharp
public class CacheManager : ICacheManager
{
    private readonly IRedisClientsManager redisClientsManager;

    public CacheManager(IRedisClientsManager redisClientsManager)
    {
        this.redisClientsManager = redisClientsManager;
    }

    private void AddKey(string key, string value, int expiresInSeconds)
    {
        using (var redis = redisClientsManager.GetClient())
        {
            redis.Set(key, value, DateTime.Now.AddSeconds(expiresInSeconds));
        }
    }

    private string GetKey(string key)
    {
        using (var redis = redisClientsManager.GetClient())
        {
            return redis.Get<string>(key);
        }
    }

    public void Add<T>(string key, T value, int expiresInSeconds)
    {
        var item = new CacheObject<T>(value, expiresInSeconds);
        var json = Newtonsoft.Json.JsonConvert.SerializeObject(item);
        AddKey(key, json, expiresInSeconds);
    }

    public CacheObject<T> Get<T>(string key)
    {
        var value = GetKey(key);
        if (string.IsNullOrEmpty(value))
        {
            return null;
        }

        return Newtonsoft.Json.JsonConvert.DeserializeObject<CacheObject<T>>(value);
    }
}
```

---

## 3. Register Redis in `Startup`

Update `ConfigureServices`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddHttpContextAccessor();
    services.AddSingleton<ICacheManager>(sp =>
    {
        var host = Environment.GetEnvironmentVariable("REDIS_HOST") ?? "redisserver:6379";
        var redisClient = new RedisManagerPool(host);
        return new CacheManager(redisClient);
    });
}
```

---

## 4. Update `ProductController`

Inject dependencies:

```csharp
public ProductController(ICacheManager cacheManager, IHttpContextAccessor httpContextAccessor)
{
    this.cacheManager = cacheManager;
    this.httpContextAccessor = httpContextAccessor;
}
```

Update `GetProductsAsync` to read cache first:

```csharp
[HttpGet("/products")]
public async Task<IActionResult> GetProductsAsync()
{
    var cache = cacheManager.Get<List<ProductModel>>("products");
    if (cache != null && cache.Item != null)
    {
        var headers = this.httpContextAccessor.HttpContext.Response.GetTypedHeaders();
        headers.CacheControl = new Microsoft.Net.Http.Headers.CacheControlHeaderValue
        {
            Public = true,
            MaxAge = TimeSpan.FromSeconds(cache.MaxAgeInSeconds)
        };

        return await Task.FromResult(Ok(cache.Item));
    }

    var result = new List<ProductModel>
    {
        new ProductModel { Id = 1, Name = "Eggs (box with 12 units)", Price = (decimal)1.50 },
        new ProductModel { Id = 2, Name = "Chocolate", Price = (decimal)1.99 },
        new ProductModel { Id = 3, Name = "Butter (President)", Price = (decimal)2.25 },
        new ProductModel { Id = 4, Name = "Codfish (Pascoal)", Price = (decimal)12.25 },
        new ProductModel { Id = 5, Name = "Cheese 500g (Flamingo)", Price = (decimal)2.40 },
        new ProductModel { Id = 6, Name = "Yogurt Danone (6 units)", Price = (decimal)1.45 },
        new ProductModel { Id = 7, Name = "Bread (6 units)", Price = (decimal)0.45 }
    };

    return await Task.FromResult(Ok(result));
}
```

Temporary endpoint to set cache:

```csharp
[HttpPut("/products/{cache}")]
public async Task<IActionResult> PutProductsAsync([FromRoute] string cache, [FromBody] List<ProductModel> products)
{
    this.cacheManager.Add(cache, products, 30);
    return await Task.FromResult(NoContent());
}
```

---

## 5. Build and Run

Build image `1.1`:

```powershell
cd ...\vs\CoreDockerApiWithRedis
docker build -t core-docker-api:1.1 .
```

Stop and remove old container if needed:

```powershell
docker container stop core_docker_api
docker container rm core_docker_api
```

Get Redis container IP and test reachability from Alpine:

```powershell
docker container inspect redisserver | grep IPAddress
docker run -d -it --name myalpine alpine
docker container start -ai myalpine
ping <redis-ip>
```

Run API with Redis host:

```powershell
docker run -d -p 8001:80 --name core_docker_api -e REDIS_HOST=<redis-ip>:6379 core-docker-api:1.1
```

Test:

```powershell
curl http://localhost:8001/products
curl -X PUT http://localhost:8001/products/products -H "content-type: application/json" -d "[{\"Id\":1,\"Name\":\"From cache\",\"Price\":1.5}]"
curl http://localhost:8001/products
```

---

## 6. Push New Image

```bash
docker tag core-docker-api:1.1 <your-user>/apis:core-docker-api-1.1
docker image push <your-user>/apis:core-docker-api-1.1
```
