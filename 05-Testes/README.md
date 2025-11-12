# 🧪 Testes em .NET

## 🎯 Objetivos de Aprendizado

Ao completar este módulo, você será capaz de:
- Escrever testes unitários eficazes
- Criar testes de integração
- Utilizar frameworks de teste (xUnit, NUnit, MSTest)
- Implementar mocking e test doubles
- Aplicar TDD (Test-Driven Development)
- Medir cobertura de código

## 📚 Conteúdo

### 1. Introdução aos Testes

#### Tipos de Testes
- **Testes Unitários**: Testam unidades isoladas de código
- **Testes de Integração**: Testam integração entre componentes
- **Testes End-to-End**: Testam o sistema completo
- **Testes de Performance**: Testam velocidade e escalabilidade

#### Pirâmide de Testes
```
        /\
       /E2E\      (Poucos)
      /------\
     /  Int   \   (Alguns)
    /----------\
   /   Unit     \ (Muitos)
  /--------------\
```

### 2. xUnit - Framework de Testes

```bash
# Criar projeto de teste
dotnet new xunit -n MeuProjeto.Tests

# Adicionar referência ao projeto principal
dotnet add reference ../MeuProjeto/MeuProjeto.csproj

# Executar testes
dotnet test
```

#### Estrutura Básica
```csharp
public class CalculadoraTests
{
    [Fact]
    public void Somar_DoisNumeros_RetornaSoma()
    {
        // Arrange (Preparar)
        var calculadora = new Calculadora();
        
        // Act (Agir)
        var resultado = calculadora.Somar(2, 3);
        
        // Assert (Verificar)
        Assert.Equal(5, resultado);
    }
    
    [Theory]
    [InlineData(1, 1, 2)]
    [InlineData(5, 5, 10)]
    [InlineData(-2, 3, 1)]
    public void Somar_VariosNumeros_RetornaResultadoCorreto(
        int a, int b, int esperado)
    {
        var calculadora = new Calculadora();
        var resultado = calculadora.Somar(a, b);
        Assert.Equal(esperado, resultado);
    }
}
```

### 3. Assertions Comuns

```csharp
// Igualdade
Assert.Equal(esperado, atual);
Assert.NotEqual(esperado, atual);

// Booleanos
Assert.True(condicao);
Assert.False(condicao);

// Nulos
Assert.Null(objeto);
Assert.NotNull(objeto);

// Strings
Assert.Contains("texto", stringCompleta);
Assert.StartsWith("inicio", string);
Assert.EndsWith("fim", string);

// Coleções
Assert.Empty(lista);
Assert.NotEmpty(lista);
Assert.Contains(item, lista);
Assert.Collection(lista, 
    item => Assert.Equal("primeiro", item),
    item => Assert.Equal("segundo", item));

// Exceções
var exception = Assert.Throws<ArgumentException>(() => 
    metodo.ExecutarComParametroInvalido());
Assert.Equal("mensagem esperada", exception.Message);

// Tipos
Assert.IsType<MinhaClasse>(objeto);
Assert.IsAssignableFrom<IMinhaInterface>(objeto);
```

### 4. Mocking com Moq

```bash
dotnet add package Moq
```

```csharp
public interface IProdutoRepository
{
    Task<Produto> ObterPorIdAsync(int id);
    Task<List<Produto>> ObterTodosAsync();
}

public class ProdutoServiceTests
{
    [Fact]
    public async Task ObterProduto_IdValido_RetornaProduto()
    {
        // Arrange
        var mockRepo = new Mock<IProdutoRepository>();
        mockRepo.Setup(r => r.ObterPorIdAsync(1))
            .ReturnsAsync(new Produto 
            { 
                Id = 1, 
                Nome = "Produto Teste" 
            });
        
        var service = new ProdutoService(mockRepo.Object);
        
        // Act
        var resultado = await service.ObterProdutoAsync(1);
        
        // Assert
        Assert.NotNull(resultado);
        Assert.Equal("Produto Teste", resultado.Nome);
        mockRepo.Verify(r => r.ObterPorIdAsync(1), Times.Once);
    }
    
    [Fact]
    public async Task ObterProduto_IdInvalido_LancaExcecao()
    {
        // Arrange
        var mockRepo = new Mock<IProdutoRepository>();
        mockRepo.Setup(r => r.ObterPorIdAsync(It.IsAny<int>()))
            .ReturnsAsync((Produto)null);
        
        var service = new ProdutoService(mockRepo.Object);
        
        // Act & Assert
        await Assert.ThrowsAsync<NotFoundException>(
            () => service.ObterProdutoAsync(999));
    }
}
```

### 5. Fixtures e Setup

```csharp
// Compartilhar contexto entre testes
public class DatabaseFixture : IDisposable
{
    public AppDbContext Context { get; private set; }
    
    public DatabaseFixture()
    {
        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseInMemoryDatabase("TestDb")
            .Options;
        
        Context = new AppDbContext(options);
        Context.Database.EnsureCreated();
    }
    
    public void Dispose()
    {
        Context.Database.EnsureDeleted();
        Context.Dispose();
    }
}

public class ProdutoRepositoryTests : IClassFixture<DatabaseFixture>
{
    private readonly AppDbContext _context;
    
    public ProdutoRepositoryTests(DatabaseFixture fixture)
    {
        _context = fixture.Context;
    }
    
    [Fact]
    public async Task Adicionar_ProdutoValido_AdicionaAoBanco()
    {
        // Arrange
        var repository = new ProdutoRepository(_context);
        var produto = new Produto { Nome = "Teste", Preco = 10 };
        
        // Act
        await repository.AdicionarAsync(produto);
        
        // Assert
        var produtoSalvo = await _context.Produtos.FirstAsync();
        Assert.Equal("Teste", produtoSalvo.Nome);
    }
}
```

