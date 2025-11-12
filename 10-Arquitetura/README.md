# 🏗️ Arquitetura de Software

## 🎯 Objetivos de Aprendizado

Ao completar este módulo, você será capaz de:
- Aplicar princípios SOLID
- Implementar Design Patterns
- Criar arquiteturas limpas e escaláveis
- Trabalhar com Domain-Driven Design (DDD)
- Implementar CQRS e Event Sourcing
- Desenvolver microservices

## 📚 Conteúdo

### 1. Princípios SOLID

#### Single Responsibility Principle (SRP)
```csharp
// ❌ ERRADO - Múltiplas responsabilidades
public class Usuario
{
    public void Salvar() { /* salva no DB */ }
    public void EnviarEmail() { /* envia email */ }
    public void GerarRelatorio() { /* gera PDF */ }
}

// ✅ CORRETO - Uma responsabilidade por classe
public class Usuario
{
    public string Nome { get; set; }
    public string Email { get; set; }
}

public class UsuarioRepository
{
    public void Salvar(Usuario usuario) { /* salva no DB */ }
}

public class EmailService
{
    public void EnviarBemVindo(Usuario usuario) { /* envia email */ }
}

public class RelatorioService
{
    public byte[] GerarRelatorioUsuario(Usuario usuario) { /* gera PDF */ }
}
```

#### Open/Closed Principle (OCP)
```csharp
// Aberto para extensão, fechado para modificação
public abstract class DescontoStrategy
{
    public abstract decimal CalcularDesconto(decimal valor);
}

public class DescontoNatal : DescontoStrategy
{
    public override decimal CalcularDesconto(decimal valor)
    {
        return valor * 0.15m;
    }
}

public class DescontoBlackFriday : DescontoStrategy
{
    public override decimal CalcularDesconto(decimal valor)
    {
        return valor * 0.30m;
    }
}

public class CalculadoraPreco
{
    public decimal CalcularPrecoFinal(decimal valor, DescontoStrategy desconto)
    {
        return valor - desconto.CalcularDesconto(valor);
    }
}
```

#### Liskov Substitution Principle (LSP)
```csharp
// Subtipos devem ser substituíveis por seus tipos base
public abstract class Ave
{
    public abstract void Mover();
}

public class Pardal : Ave
{
    public override void Mover()
    {
        Voar();
    }
    
    private void Voar() { /* voa */ }
}

public class Pinguim : Ave
{
    public override void Mover()
    {
        Nadar();
    }
    
    private void Nadar() { /* nada */ }
}
```

#### Interface Segregation Principle (ISP)
```csharp
// ❌ ERRADO - Interface grande demais
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

// ✅ CORRETO - Interfaces segregadas
public interface IWorkable
{
    void Work();
}

public interface IFeedable
{
    void Eat();
}

public interface ISleepable
{
    void Sleep();
}

public class Human : IWorkable, IFeedable, ISleepable
{
    public void Work() { }
    public void Eat() { }
    public void Sleep() { }
}

public class Robot : IWorkable
{
    public void Work() { }
}
```

#### Dependency Inversion Principle (DIP)
```csharp
// Dependa de abstrações, não de implementações
public interface INotificationService
{
    void Enviar(string mensagem, string destinatario);
}

public class EmailService : INotificationService
{
    public void Enviar(string mensagem, string destinatario)
    {
        // Enviar email
    }
}

public class SmsService : INotificationService
{
    public void Enviar(string mensagem, string destinatario)
    {
        // Enviar SMS
    }
}

public class PedidoService
{
    private readonly INotificationService _notificationService;
    
    public PedidoService(INotificationService notificationService)
    {
        _notificationService = notificationService;
    }
    
    public void ProcessarPedido(Pedido pedido)
    {
        // Processar pedido
        _notificationService.Enviar("Pedido confirmado", pedido.ClienteEmail);
    }
}
```

### 2. Design Patterns

#### Creational Patterns

