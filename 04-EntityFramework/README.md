# 🗄️ Entity Framework Core - ORM e Acesso a Dados

## 🎯 Objetivos de Aprendizado

Ao completar este módulo, você será capaz de:
- Trabalhar com Entity Framework Core como ORM
- Criar e gerenciar modelos de dados
- Realizar operações CRUD no banco de dados
- Utilizar LINQ para consultas
- Gerenciar migrations e schemas
- Implementar relacionamentos entre entidades

## 📚 Conteúdo

### 1. Introdução ao EF Core

#### O que é Entity Framework Core?
- ORM (Object-Relational Mapper) moderno e leve
- Suporta múltiplos bancos de dados
- Code-First e Database-First
- LINQ para consultas type-safe

#### Bancos Suportados
- SQL Server
- PostgreSQL
- MySQL/MariaDB
- SQLite
- Oracle
- In-Memory (para testes)

### 2. Configuração Inicial

```bash
# Instalar pacotes
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.Design
```

#### DbContext
```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }
    
    public DbSet<Produto> Produtos { get; set; }
    public DbSet<Categoria> Categorias { get; set; }
    public DbSet<Pedido> Pedidos { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configurações adicionais
        modelBuilder.Entity<Produto>()
            .Property(p => p.Preco)
            .HasPrecision(18, 2);
            
        modelBuilder.Entity<Categoria>()
            .HasIndex(c => c.Nome)
            .IsUnique();
    }
}
```

#### Configuração no Program.cs
```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("Default")));
```

### 3. Modelos e Entidades

```csharp
public class Produto
{
    public int Id { get; set; }
    
    [Required]
    [MaxLength(100)]
    public string Nome { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal Preco { get; set; }
    
    public string? Descricao { get; set; }
    
    // Navigation Properties
    public int CategoriaId { get; set; }
    public Categoria Categoria { get; set; }
    
    public ICollection<ItemPedido> ItensPedido { get; set; }
}

public class Categoria
{
    public int Id { get; set; }
    public string Nome { get; set; }
    
    public ICollection<Produto> Produtos { get; set; }
}
```

### 4. Migrations

```bash
# Criar migration
dotnet ef migrations add InitialCreate

# Aplicar migrations
dotnet ef database update

# Remover última migration
dotnet ef migrations remove

# Gerar script SQL
dotnet ef migrations script
```

#### Migration Customizada
```csharp
public partial class AddProdutoIndex : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateIndex(
            name: "IX_Produtos_Nome",
            table: "Produtos",
            column: "Nome");
    }
    
    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropIndex(
            name: "IX_Produtos_Nome",
            table: "Produtos");
    }
}
```

### 5. Operações CRUD

```csharp
public class ProdutoRepository
{
    private readonly AppDbContext _context;
    
    public ProdutoRepository(AppDbContext context)
    {
        _context = context;
    }
    
    // Create
    public async Task<Produto> AdicionarAsync(Produto produto)
    {
        _context.Produtos.Add(produto);
        await _context.SaveChangesAsync();
        return produto;
    }
    
    // Read
    public async Task<Produto?> ObterPorIdAsync(int id)
    {
        return await _context.Produtos
            .Include(p => p.Categoria)
            .FirstOrDefaultAsync(p => p.Id == id);
    }
    
    public async Task<List<Produto>> ObterTodosAsync()
    {
        return await _context.Produtos
            .Include(p => p.Categoria)
            .ToListAsync();
    }
    
    // Update
    public async Task AtualizarAsync(Produto produto)
    {
        _context.Entry(produto).State = EntityState.Modified;
        await _context.SaveChangesAsync();
    }
    
    // Delete
    public async Task DeletarAsync(int id)
    {
        var produto = await _context.Produtos.FindAsync(id);
        if (produto != null)
        {
            _context.Produtos.Remove(produto);
            await _context.SaveChangesAsync();
        }
    }
}
```

### 6. Consultas LINQ

```csharp
// Filtros
var produtosCaros = await _context.Produtos
    .Where(p => p.Preco > 100)
    .ToListAsync();

// Ordenação
var produtosOrdenados = await _context.Produtos
    .OrderByDescending(p => p.Preco)
    .ThenBy(p => p.Nome)
    .ToListAsync();

// Projeção
var produtosDto = await _context.Produtos
    .Select(p => new ProdutoDto
    {
        Id = p.Id,
        Nome = p.Nome,
        CategoriaNome = p.Categoria.Nome
    })
    .ToListAsync();

// Agregação
var total = await _context.Produtos.CountAsync();
var precoMedio = await _context.Produtos.AverageAsync(p => p.Preco);
var precoMaximo = await _context.Produtos.MaxAsync(p => p.Preco);

// Paginação
var produtosPaginados = await _context.Produtos
    .Skip((pageNumber - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();
```

### 7. Relacionamentos

