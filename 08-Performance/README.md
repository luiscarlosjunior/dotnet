# ⚡ Performance e Otimização

## 🎯 Objetivos de Aprendizado

Ao completar este módulo, você será capaz de:
- Escrever código assíncrono eficiente
- Implementar caching estratégias
- Otimizar consultas de banco de dados
- Fazer profiling e diagnosticar problemas
- Gerenciar memória eficientemente
- Aplicar padrões de otimização

## 📚 Conteúdo

### 1. Async/Await

#### Conceitos Fundamentais
```csharp
// ❌ ERRADO - Bloqueante
public string GetData()
{
    var client = new HttpClient();
    var response = client.GetStringAsync("https://api.exemplo.com").Result;
    return response;
}

// ✅ CORRETO - Assíncrono
public async Task<string> GetDataAsync()
{
    using var client = new HttpClient();
    var response = await client.GetStringAsync("https://api.exemplo.com");
    return response;
}
```

#### Async Best Practices
```csharp
// Usar ConfigureAwait(false) em bibliotecas
public async Task<string> ProcessDataAsync()
{
    var data = await GetDataAsync().ConfigureAwait(false);
    return ProcessData(data);
}

// Task.WhenAll para operações paralelas
public async Task<List<string>> GetMultipleDataAsync()
{
    var task1 = GetDataAsync("url1");
    var task2 = GetDataAsync("url2");
    var task3 = GetDataAsync("url3");
    
    var results = await Task.WhenAll(task1, task2, task3);
    return results.ToList();
}

// Task.WhenAny para primeira resposta
public async Task<string> GetFastestDataAsync()
{
    var task1 = GetDataAsync("url1");
    var task2 = GetDataAsync("url2");
    
    var completedTask = await Task.WhenAny(task1, task2);
    return await completedTask;
}

// Cancelamento
public async Task<string> GetDataAsync(CancellationToken cancellationToken)
{
    using var client = new HttpClient();
    var response = await client.GetStringAsync("url", cancellationToken);
    return response;
}
```

### 2. Caching

#### In-Memory Caching
```csharp
// Program.cs
builder.Services.AddMemoryCache();

public class ProdutoService
{
    private readonly IMemoryCache _cache;
    private readonly IProdutoRepository _repository;
    
    public ProdutoService(IMemoryCache cache, IProdutoRepository repository)
    {
        _cache = cache;
        _repository = repository;
    }
    
    public async Task<List<Produto>> ObterTodosAsync()
    {
        const string cacheKey = "produtos_todos";
        
        if (!_cache.TryGetValue(cacheKey, out List<Produto> produtos))
        {
            produtos = await _repository.ObterTodosAsync();
            
            var cacheOptions = new MemoryCacheEntryOptions()
                .SetSlidingExpiration(TimeSpan.FromMinutes(5))
                .SetAbsoluteExpiration(TimeSpan.FromHours(1))
                .SetPriority(CacheItemPriority.Normal);
            
            _cache.Set(cacheKey, produtos, cacheOptions);
        }
        
        return produtos;
    }
}
```

#### Distributed Caching (Redis)
```bash
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis
```

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "localhost:6379";
    options.InstanceName = "MinhaApp_";
});

public class ProdutoService
{
    private readonly IDistributedCache _cache;
    private readonly IProdutoRepository _repository;
    
    public async Task<Produto?> ObterPorIdAsync(int id)
    {
        string cacheKey = $"produto_{id}";
        var produtoCache = await _cache.GetStringAsync(cacheKey);
        
        if (produtoCache != null)
        {
            return JsonSerializer.Deserialize<Produto>(produtoCache);
        }
        
        var produto = await _repository.ObterPorIdAsync(id);
        if (produto != null)
        {
            var options = new DistributedCacheEntryOptions()
                .SetSlidingExpiration(TimeSpan.FromMinutes(5))
                .SetAbsoluteExpiration(TimeSpan.FromHours(1));
            
            await _cache.SetStringAsync(
                cacheKey,
                JsonSerializer.Serialize(produto),
                options);
        }
        
        return produto;
    }
}
```

#### Response Caching
```csharp
// Program.cs
builder.Services.AddResponseCaching();

app.UseResponseCaching();

// Controller
[HttpGet]
[ResponseCache(Duration = 60, Location = ResponseCacheLocation.Any)]
public async Task<ActionResult<List<Produto>>> GetAll()
{
    return await _service.ObterTodosAsync();
}
```

### 3. Database Optimization

#### Consultas Eficientes
```csharp
// ❌ ERRADO - N+1 Problem
var produtos = await _context.Produtos.ToListAsync();
foreach (var produto in produtos)
{
    var categoria = await _context.Categorias
        .FirstAsync(c => c.Id == produto.CategoriaId); // N queries extras
}