**Factory Method**
```csharp
public interface IPaymentProcessor
{
    void ProcessPayment(decimal amount);
}

public class CreditCardProcessor : IPaymentProcessor
{
    public void ProcessPayment(decimal amount) { }
}

public class PayPalProcessor : IPaymentProcessor
{
    public void ProcessPayment(decimal amount) { }
}

public class PaymentProcessorFactory
{
    public IPaymentProcessor Create(string type)
    {
        return type switch
        {
            "credit-card" => new CreditCardProcessor(),
            "paypal" => new PayPalProcessor(),
            _ => throw new ArgumentException("Invalid payment type")
        };
    }
}
```

**Singleton**
```csharp
public class ConfigurationManager
{
    private static readonly Lazy<ConfigurationManager> _instance = 
        new Lazy<ConfigurationManager>(() => new ConfigurationManager());
    
    private ConfigurationManager()
    {
        // Load configuration
    }
    
    public static ConfigurationManager Instance => _instance.Value;
    
    public string GetSetting(string key) { return ""; }
}
```

#### Structural Patterns

**Repository Pattern**
```csharp
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

public class Repository<T> : IRepository<T> where T : class
{
    private readonly AppDbContext _context;
    private readonly DbSet<T> _dbSet;
    
    public Repository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public async Task<T?> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }
    
    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }
    
    public async Task AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        await _context.SaveChangesAsync();
    }
    
    public async Task UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        await _context.SaveChangesAsync();
    }
    
    public async Task DeleteAsync(int id)
    {
        var entity = await GetByIdAsync(id);
        if (entity != null)
        {
            _dbSet.Remove(entity);
            await _context.SaveChangesAsync();
        }
    }
}
```

**Decorator Pattern**
```csharp
public interface IOrderService
{
    Task ProcessOrder(Order order);
}

public class OrderService : IOrderService
{
    public async Task ProcessOrder(Order order)
    {
        // Processar pedido
    }
}

public class LoggingOrderService : IOrderService
{
    private readonly IOrderService _inner;
    private readonly ILogger _logger;
    
    public LoggingOrderService(IOrderService inner, ILogger logger)
    {
        _inner = inner;
        _logger = logger;
    }
    
    public async Task ProcessOrder(Order order)
    {
        _logger.LogInformation($"Processing order {order.Id}");
        await _inner.ProcessOrder(order);
        _logger.LogInformation($"Order {order.Id} processed");
    }
}
```

#### Behavioral Patterns

**Strategy Pattern**
```csharp
public interface IShippingStrategy
{
    decimal CalculateCost(decimal weight, string destination);
}

public class StandardShipping : IShippingStrategy
{
    public decimal CalculateCost(decimal weight, string destination)
    {
        return weight * 0.5m;
    }
}

public class ExpressShipping : IShippingStrategy
{
    public decimal CalculateCost(decimal weight, string destination)
    {
        return weight * 1.5m;
    }
}

public class ShippingCalculator
{
    public decimal Calculate(decimal weight, string destination, IShippingStrategy strategy)
    {
        return strategy.CalculateCost(weight, destination);
    }
}
```

### 3. Clean Architecture

```
┌─────────────────────────────────────────────────┐
│              Presentation Layer                 │
│         (API, MVC, Razor Pages)                 │
├─────────────────────────────────────────────────┤
│           Application Layer                     │
│     (Use Cases, DTOs, Interfaces)               │
├─────────────────────────────────────────────────┤
│             Domain Layer                        │
│   (Entities, Value Objects, Domain Events)      │
├─────────────────────────────────────────────────┤
│          Infrastructure Layer                   │
│  (Data Access, External Services, File I/O)     │
└─────────────────────────────────────────────────┘
```

#### Estrutura de Projeto
```
Solution/
├── src/
│   ├── Domain/              # Entidades, Value Objects
│   ├── Application/         # Use Cases, Interfaces
│   ├── Infrastructure/      # EF Core, External APIs
│   └── WebAPI/             # Controllers, Startup
└── tests/
    ├── Domain.Tests/
    ├── Application.Tests/
    └── Integration.Tests/
```

