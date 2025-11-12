# 🌐 ASP.NET Core - Desenvolvimento Web

## 🎯 Objetivos de Aprendizado

Ao completar este módulo, você será capaz de:
- Criar aplicações web com ASP.NET Core
- Entender o padrão MVC (Model-View-Controller)
- Trabalhar com Razor Pages e Views
- Criar APIs RESTful
- Implementar middleware e dependency injection
- Gerenciar rotas e configurações

## 📚 Conteúdo

### 1. Introdução ao ASP.NET Core

#### O que é ASP.NET Core?
- Framework web moderno, cross-platform e open-source
- Alto desempenho e escalável
- Suporte a Docker e cloud-native
- Integração com front-end moderno (React, Angular, Vue)

#### Tipos de Aplicação
- **MVC Applications**: Apps web tradicionais
- **Razor Pages**: Páginas focadas em simplicidade
- **Web APIs**: Serviços REST/GraphQL
- **Blazor**: SPAs com C# (WebAssembly ou Server)

### 2. Criando Seu Primeiro Projeto

```bash
# MVC Application
dotnet new mvc -n MeuAppMVC

# Web API
dotnet new webapi -n MinhaAPI

# Razor Pages
dotnet new webapp -n MeuSiteRazor
```

#### Estrutura de um Projeto MVC
```
MeuAppMVC/
├── Controllers/         # Lógica de controle
├── Models/             # Modelos de dados
├── Views/              # Templates Razor
├── wwwroot/            # Arquivos estáticos (CSS, JS, imagens)
├── appsettings.json    # Configurações
└── Program.cs          # Ponto de entrada
```

### 3. MVC Pattern

#### Model
```csharp
public class Produto
{
    public int Id { get; set; }
    public string Nome { get; set; }
    public decimal Preco { get; set; }
    public string Descricao { get; set; }
}
```

#### Controller
```csharp
public class ProdutosController : Controller
{
    private readonly IProdutoService _produtoService;
    
    public ProdutosController(IProdutoService produtoService)
    {
        _produtoService = produtoService;
    }
    
    public IActionResult Index()
    {
        var produtos = _produtoService.ObterTodos();
        return View(produtos);
    }
    
    public IActionResult Detalhes(int id)
    {
        var produto = _produtoService.ObterPorId(id);
        if (produto == null)
            return NotFound();
        
        return View(produto);
    }
    
    [HttpPost]
    public IActionResult Criar(Produto produto)
    {
        if (ModelState.IsValid)
        {
            _produtoService.Adicionar(produto);
            return RedirectToAction(nameof(Index));
        }
        return View(produto);
    }
}
```

#### View (Razor)
```razor
@model IEnumerable<Produto>

<h1>Produtos</h1>

<table class="table">
    <thead>
        <tr>
            <th>Nome</th>
            <th>Preço</th>
            <th>Ações</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var produto in Model)
        {
            <tr>
                <td>@produto.Nome</td>
                <td>@produto.Preco.ToString("C")</td>
                <td>
                    <a asp-action="Detalhes" asp-route-id="@produto.Id">Ver</a>
                </td>
            </tr>
        }
    </tbody>
</table>
```

### 4. Razor Pages

```csharp
// Pages/Produtos/Index.cshtml.cs
public class IndexModel : PageModel
{
    private readonly IProdutoService _produtoService;
    
    public List<Produto> Produtos { get; set; }
    
    public IndexModel(IProdutoService produtoService)
    {
        _produtoService = produtoService;
    }
    
    public void OnGet()
    {
        Produtos = _produtoService.ObterTodos().ToList();
    }
}
```

```razor
@* Pages/Produtos/Index.cshtml *@
@page
@model IndexModel

<h1>Produtos</h1>

@foreach (var produto in Model.Produtos)
{
    <div>
        <h3>@produto.Nome</h3>
        <p>@produto.Preco.ToString("C")</p>
    </div>
}
```

### 5. Web API

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProdutosController : ControllerBase
{
    private readonly IProdutoService _produtoService;
    
    public ProdutosController(IProdutoService produtoService)
    {
        _produtoService = produtoService;
    }
    
    [HttpGet]
    public ActionResult<IEnumerable<Produto>> GetTodos()
    {
        return Ok(_produtoService.ObterTodos());
    }
    
    [HttpGet("{id}")]
    public ActionResult<Produto> GetPorId(int id)
    {
        var produto = _produtoService.ObterPorId(id);
        if (produto == null)
            return NotFound();
        
        return Ok(produto);
    }
    
    [HttpPost]
    public ActionResult<Produto> Criar(Produto produto)
    {
        _produtoService.Adicionar(produto);
        return CreatedAtAction(nameof(GetPorId), 
            new { id = produto.Id }, produto);
    }
    
    [HttpPut("{id}")]
    public IActionResult Atualizar(int id, Produto produto)
    {
        if (id != produto.Id)
            return BadRequest();
        
        _produtoService.Atualizar(produto);
        return NoContent();
    }
    
    [HttpDelete("{id}")]
    public IActionResult Deletar(int id)
    {
        _produtoService.Deletar(id);
        return NoContent();
    }
}
```

### 6. Dependency Injection

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Registrar serviços
builder.Services.AddScoped<IProdutoService, ProdutoService>();
builder.Services.AddScoped<IProdutoRepository, ProdutoRepository>();

// DbContext
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

var app = builder.Build();
```