// ✅ CORRETO - Eager Loading
var produtos = await _context.Produtos
    .Include(p => p.Categoria)
    .ToListAsync(); // 1 query

// Projeção para reduzir dados
var produtosDto = await _context.Produtos
    .Select(p => new ProdutoDto
    {
        Id = p.Id,
        Nome = p.Nome,
        CategoriaNome = p.Categoria.Nome
    })
    .ToListAsync();

// AsNoTracking para queries read-only
var produtos = await _context.Produtos
    .AsNoTracking()
    .ToListAsync();

// Paginação eficiente
var produtos = await _context.Produtos
    .OrderBy(p => p.Id)
    .Skip((pageNumber - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();
```

#### Índices
```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Índice simples
    modelBuilder.Entity<Produto>()
        .HasIndex(p => p.Nome);
    
    // Índice único
    modelBuilder.Entity<Usuario>()
        .HasIndex(u => u.Email)
        .IsUnique();
    
    // Índice composto
    modelBuilder.Entity<Pedido>()
        .HasIndex(p => new { p.ClienteId, p.Data });
}
```

#### Bulk Operations
```bash
dotnet add package EFCore.BulkExtensions
```

```csharp
// Inserção em massa
await _context.BulkInsertAsync(produtos);

// Atualização em massa
await _context.BulkUpdateAsync(produtos);

// Deleção em massa
await _context.BulkDeleteAsync(produtos);
```

### 4. Memory Management

#### Object Pooling
```csharp
public class ObjectPoolService
{
    private readonly ObjectPool<StringBuilder> _stringBuilderPool;
    
    public ObjectPoolService()
    {
        var policy = new DefaultPooledObjectPolicy<StringBuilder>();
        _stringBuilderPool = new DefaultObjectPool<StringBuilder>(policy);
    }
    
    public string BuildString()
    {
        var sb = _stringBuilderPool.Get();
        try
        {
            sb.Append("Hello");
            sb.Append(" World");
            return sb.ToString();
        }
        finally
        {
            sb.Clear();
            _stringBuilderPool.Return(sb);
        }
    }
}
```

#### ArrayPool
```csharp
public void ProcessLargeData()
{
    var pool = ArrayPool<byte>.Shared;
    byte[] buffer = pool.Rent(1024);
    
    try
    {
        // Usar buffer
        ProcessData(buffer);
    }
    finally
    {
        pool.Return(buffer);
    }
}
```

#### Span<T> e Memory<T>
```csharp
// Evita alocações desnecessárias
public int SumNumbers(ReadOnlySpan<int> numbers)
{
    int sum = 0;
    foreach (var num in numbers)
    {
        sum += num;
    }
    return sum;
}

// Uso
int[] array = { 1, 2, 3, 4, 5 };
var sum = SumNumbers(array.AsSpan(1, 3)); // Soma apenas índices 1-3
```

### 5. HTTP Performance

#### HTTP Client Factory
```csharp
// Program.cs
builder.Services.AddHttpClient("MyAPI", client =>
{
    client.BaseAddress = new Uri("https://api.exemplo.com");
    client.DefaultRequestHeaders.Add("Accept", "application/json");
    client.Timeout = TimeSpan.FromSeconds(30);
});

// Uso
public class MyService
{
    private readonly IHttpClientFactory _httpClientFactory;
    
    public MyService(IHttpClientFactory httpClientFactory)
    {
        _httpClientFactory = httpClientFactory;
    }
    
    public async Task<string> GetDataAsync()
    {
        var client = _httpClientFactory.CreateClient("MyAPI");
        return await client.GetStringAsync("/endpoint");
    }
}
```

#### Compression
```csharp
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
});

app.UseResponseCompression();
```

### 6. LINQ Optimization

```csharp
// ❌ ERRADO - Múltiplas iterações
var list = GetLargeList();
var count = list.Count();
var first = list.First();
var last = list.Last();

// ✅ CORRETO - Uma iteração
var list = GetLargeList().ToList(); // Materializar uma vez
var count = list.Count;
var first = list[0];
var last = list[list.Count - 1];

// Usar Any() ao invés de Count() > 0
if (list.Any()) // ✅ Para quando encontra primeiro
if (list.Count() > 0) // ❌ Conta todos

// FirstOrDefault ao invés de Where().FirstOrDefault()
var item = list.FirstOrDefault(x => x.Id == id); // ✅
var item = list.Where(x => x.Id == id).FirstOrDefault(); // ❌
```

### 7. Profiling e Diagnóstico

#### BenchmarkDotNet
```bash
dotnet add package BenchmarkDotNet
```

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

[MemoryDiagnoser]
public class StringConcatBenchmark
{
    [Benchmark]
    public string UsingPlus()
    {
        string result = "";
        for (int i = 0; i < 100; i++)
            result += i.ToString();
        return result;
    }
    
    [Benchmark]
    public string UsingStringBuilder()
    {
        var sb = new StringBuilder();
        for (int i = 0; i < 100; i++)
            sb.Append(i);
        return sb.ToString();
    }
}

class Program
{
    static void Main(string[] args)
    {
        BenchmarkRunner.Run<StringConcatBenchmark>();
    }
}
```