#### Domain Layer
```csharp
// Domain/Entities/Order.cs
public class Order
{
    public int Id { get; private set; }
    public DateTime CreatedAt { get; private set; }
    public OrderStatus Status { get; private set; }
    public decimal Total { get; private set; }
    
    private readonly List<OrderItem> _items = new();
    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();
    
    public void AddItem(Product product, int quantity)
    {
        if (quantity <= 0)
            throw new InvalidOperationException("Quantity must be positive");
        
        var item = new OrderItem(product, quantity);
        _items.Add(item);
        RecalculateTotal();
    }
    
    public void ConfirmOrder()
    {
        if (!_items.Any())
            throw new InvalidOperationException("Cannot confirm empty order");
        
        Status = OrderStatus.Confirmed;
    }
    
    private void RecalculateTotal()
    {
        Total = _items.Sum(i => i.Subtotal);
    }
}
```

#### Application Layer
```csharp
// Application/Orders/Commands/CreateOrderCommand.cs
public record CreateOrderCommand(int CustomerId, List<OrderItemDto> Items) 
    : IRequest<int>;

// Application/Orders/Handlers/CreateOrderCommandHandler.cs
public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, int>
{
    private readonly IOrderRepository _orderRepository;
    private readonly IProductRepository _productRepository;
    
    public CreateOrderCommandHandler(
        IOrderRepository orderRepository,
        IProductRepository productRepository)
    {
        _orderRepository = orderRepository;
        _productRepository = productRepository;
    }
    
    public async Task<int> Handle(
        CreateOrderCommand request, 
        CancellationToken cancellationToken)
    {
        var order = new Order();
        
        foreach (var item in request.Items)
        {
            var product = await _productRepository.GetByIdAsync(item.ProductId);
            order.AddItem(product, item.Quantity);
        }
        
        order.ConfirmOrder();
        await _orderRepository.AddAsync(order);
        
        return order.Id;
    }
}
```

### 4. CQRS (Command Query Responsibility Segregation)

```csharp
// Commands (Write)
public record CreateProductCommand(string Name, decimal Price) : IRequest<int>;

public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, int>
{
    private readonly AppDbContext _context;
    
    public async Task<int> Handle(CreateProductCommand request, CancellationToken ct)
    {
        var product = new Product { Name = request.Name, Price = request.Price };
        _context.Products.Add(product);
        await _context.SaveChangesAsync(ct);
        return product.Id;
    }
}

// Queries (Read)
public record GetProductQuery(int Id) : IRequest<ProductDto>;

public class GetProductQueryHandler : IRequestHandler<GetProductQuery, ProductDto>
{
    private readonly AppDbContext _context;
    
    public async Task<ProductDto> Handle(GetProductQuery request, CancellationToken ct)
    {
        return await _context.Products
            .Where(p => p.Id == request.Id)
            .Select(p => new ProductDto { Id = p.Id, Name = p.Name, Price = p.Price })
            .FirstOrDefaultAsync(ct);
    }
}
```

### 5. Domain-Driven Design (DDD)

#### Aggregates
```csharp
public class Order  // Aggregate Root
{
    private readonly List<OrderLine> _orderLines = new();
    
    public void AddProduct(Product product, int quantity)
    {
        var existingLine = _orderLines
            .FirstOrDefault(l => l.ProductId == product.Id);
        
        if (existingLine != null)
        {
            existingLine.IncreaseQuantity(quantity);
        }
        else
        {
            _orderLines.Add(new OrderLine(product.Id, quantity, product.Price));
        }
    }
}

public class OrderLine  // Entity dentro do Aggregate
{
    public int ProductId { get; private set; }
    public int Quantity { get; private set; }
    public decimal UnitPrice { get; private set; }
    
    public void IncreaseQuantity(int amount)
    {
        Quantity += amount;
    }
}
```

