# 🌐 APIs - Desenvolvimento de APIs Modernas

## 🎯 Objetivos de Aprendizado

Ao completar este módulo, você será capaz de:
- Criar APIs RESTful profissionais
- Implementar versionamento de APIs
- Documentar APIs com Swagger/OpenAPI
- Trabalhar com diferentes formatos (REST, GraphQL, gRPC)
- Implementar paginação, filtros e ordenação
- Aplicar rate limiting e throttling

## 📚 Conteúdo

### 1. REST API Fundamentals

#### Princípios REST
- **Stateless**: Cada requisição é independente
- **Client-Server**: Separação de responsabilidades
- **Cacheable**: Respostas podem ser cacheadas
- **Uniform Interface**: Interface consistente
- **Layered System**: Arquitetura em camadas

#### Verbos HTTP
```
GET     /api/produtos           # Listar todos
GET     /api/produtos/1         # Obter por ID
POST    /api/produtos           # Criar novo
PUT     /api/produtos/1         # Atualizar completo
PATCH   /api/produtos/1         # Atualizar parcial
DELETE  /api/produtos/1         # Deletar
```

#### Status Codes
```
200 OK                  # Sucesso
201 Created             # Recurso criado
204 No Content          # Sucesso sem conteúdo
400 Bad Request         # Erro no request
401 Unauthorized        # Não autenticado
403 Forbidden           # Não autorizado
404 Not Found           # Não encontrado
500 Internal Server     # Erro do servidor
```