### 7. Middleware

```csharp
var app = builder.Build();

// Pipeline de middleware
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.MapRazorPages();

app.Run();
```

#### Middleware Customizado
```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;
    
    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        Console.WriteLine($"Request: {context.Request.Path}");
        await _next(context);
        Console.WriteLine($"Response: {context.Response.StatusCode}");
    }
}

// Uso
app.UseMiddleware<LoggingMiddleware>();
```

### 8. Configuração

```json
// appsettings.json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=MinhaDb;..."
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AppSettings": {
    "ApiKey": "minha-chave-secreta"
  }
}
```

```csharp
// Acessar configuração
public class MyService
{
    private readonly IConfiguration _config;
    
    public MyService(IConfiguration config)
    {
        _config = config;
    }
    
    public void Usar()
    {
        var apiKey = _config["AppSettings:ApiKey"];
        var connString = _config.GetConnectionString("Default");
    }
}
```

## 💡 Dicas de Estudo

1. **Comece com Razor Pages**: Mais simples para começar
2. **Entenda o Pipeline de Middleware**: Ordem importa!
3. **Use DI**: Facilita testes e manutenção
4. **Aprenda Routing**: Fundamental para APIs
5. **Pratique APIs RESTful**: Muito usado no mercado

## 📝 Exercícios Práticos

### Básico
1. Crie um projeto MVC simples com CRUD de produtos
2. Faça uma Razor Page que exibe uma lista de tarefas
3. Crie uma Web API simples com endpoints GET/POST
4. Configure diferentes ambientes (Development, Production)

### Intermediário
5. Implemente validação de dados com Data Annotations
6. Crie middleware customizado para logging
7. Configure CORS para sua API
8. Implemente upload de arquivos
9. Use Tag Helpers em views Razor

### Avançado
10. Crie uma API completa com autenticação JWT
11. Implemente paginação e filtros em endpoints
12. Use filters (ActionFilters, ExceptionFilters)
13. Configure response caching

### Projeto
**Sistema de Blog**: Crie uma aplicação web que:
- Lista posts (página inicial)
- Permite criar, editar e deletar posts (área admin)
- Mostra detalhes de um post com comentários
- Usa Entity Framework Core
- Implementa autenticação
- Tem uma API para consumo externo

## 🔗 Recursos Recomendados

### Documentação
- [ASP.NET Core Documentation](https://learn.microsoft.com/aspnet/core/)
- [Razor Syntax Reference](https://learn.microsoft.com/aspnet/core/mvc/views/razor)
- [Web API Tutorial](https://learn.microsoft.com/aspnet/core/tutorials/first-web-api)

### Tutoriais
- [Get Started with ASP.NET Core](https://learn.microsoft.com/aspnet/core/getting-started/)
- [Build Web Apps with ASP.NET Core (Microsoft Learn)](https://learn.microsoft.com/training/paths/aspnet-core-web-app/)

### Vídeos
- [ASP.NET Core 101](https://www.youtube.com/playlist?list=PLdo4fOcmZ0oW8nviYduHq7bmKode-p8Wy)
- [Tim Corey - ASP.NET Core](https://www.youtube.com/c/IAmTimCorey)

### Livros
- "ASP.NET Core in Action" - Andrew Lock
- "Pro ASP.NET Core 6" - Adam Freeman

## ✅ Checklist de Conclusão

Antes de avançar, certifique-se de que você:

- [ ] Entende o padrão MVC
- [ ] Sabe criar Controllers e Actions
- [ ] Conhece Razor syntax
- [ ] Consegue criar uma Web API RESTful
- [ ] Entende Dependency Injection
- [ ] Sabe o que é middleware e sua ordem
- [ ] Conhece routing e attribute routing
- [ ] Sabe trabalhar com configurações
- [ ] Implementou validação de modelos
- [ ] Completou o projeto do Sistema de Blog

## ⏭️ Próximo Passo

Após dominar ASP.NET Core, avance para [04-EntityFramework](../04-EntityFramework/README.md) para aprender acesso a dados.

---

*"Make it work, make it right, make it fast." - Kent Beck*