#### Application Insights
```bash
dotnet add package Microsoft.ApplicationInsights.AspNetCore
```

```csharp
builder.Services.AddApplicationInsightsTelemetry();
```

#### MiniProfiler
```bash
dotnet add package MiniProfiler.AspNetCore.Mvc
```

```csharp
builder.Services.AddMiniProfiler(options =>
{
    options.RouteBasePath = "/profiler";
}).AddEntityFramework();

app.UseMiniProfiler();
```

### 8. Performance Best Practices

```csharp
// 1. Use StringBuilder para concatenação em loops
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
    sb.Append(i);

// 2. Evite boxing/unboxing
int value = 42;
object obj = value; // Boxing ❌
int back = (int)obj; // Unboxing ❌

// 3. Use structs para tipos pequenos e imutáveis
public readonly struct Point
{
    public int X { get; }
    public int Y { get; }
}

// 4. Reutilize objetos quando possível
private static readonly HttpClient _httpClient = new HttpClient();

// 5. Use lazy loading quando apropriado
private readonly Lazy<ExpensiveObject> _expensive = 
    new Lazy<ExpensiveObject>(() => new ExpensiveObject());

// 6. Evite closures desnecessários
// ❌
var result = list.Where(x => x.Price > minPrice).ToList();

// ✅ (se minPrice for constante)
Func<Item, bool> filter = x => x.Price > 100;
var result = list.Where(filter).ToList();
```

## 💡 Dicas de Estudo

1. **Meça Antes de Otimizar**: Profile primeiro, otimize depois
2. **Async All the Way**: Não misture sync e async
3. **Cache Inteligentemente**: Nem sempre mais cache é melhor
4. **Use Ferramentas**: BenchmarkDotNet, dotMemory, dotTrace
5. **Leia o Código Gerado**: Entenda o que o compilador faz

## 📝 Exercícios Práticos

### Básico
1. Converta código síncrono para assíncrono
2. Implemente caching em memória
3. Use AsNoTracking em queries read-only
4. Configure Response Caching

### Intermediário
5. Implemente distributed caching com Redis
6. Otimize consultas EF Core (evite N+1)
7. Use HttpClientFactory
8. Implemente compression
9. Use BenchmarkDotNet para comparar algoritmos

### Avançado
10. Implemente object pooling
11. Use Span<T> para operações em memória
12. Configure Application Insights
13. Otimize queries com índices apropriados
14. Implemente bulk operations

### Projeto
**Otimização de API Existente**: Pegue uma API e:
- Profile para identificar gargalos
- Adicione caching apropriado
- Otimize queries de banco de dados
- Implemente async/await corretamente
- Configure compression
- Adicione índices necessários
- Documente melhorias de performance (antes/depois)

## 🔗 Recursos Recomendados

### Documentação
- [Performance Best Practices](https://learn.microsoft.com/aspnet/core/performance/performance-best-practices)
- [EF Core Performance](https://learn.microsoft.com/ef/core/performance/)
- [Async Programming](https://learn.microsoft.com/dotnet/csharp/async)

### Vídeos
- [High Performance ASP.NET Core](https://www.youtube.com/watch?v=9PixRRnN7Lk)
- [.NET Performance Tips](https://www.youtube.com/watch?v=qCN0ZL9sS7Q)

### Artigos
- [Memory Management](https://learn.microsoft.com/dotnet/standard/garbage-collection/)
- [Caching in ASP.NET Core](https://learn.microsoft.com/aspnet/core/performance/caching/memory)

### Livros
- "Pro .NET Performance" - Sasha Goldshtein
- "Writing High-Performance .NET Code" - Ben Watson

## ✅ Checklist de Conclusão

Antes de avançar, certifique-se de que você:

- [ ] Domina async/await
- [ ] Sabe implementar caching (memory e distributed)
- [ ] Conhece otimizações de EF Core
- [ ] Entende memory management (.NET)
- [ ] Sabe usar HttpClientFactory
- [ ] Conhece ferramentas de profiling
- [ ] Implementou compression
- [ ] Sabe quando usar Span<T> e Memory<T>
- [ ] Consegue medir e comparar performance
- [ ] Completou projeto de otimização

## ⏭️ Próximo Passo

Após dominar performance, avance para [09-DevOps](../09-DevOps/README.md) para deployment e CI/CD.

---

*"Premature optimization is the root of all evil." - Donald Knuth*