### 2. Criando uma API RESTful

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class ProdutosController : ControllerBase
{
    private readonly IProdutoService _service;
    private readonly ILogger<ProdutosController> _logger;
    
    public ProdutosController(
        IProdutoService service,
        ILogger<ProdutosController> logger)
    {
        _service = service;
        _logger = logger;
    }
    
    /// <summary>
    /// Lista todos os produtos
    /// </summary>
    [HttpGet]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public async Task<ActionResult<IEnumerable<ProdutoDto>>> GetAll(
        [FromQuery] ProdutoFiltros filtros)
    {
        var produtos = await _service.ObterTodosAsync(filtros);
        return Ok(produtos);
    }
    
    /// <summary>
    /// Obtém um produto por ID
    /// </summary>
    [HttpGet("{id}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProdutoDto>> GetById(int id)
    {
        var produto = await _service.ObterPorIdAsync(id);
        
        if (produto == null)
            return NotFound();
        
        return Ok(produto);
    }
    
    /// <summary>
    /// Cria um novo produto
    /// </summary>
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ProdutoDto>> Create(
        [FromBody] CriarProdutoDto dto)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
        
        var produto = await _service.CriarAsync(dto);
        
        return CreatedAtAction(
            nameof(GetById),
            new { id = produto.Id },
            produto);
    }
    
    /// <summary>
    /// Atualiza um produto existente
    /// </summary>
    [HttpPut("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> Update(
        int id,
        [FromBody] AtualizarProdutoDto dto)
    {
        if (id != dto.Id)
            return BadRequest();
        
        var sucesso = await _service.AtualizarAsync(dto);
        
        if (!sucesso)
            return NotFound();
        
        return NoContent();
    }
    
    /// <summary>
    /// Deleta um produto
    /// </summary>
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> Delete(int id)
    {
        var sucesso = await _service.DeletarAsync(id);
        
        if (!sucesso)
            return NotFound();
        
        return NoContent();
    }
}
```

### 3. DTOs e Validação

```csharp
public class CriarProdutoDto
{
    [Required(ErrorMessage = "Nome é obrigatório")]
    [StringLength(100, ErrorMessage = "Nome deve ter no máximo 100 caracteres")]
    public string Nome { get; set; }
    
    [Range(0.01, 999999.99, ErrorMessage = "Preço deve estar entre 0.01 e 999999.99")]
    public decimal Preco { get; set; }
    
    [StringLength(500)]
    public string? Descricao { get; set; }
    
    [Required]
    public int CategoriaId { get; set; }
}

// FluentValidation (alternativa mais poderosa)
public class CriarProdutoDtoValidator : AbstractValidator<CriarProdutoDto>
{
    public CriarProdutoDtoValidator()
    {
        RuleFor(x => x.Nome)
            .NotEmpty()
            .MaximumLength(100);
        
        RuleFor(x => x.Preco)
            .GreaterThan(0)
            .LessThanOrEqualTo(999999.99m);
        
        RuleFor(x => x.CategoriaId)
            .GreaterThan(0)
            .WithMessage("Categoria inválida");
    }
}
```

### 4. Paginação, Filtros e Ordenação

```csharp
public class ProdutoFiltros
{
    public int PageNumber { get; set; } = 1;
    public int PageSize { get; set; } = 10;
    public string? Nome { get; set; }
    public decimal? PrecoMinimo { get; set; }
    public decimal? PrecoMaximo { get; set; }
    public string? OrdenarPor { get; set; } = "nome";
    public string? Ordem { get; set; } = "asc";
}

public class PaginatedResult<T>
{
    public List<T> Items { get; set; }
    public int PageNumber { get; set; }
    public int PageSize { get; set; }
    public int TotalPages { get; set; }
    public int TotalCount { get; set; }
    public bool HasPrevious => PageNumber > 1;
    public bool HasNext => PageNumber < TotalPages;
}

public async Task<PaginatedResult<ProdutoDto>> ObterTodosAsync(
    ProdutoFiltros filtros)
{
    var query = _context.Produtos.AsQueryable();
    
    // Filtros
    if (!string.IsNullOrEmpty(filtros.Nome))
        query = query.Where(p => p.Nome.Contains(filtros.Nome));
    
    if (filtros.PrecoMinimo.HasValue)
        query = query.Where(p => p.Preco >= filtros.PrecoMinimo);
    
    if (filtros.PrecoMaximo.HasValue)
        query = query.Where(p => p.Preco <= filtros.PrecoMaximo);
    
    // Ordenação
    query = filtros.OrdenarPor?.ToLower() switch
    {
        "preco" => filtros.Ordem == "desc" 
            ? query.OrderByDescending(p => p.Preco)
            : query.OrderBy(p => p.Preco),
        _ => filtros.Ordem == "desc"
            ? query.OrderByDescending(p => p.Nome)
            : query.OrderBy(p => p.Nome)
    };
    
    // Contar total
    var totalCount = await query.CountAsync();
    
    // Paginação
    var items = await query
        .Skip((filtros.PageNumber - 1) * filtros.PageSize)
        .Take(filtros.PageSize)
        .Select(p => new ProdutoDto { /* mapeamento */ })
        .ToListAsync();
    
    return new PaginatedResult<ProdutoDto>
    {
        Items = items,
        PageNumber = filtros.PageNumber,
        PageSize = filtros.PageSize,
        TotalCount = totalCount,
        TotalPages = (int)Math.Ceiling(totalCount / (double)filtros.PageSize)
    };
}
```

### 5. Swagger/OpenAPI

```bash
dotnet add package Swashbuckle.AspNetCore
```

```csharp
// Program.cs
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Minha API",
        Version = "v1",
        Description = "API para gerenciamento de produtos",
        Contact = new OpenApiContact
        {
            Name = "Seu Nome",
            Email = "email@exemplo.com"
        }
    });
    
    // Incluir comentários XML
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "Minha API v1");
    });
}
```

### 6. Versionamento de API

```csharp
// Instalar pacote
// dotnet add package Asp.Versioning.Mvc

builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
});

// URL Versioning
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
public class ProdutosV1Controller : ControllerBase
{
    // Versão 1.0
}

[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("2.0")]
public class ProdutosV2Controller : ControllerBase
{
    // Versão 2.0
}

// Header Versioning
[ApiController]
[Route("api/[controller]")]
[ApiVersion("1.0")]
public class ProdutosController : ControllerBase
{
    // Usar header: api-version: 1.0
}
```

### 7. CORS

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", builder =>
    {
        builder.AllowAnyOrigin()
               .AllowAnyMethod()
               .AllowAnyHeader();
    });
    
    options.AddPolicy("Production", builder =>
    {
        builder.WithOrigins("https://meusite.com")
               .AllowAnyMethod()
               .AllowAnyHeader()
               .AllowCredentials();
    });
});

app.UseCors("AllowAll");
```