### 6. Testes de Integração

```csharp
public class IntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    
    public IntegrationTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
    }
    
    [Fact]
    public async Task Get_Produtos_RetornaListaDeProdutos()
    {
        // Arrange
        var client = _factory.CreateClient();
        
        // Act
        var response = await client.GetAsync("/api/produtos");
        
        // Assert
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync();
        Assert.Contains("Produto", content);
    }
    
    [Fact]
    public async Task Post_Produto_CriaNovoProduto()
    {
        // Arrange
        var client = _factory.CreateClient();
        var produto = new { Nome = "Novo Produto", Preco = 99.99 };
        var content = new StringContent(
            JsonSerializer.Serialize(produto),
            Encoding.UTF8,
            "application/json");
        
        // Act
        var response = await client.PostAsync("/api/produtos", content);
        
        // Assert
        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
    }
}
```

### 7. TDD (Test-Driven Development)

#### Ciclo Red-Green-Refactor
1. **Red**: Escreva um teste que falha
2. **Green**: Escreva código mínimo para passar
3. **Refactor**: Melhore o código mantendo testes verdes

```csharp
// 1. RED - Escrever teste primeiro
[Fact]
public void CalcularDesconto_ValorMaiorQue100_Retorna10Porcento()
{
    var calculadora = new CalculadoraDesconto();
    var resultado = calculadora.CalcularDesconto(150);
    Assert.Equal(15, resultado);
}

// 2. GREEN - Implementar código simples
public class CalculadoraDesconto
{
    public decimal CalcularDesconto(decimal valor)
    {
        if (valor > 100)
            return valor * 0.1m;
        return 0;
    }
}

// 3. REFACTOR - Melhorar sem quebrar testes
public class CalculadoraDesconto
{
    private const decimal VALOR_MINIMO = 100m;
    private const decimal PERCENTUAL_DESCONTO = 0.1m;
    
    public decimal CalcularDesconto(decimal valor)
    {
        return valor > VALOR_MINIMO 
            ? valor * PERCENTUAL_DESCONTO 
            : 0;
    }
}
```

### 8. Cobertura de Código

```bash
# Instalar ferramenta
dotnet tool install --global coverlet.console

# Executar com cobertura
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover

# Gerar relatório HTML
dotnet tool install --global dotnet-reportgenerator-globaltool
reportgenerator -reports:coverage.opencover.xml -targetdir:coveragereport
```

## 💡 Dicas de Estudo

1. **AAA Pattern**: Sempre use Arrange-Act-Assert
2. **Testes Isolados**: Cada teste deve ser independente
3. **Nome Descritivo**: Use nomes que explicam o teste
4. **Um Assert por Teste**: Prefira testes focados
5. **Mock Apenas Dependências**: Não mock tudo

## 📝 Exercícios Práticos

### Básico
1. Crie projeto de teste xUnit
2. Escreva testes para classe Calculadora
3. Use [Theory] com múltiplos casos
4. Teste exceções com Assert.Throws

### Intermediário
5. Use Moq para mockar dependências
6. Crie testes com fixtures
7. Implemente testes para um repository
8. Escreva testes de integração para uma API
9. Configure InMemory database para testes

### Avançado
10. Pratique TDD em um novo recurso
11. Configure cobertura de código
12. Crie factory customizada para testes de integração
13. Implemente testes parametrizados complexos
14. Use AutoFixture para geração de dados

### Projeto
**Sistema de Carrinho de Compras com TDD**: Implemente usando TDD:
- Adicionar/remover produtos do carrinho
- Calcular total com descontos
- Validar estoque
- Aplicar cupons promocionais
- Alcançar 80%+ de cobertura
- Incluir testes unitários e de integração

## 🔗 Recursos Recomendados

### Documentação
- [Unit Testing in .NET](https://learn.microsoft.com/dotnet/core/testing/)
- [xUnit Documentation](https://xunit.net/)
- [Moq Quickstart](https://github.com/moq/moq4)

### Vídeos
- [Unit Testing C# Code - Tutorial](https://www.youtube.com/watch?v=HYrXogLj7vg)
- [TDD in C#](https://www.youtube.com/watch?v=KtHQGs3zFAM)

### Artigos
- [Best Practices for Unit Testing](https://learn.microsoft.com/dotnet/core/testing/unit-testing-best-practices)
- [Integration Testing in ASP.NET Core](https://learn.microsoft.com/aspnet/core/test/integration-tests)

### Livros
- "The Art of Unit Testing" - Roy Osherove
- "Test Driven Development: By Example" - Kent Beck

## ✅ Checklist de Conclusão

Antes de avançar, certifique-se de que você:

- [ ] Sabe criar e executar testes com xUnit
- [ ] Conhece AAA pattern
- [ ] Domina assertions comuns
- [ ] Sabe usar Moq para mocking
- [ ] Entende fixtures e setup/teardown
- [ ] Consegue escrever testes de integração
- [ ] Conhece TDD e ciclo Red-Green-Refactor
- [ ] Sabe medir cobertura de código
- [ ] Completou projeto com TDD e alta cobertura

## ⏭️ Próximo Passo

Após dominar testes, avance para [06-APIs](../06-APIs/README.md) para aprofundar em desenvolvimento de APIs.

---

*"Code without tests is broken by design." - Jacob Kaplan-Moss*