#### Value Objects
```csharp
public record Money
{
    public decimal Amount { get; init; }
    public string Currency { get; init; }
    
    public Money(decimal amount, string currency)
    {
        if (amount < 0)
            throw new ArgumentException("Amount cannot be negative");
        
        Amount = amount;
        Currency = currency ?? throw new ArgumentNullException(nameof(currency));
    }
    
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Cannot add different currencies");
        
        return new Money(Amount + other.Amount, Currency);
    }
}
```

### 6. Microservices

#### Estrutura
```
Microservices/
├── Services/
│   ├── Catalog.API/
│   ├── Ordering.API/
│   ├── Payment.API/
│   └── Identity.API/
├── ApiGateway/
└── Shared/
    ├── EventBus/
    └── Common/
```

#### Message Bus (RabbitMQ/Azure Service Bus)
```csharp
public interface IEventBus
{
    Task PublishAsync<T>(T @event) where T : IntegrationEvent;
    void Subscribe<T, TH>() 
        where T : IntegrationEvent
        where TH : IIntegrationEventHandler<T>;
}

public class OrderCreatedEvent : IntegrationEvent
{
    public int OrderId { get; set; }
    public decimal Total { get; set; }
}

public class OrderCreatedEventHandler : IIntegrationEventHandler<OrderCreatedEvent>
{
    public async Task Handle(OrderCreatedEvent @event)
    {
        // Processar evento em outro microservice
    }
}
```

## 💡 Dicas de Estudo

1. **Comece com SOLID**: Base de tudo
2. **Aprenda Padrões Gradualmente**: Não decore, entenda quando usar
3. **Pratique Refactoring**: Transforme código ruim em bom
4. **Leia Código de Outros**: Projetos open-source
5. **Não Over-Engineer**: Simplicidade também é arquitetura

## 📝 Exercícios Práticos

### Básico
1. Refatore código violando SOLID
2. Implemente Repository Pattern
3. Use Strategy Pattern em um cenário real
4. Crie Factory para objetos complexos

### Intermediário
5. Implemente Clean Architecture em projeto
6. Use MediatR para CQRS
7. Crie Value Objects e Entities DDD
8. Implemente Unit of Work Pattern

### Avançado
9. Arquitetura completa com CQRS
10. Implemente Event Sourcing
11. Crie sistema de microservices
12. Implemente Saga Pattern
13. Configure API Gateway

### Projeto
**E-commerce com Clean Architecture e DDD**:
- Clean Architecture completa
- Domain-Driven Design
- CQRS com MediatR
- Event Bus para comunicação
- Múltiplos contextos limitados (Bounded Contexts)
- API Gateway
- Documentação de decisões arquiteturais

## 🔗 Recursos Recomendados

### Livros
- "Clean Architecture" - Robert C. Martin
- "Domain-Driven Design" - Eric Evans
- "Patterns of Enterprise Application Architecture" - Martin Fowler
- "Building Microservices" - Sam Newman

### Cursos
- [Clean Architecture (Pluralsight)](https://www.pluralsight.com/courses/clean-architecture-patterns-practices-principles)
- [Domain-Driven Design Fundamentals](https://www.pluralsight.com/courses/fundamentals-domain-driven-design)

### Artigos
- [Microsoft Architecture Guide](https://learn.microsoft.com/dotnet/architecture/)
- [Clean Architecture with .NET Core](https://jasontaylor.dev/clean-architecture-getting-started/)

## ✅ Checklist de Conclusão

Antes de concluir, certifique-se de que você:

- [ ] Domina princípios SOLID
- [ ] Conhece principais Design Patterns
- [ ] Entende Clean Architecture
- [ ] Sabe implementar Repository Pattern
- [ ] Conhece CQRS
- [ ] Entende conceitos de DDD
- [ ] Tem noção de microservices
- [ ] Sabe quando aplicar cada padrão
- [ ] Completou projeto arquitetural completo

## 🎓 Parabéns!

Você chegou ao fim do roadmap de aprendizado .NET! Continue praticando, construindo projetos e aprendendo. A jornada de um desenvolvedor nunca termina.

---

*"Any fool can write code that a computer can understand. Good programmers write code that humans can understand." - Martin Fowler*