### 8. Rate Limiting

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(
        httpContext => RateLimitPartition.GetFixedWindowLimiter(
            partitionKey: httpContext.User.Identity?.Name ?? "anonymous",
            factory: partition => new FixedWindowRateLimiterOptions
            {
                PermitLimit = 100,
                Window = TimeSpan.FromMinutes(1)
            }));
});

app.UseRateLimiter();

// Em controller específico
[EnableRateLimiting("fixed")]
public class ProdutosController : ControllerBase
{
}
```

### 9. GraphQL com HotChocolate

```bash
dotnet add package HotChocolate.AspNetCore
```

```csharp
public class Query
{
    public async Task<IEnumerable<Produto>> GetProdutos(
        [Service] IProdutoService service)
    {
        return await service.ObterTodosAsync();
    }
    
    public async Task<Produto?> GetProduto(
        int id,
        [Service] IProdutoService service)
    {
        return await service.ObterPorIdAsync(id);
    }
}

// Program.cs
builder.Services
    .AddGraphQLServer()
    .AddQueryType<Query>();

app.MapGraphQL();
```

### 10. gRPC

```protobuf
// Protos/produtos.proto
syntax = "proto3";

service ProdutoService {
  rpc ObterProduto (ProdutoRequest) returns (ProdutoResponse);
  rpc ListarProdutos (Empty) returns (ProdutosResponse);
}

message ProdutoRequest {
  int32 id = 1;
}

message ProdutoResponse {
  int32 id = 1;
  string nome = 2;
  double preco = 3;
}
```

## 💡 Dicas de Estudo

1. **Use DTOs**: Separe modelos de domínio de API
2. **Documente com Swagger**: Facilita consumo da API
3. **Versione Desde o Início**: Evita breaking changes
4. **Valide Entrada**: Sempre valide dados recebidos
5. **Use Status Codes Corretos**: Comunica claramente o resultado

## 📝 Exercícios Práticos

### Básico
1. Crie uma API RESTful completa para um recurso
2. Implemente validação com Data Annotations
3. Configure Swagger e documente endpoints
4. Teste API com Postman ou Insomnia

### Intermediário
5. Implemente paginação e filtros
6. Configure CORS para diferentes origens
7. Adicione versionamento à sua API
8. Implemente rate limiting
9. Use FluentValidation

### Avançado
10. Crie API com GraphQL
11. Implemente gRPC service
12. Configure API Gateway
13. Adicione suporte a ETags e caching
14. Implemente HATEOAS

### Projeto
**API de E-commerce Completa**: Desenvolva uma API que:
- Gerencia produtos, categorias, clientes e pedidos
- Implementa paginação, filtros e ordenação
- Tem documentação Swagger completa
- Suporta versionamento
- Inclui rate limiting
- Tem testes de integração
- Implementa autenticação JWT (veja módulo 07)

## 🔗 Recursos Recomendados

### Documentação
- [Web API Tutorial](https://learn.microsoft.com/aspnet/core/tutorials/first-web-api)
- [REST API Guidelines](https://github.com/microsoft/api-guidelines)
- [Swagger/OpenAPI](https://swagger.io/)

### Vídeos
- [RESTful API Design](https://www.youtube.com/watch?v=qVTAB8Z2VmA)
- [Building APIs with ASP.NET Core](https://www.youtube.com/watch?v=fmvcAzHpsk8)

### Artigos
- [Best Practices for REST API Design](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)
- [API Versioning](https://learn.microsoft.com/aspnet/core/fundamentals/api-versioning)

## ✅ Checklist de Conclusão

Antes de avançar, certifique-se de que você:

- [ ] Entende princípios REST
- [ ] Sabe criar endpoints RESTful completos
- [ ] Domina DTOs e validação
- [ ] Conhece paginação e filtros
- [ ] Configurou Swagger/OpenAPI
- [ ] Implementou versionamento
- [ ] Conhece CORS e suas configurações
- [ ] Sabe implementar rate limiting
- [ ] Tem noção de GraphQL e gRPC
- [ ] Completou o projeto de API de E-commerce

## ⏭️ Próximo Passo

Após dominar APIs, avance para [07-Seguranca](../07-Seguranca/README.md) para aprender sobre segurança.

---

*"Good API design is hard." - Designing APIs that developers love to use*
