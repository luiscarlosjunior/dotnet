# 🔤 C# - Linguagem de Programação

## 🎯 Objetivos de Aprendizado

Ao completar este módulo, você será capaz de:
- Dominar a sintaxe e recursos da linguagem C#
- Aplicar conceitos de Programação Orientada a Objetos
- Trabalhar com tipos de dados, coleções e LINQ
- Utilizar recursos modernos do C# (async/await, pattern matching, etc.)
- Escrever código limpo e idiomático

## 📚 Conteúdo

### 1. Sintaxe Básica

#### Tipos de Dados
```csharp
// Tipos Primitivos
int numero = 42;
double preco = 19.99;
string nome = "Maria";
bool ativo = true;
char inicial = 'M';

// Tipos Nullable
int? idade = null;

// Var (inferência de tipo)
var mensagem = "Olá"; // string
var contador = 10;     // int
```

#### Controle de Fluxo
```csharp
// If/Else
if (idade >= 18)
{
    Console.WriteLine("Maior de idade");
}
else
{
    Console.WriteLine("Menor de idade");
}

// Switch Expression (C# 8+)
var tipo = tamanho switch
{
    < 10 => "Pequeno",
    < 50 => "Médio",
    _ => "Grande"
};

// Loops
for (int i = 0; i < 10; i++) { }
while (condicao) { }
foreach (var item in lista) { }
```

### 2. Programação Orientada a Objetos

#### Classes e Objetos
```csharp
public class Pessoa
{
    // Propriedades
    public string Nome { get; set; }
    public int Idade { get; set; }
    
    // Propriedade Somente Leitura
    public string NomeCompleto { get; }
    
    // Construtor
    public Pessoa(string nome, int idade)
    {
        Nome = nome;
        Idade = idade;
    }
    
    // Método
    public void ApresentarSe()
    {
        Console.WriteLine($"Olá, meu nome é {Nome}");
    }
}

// Uso
var pessoa = new Pessoa("João", 30);
pessoa.ApresentarSe();
```

#### Herança
```csharp
public class Animal
{
    public virtual void FazerSom()
    {
        Console.WriteLine("Som genérico");
    }
}

public class Cachorro : Animal
{
    public override void FazerSom()
    {
        Console.WriteLine("Au au!");
    }
}
```

#### Interfaces
```csharp
public interface IRepositorio<T>
{
    void Adicionar(T item);
    T ObterPorId(int id);
    IEnumerable<T> ObterTodos();
}

public class RepositorioPessoa : IRepositorio<Pessoa>
{
    public void Adicionar(Pessoa item) { }
    public Pessoa ObterPorId(int id) { return null; }
    public IEnumerable<Pessoa> ObterTodos() { return null; }
}
```

### 3. Coleções

#### Arrays
```csharp
int[] numeros = new int[5];
string[] nomes = { "Ana", "Bruno", "Carlos" };
```

#### Listas
```csharp
List<string> frutas = new List<string>();
frutas.Add("Maçã");
frutas.Add("Banana");
frutas.Remove("Maçã");

// Inicialização
var cores = new List<string> { "Vermelho", "Verde", "Azul" };
```

#### Dicionários
```csharp
Dictionary<string, int> idades = new Dictionary<string, int>
{
    { "João", 30 },
    { "Maria", 25 }
};

idades["Pedro"] = 35;
int idadeJoao = idades["João"];
```

### 4. LINQ (Language Integrated Query)

```csharp
var numeros = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Filtrar
var pares = numeros.Where(n => n % 2 == 0);

// Mapear
var dobrados = numeros.Select(n => n * 2);

// Ordenar
var ordenados = numeros.OrderByDescending(n => n);

// Agregação
var soma = numeros.Sum();
var media = numeros.Average();
var maximo = numeros.Max();

// Query Syntax
var query = from n in numeros
            where n > 5
            orderby n
            select n * 2;
```

### 5. Recursos Modernos

#### Records (C# 9+)
```csharp
public record Pessoa(string Nome, int Idade);

var pessoa1 = new Pessoa("João", 30);
var pessoa2 = pessoa1 with { Idade = 31 };
```

#### Pattern Matching
```csharp
object obj = "texto";

var resultado = obj switch
{
    string s => $"String: {s}",
    int i => $"Int: {i}",
    null => "Null",
    _ => "Outro tipo"
};
```

#### Null-Coalescing
```csharp
string? nome = null;
string nomeCompleto = nome ?? "Anônimo";

// Null-coalescing assignment
nome ??= "Valor padrão";
```

