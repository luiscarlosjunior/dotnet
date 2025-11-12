# 📘 Fundamentos do .NET

## 🎯 Objetivos de Aprendizado

Ao completar este módulo, você será capaz de:
- Entender o que é .NET e sua evolução
- Conhecer os componentes do ecossistema .NET
- Instalar e configurar o ambiente de desenvolvimento
- Criar e executar seu primeiro programa em C#
- Compreender conceitos básicos de compilação e execução

## 📚 Conteúdo

### 1. O que é .NET?

#### História e Evolução
- **.NET Framework** (2002): Framework Windows-only
- **.NET Core** (2016): Cross-platform, open-source
- **.NET 5+** (2020): Unificação do .NET Framework e .NET Core

#### Componentes Principais
- **CLR (Common Language Runtime)**: Runtime que executa aplicações .NET
- **BCL (Base Class Library)**: Biblioteca de classes base
- **SDK**: Ferramentas de desenvolvimento
- **Runtime**: Necessário para executar aplicações

### 2. O Ecossistema .NET

#### Linguagens Suportadas
- **C#**: Principal linguagem (foco deste guia)
- **F#**: Linguagem funcional
- **Visual Basic**: Legado

#### Tipos de Aplicações
- Console Applications
- Web Applications (ASP.NET Core)
- Desktop Applications (WPF, WinForms, MAUI)
- Mobile Applications (.NET MAUI)
- APIs e Microservices
- Cloud Applications (Azure Functions)
- IoT Applications

### 3. .NET CLI (Command Line Interface)

#### Comandos Essenciais
```bash
# Verificar versão instalada
dotnet --version

# Listar SDKs instalados
dotnet --list-sdks

# Criar novo projeto
dotnet new console -n MeuProjeto

# Compilar projeto
dotnet build

# Executar projeto
dotnet run

# Publicar aplicação
dotnet publish
```

### 4. Estrutura de um Projeto .NET

```
MeuProjeto/
├── MeuProjeto.csproj    # Arquivo de projeto (XML)
├── Program.cs           # Código principal
├── bin/                 # Binários compilados
└── obj/                 # Arquivos temporários de build
```

#### Arquivo .csproj
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
</Project>
```

### 5. Seu Primeiro Programa

#### Hello World Moderno (C# 10+)
```csharp
// Program.cs
Console.WriteLine("Olá, .NET!");
```

#### Hello World Tradicional
```csharp
using System;

namespace MeuProjeto
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Olá, .NET!");
        }
    }
}
```

## 🛠️ Configuração do Ambiente

### Windows
1. Baixar [.NET SDK](https://dotnet.microsoft.com/download)
2. Instalar [Visual Studio Community](https://visualstudio.microsoft.com/) ou [VS Code](https://code.visualstudio.com/)
3. Se usar VS Code, instalar extensão C#

### Linux/macOS
```bash
# Ubuntu/Debian
wget https://dot.net/v1/dotnet-install.sh
chmod +x dotnet-install.sh
./dotnet-install.sh

# macOS com Homebrew
brew install --cask dotnet-sdk
```

### Verificação da Instalação
```bash
dotnet --version
dotnet --info
```

## 💡 Dicas de Estudo

1. **Pratique Diariamente**: Crie pequenos projetos todos os dias
2. **Use a Documentação**: [Microsoft Learn](https://learn.microsoft.com/dotnet/) é seu melhor amigo
3. **Experimente no VS Code**: Leve e rápido para aprender
4. **Entenda os Erros**: Leia mensagens de erro com atenção

## 📝 Exercícios Práticos

### Básico
1. Instale o .NET SDK e verifique a versão
2. Crie um projeto console chamado "PrimeiroPrograma"
3. Modifique o Hello World para exibir seu nome
4. Faça o programa perguntar seu nome e responder personalizado

### Intermediário
5. Crie um programa que calcula a soma de dois números
6. Faça um programa que lê dados do usuário via console
7. Experimente diferentes templates do .NET (`dotnet new --list`)
8. Compile e publique seu programa para diferentes sistemas operacionais

### Projeto
**Calculadora Console**: Crie uma calculadora que:
- Lê dois números do usuário
- Pergunta qual operação (+, -, *, /)
- Exibe o resultado
- Trata erros (divisão por zero, entrada inválida)

## 🔗 Recursos Recomendados

### Documentação
- [Tour do C#](https://learn.microsoft.com/dotnet/csharp/tour-of-csharp/)
- [.NET Fundamentals](https://learn.microsoft.com/dotnet/fundamentals/)
- [CLI Reference](https://learn.microsoft.com/dotnet/core/tools/)

### Vídeos
- [.NET for Beginners (Microsoft)](https://www.youtube.com/playlist?list=PLdo4fOcmZ0oUwBEC2bnwPtHqbU8Vmh_tj)
- [C# Tutorial - Full Course for Beginners](https://www.youtube.com/watch?v=GhQdlIFylQ8)

### Artigos
- [Introduction to .NET](https://learn.microsoft.com/dotnet/core/introduction)
- [What is .NET? Introduction and overview](https://dotnet.microsoft.com/learn/dotnet/what-is-dotnet)

## ✅ Checklist de Conclusão

Antes de avançar para o próximo módulo, certifique-se de que você:

- [ ] Instalou o .NET SDK com sucesso
- [ ] Configurou sua IDE/Editor de código
- [ ] Entende o que é CLR e BCL
- [ ] Sabe criar um projeto com `dotnet new`
- [ ] Consegue compilar e executar um programa
- [ ] Entende a estrutura básica de um projeto .NET
- [ ] Conhece os comandos básicos do .NET CLI
- [ ] Completou pelo menos 3 exercícios práticos

## ⏭️ Próximo Passo

Após dominar os fundamentos, avance para [02-CSharp](../02-CSharp/README.md) para aprofundar seus conhecimentos na linguagem C#.

---

*"A jornada de mil milhas começa com um único passo." - Lao Tzu*