#### One-to-Many
```csharp
public class Categoria
{
    public int Id { get; set; }
    public string Nome { get; set; }
    
    public ICollection<Produto> Produtos { get; set; }
}

public class Produto
{
    public int Id { get; set; }
    public string Nome { get; set; }
    
    public int CategoriaId { get; set; }
    public Categoria Categoria { get; set; }
}
```

#### Many-to-Many
```csharp
public class Pedido
{
    public int Id { get; set; }
    public DateTime Data { get; set; }
    
    public ICollection<ProdutoPedido> ProdutosPedido { get; set; }
}

public class Produto
{
    public int Id { get; set; }
    public string Nome { get; set; }
    
    public ICollection<ProdutoPedido> ProdutosPedido { get; set; }
}

public class ProdutoPedido
{
    public int ProdutoId { get; set; }
    public Produto Produto { get; set; }
    
    public int PedidoId { get; set; }
    public Pedido Pedido { get; set; }
    
    public int Quantidade { get; set; }
}
```

### 8. Performance e Otimização

```csharp
// Eager Loading
var produtos = await _context.Produtos
    .Include(p => p.Categoria)
    .Include(p => p.ItensPedido)
        .ThenInclude(i => i.Pedido)
    .ToListAsync();

// Explicit Loading
var produto = await _context.Produtos.FindAsync(id);
await _context.Entry(produto)
    .Collection(p => p.ItensPedido)
    .LoadAsync();

// Lazy Loading (instalar Microsoft.EntityFrameworkCore.Proxies)
public class Produto
{
    public int Id { get; set; }
    public string Nome { get; set; }
    
    public virtual Categoria Categoria { get; set; }
}

// AsNoTracking para queries read-only
var produtos = await _context.Produtos
    .AsNoTracking()
    .ToListAsync();

// Bulk Operations (usar EFCore.BulkExtensions)
await _context.BulkInsertAsync(produtos);
```

## 💡 Dicas de Estudo

1. **Use Migrations**: Sempre para mudanças de schema
2. **AsNoTracking**: Use para queries somente leitura
3. **Include com Cuidado**: Evite over-fetching
4. **Índices**: Adicione em colunas frequentemente consultadas
5. **Async/Await**: Sempre use métodos assíncronos

## 📝 Exercícios Práticos

### Básico
1. Crie um DbContext com 2-3 entidades simples
2. Configure conexão com banco de dados
3. Crie e aplique sua primeira migration
4. Implemente operações CRUD básicas

### Intermediário
5. Crie relacionamentos one-to-many
6. Implemente consultas com LINQ
7. Adicione índices e constraints
8. Use Include para carregar dados relacionados
9. Implemente paginação

### Avançado
10. Configure relacionamento many-to-many
11. Use Data Seeding para popular banco
12. Implemente Soft Delete
13. Configure Table-Per-Hierarchy (TPH) inheritance
14. Otimize queries com AsNoTracking

### Projeto
**Sistema de E-commerce (Camada de Dados)**: Implemente:
- Entidades: Produto, Categoria, Cliente, Pedido, ItemPedido
- Relacionamentos apropriados
- Repositories com operações CRUD
- Consultas complexas (produtos por categoria, pedidos por cliente)
- Migrations completas
- Seed data para testes

## 🔗 Recursos Recomendados

### Documentação
- [EF Core Documentation](https://learn.microsoft.com/ef/core/)
- [EF Core Tutorial](https://learn.microsoft.com/ef/core/get-started/overview/first-app)
- [Migrations Overview](https://learn.microsoft.com/ef/core/managing-schemas/migrations/)

### Vídeos
- [Entity Framework Core Tutorial](https://www.youtube.com/watch?v=SryQxUeChMc)
- [EF Core Performance Tips](https://www.youtube.com/watch?v=qkJ9keBmQWo)

### Artigos
- [EF Core Best Practices](https://www.thereformedprogrammer.net/entity-framework-core-performance-tuning-a-worked-example/)
- [Common EF Core Mistakes](https://blog.jetbrains.com/dotnet/2023/06/05/common-mistakes-in-ef-core/)

### Livros
- "Entity Framework Core in Action" - Jon P Smith
- "Programming Entity Framework Core" - Julia Lerman

## ✅ Checklist de Conclusão

Antes de avançar, certifique-se de que você:

- [ ] Sabe configurar DbContext
- [ ] Conhece migrations e sabe criar/aplicar
- [ ] Domina operações CRUD assíncronas
- [ ] Entende relacionamentos entre entidades
- [ ] Sabe usar LINQ para consultas
- [ ] Conhece Include, AsNoTracking
- [ ] Entende diferença entre Eager e Lazy Loading
- [ ] Sabe otimizar queries
- [ ] Completou o projeto de E-commerce (camada de dados)

## ⏭️ Próximo Passo

Após dominar Entity Framework Core, avance para [05-Testes](../05-Testes/README.md) para aprender testing.

---

*"Data is a precious thing and will last longer than the systems themselves." - Tim Berners-Lee*