#### Async/Await
```csharp
public async Task<string> BuscarDadosAsync()
{
    using var client = new HttpClient();
    var resposta = await client.GetStringAsync("https://api.exemplo.com");
    return resposta;
}
```

### 6. Exception Handling

```csharp
try
{
    int resultado = 10 / 0;
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"Erro: {ex.Message}");
}
catch (Exception ex)
{
    Console.WriteLine($"Erro genérico: {ex.Message}");
}
finally
{
    Console.WriteLine("Sempre executado");
}
```

### 7. Delegates e Events

```csharp
// Delegate
public delegate void NotificarHandler(string mensagem);

// Event
public class Publisher
{
    public event NotificarHandler? OnNotificar;
    
    public void Notificar(string msg)
    {
        OnNotificar?.Invoke(msg);
    }
}

// Lambda expressions
Func<int, int, int> somar = (a, b) => a + b;
Action<string> imprimir = msg => Console.WriteLine(msg);
```

## 💡 Dicas de Estudo

1. **Pratique POO**: Crie classes que representem objetos do mundo real
2. **Use LINQ**: É poderoso e torna o código mais legível
3. **Aprenda Async/Await**: Fundamental para aplicações modernas
4. **Conheça os Recursos Novos**: C# evolui constantemente
5. **Leia Código de Outros**: GitHub tem milhões de projetos em C#

## 📝 Exercícios Práticos

### Básico
1. Crie uma classe `Livro` com propriedades (Título, Autor, Ano)
2. Implemente uma classe `Calculadora` com métodos estáticos
3. Use LINQ para filtrar uma lista de números ímpares
4. Crie um programa que lê e armazena nomes em uma List

### Intermediário
5. Implemente uma hierarquia de classes (Veículo -> Carro, Moto)
6. Crie uma interface `IArmazenamento` e implemente para arquivo e memória
7. Use LINQ para agrupar e agregar dados
8. Implemente um sistema de eventos (Publisher/Subscriber)

### Avançado
9. Crie um sistema de gerenciamento de biblioteca com classes, herança e interfaces
10. Implemente operações assíncronas para leitura de arquivos
11. Use pattern matching para validação de dados complexos
12. Crie um mini-framework usando generics e delegates

### Projeto
**Sistema de Gerenciamento de Tarefas**: Crie uma aplicação console que:
- Permite adicionar, listar, completar e remover tarefas
- Usa classes e POO adequadamente
- Salva/carrega dados de arquivo JSON
- Usa LINQ para filtrar e ordenar tarefas
- Implementa tratamento de exceções

## 🔗 Recursos Recomendados

### Documentação
- [C# Programming Guide](https://learn.microsoft.com/dotnet/csharp/programming-guide/)
- [C# Language Reference](https://learn.microsoft.com/dotnet/csharp/language-reference/)
- [What's New in C#](https://learn.microsoft.com/dotnet/csharp/whats-new/)

### Livros
- "C# in Depth" - Jon Skeet
- "Pro C# 10 with .NET 6" - Andrew Troelsen
- "Effective C#" - Bill Wagner

### Cursos
- [C# Fundamentals (Microsoft Learn)](https://learn.microsoft.com/training/paths/csharp-first-steps/)
- [LINQ in C#](https://learn.microsoft.com/dotnet/csharp/programming-guide/concepts/linq/)

### Prática
- [Exercism - C# Track](https://exercism.org/tracks/csharp)
- [LeetCode](https://leetcode.com/)
- [HackerRank C#](https://www.hackerrank.com/domains/tutorials/10-days-of-csharp)

## ✅ Checklist de Conclusão

Antes de avançar, certifique-se de que você:

- [ ] Domina tipos de dados e controle de fluxo
- [ ] Entende POO (classes, herança, interfaces)
- [ ] Sabe trabalhar com coleções (List, Dictionary, Array)
- [ ] Conhece LINQ e sabe usar Where, Select, OrderBy
- [ ] Entende delegates, events e lambda expressions
- [ ] Sabe tratar exceções adequadamente
- [ ] Conhece recursos modernos (records, pattern matching)
- [ ] Consegue criar classes e organizar código
- [ ] Completou pelo menos o projeto de Gerenciamento de Tarefas

## ⏭️ Próximo Passo

Após dominar C#, avance para [03-AspNetCore](../03-AspNetCore/README.md) para aprender desenvolvimento web.

---

*"Code is like humor. When you have to explain it, it's bad." - Cory House*
