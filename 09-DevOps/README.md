# 🚀 DevOps - CI/CD, Docker e Deployment

## 🎯 Objetivos de Aprendizado

Ao completar este módulo, você será capaz de:
- Containerizar aplicações .NET com Docker
- Configurar pipelines de CI/CD
- Fazer deploy em diferentes ambientes
- Trabalhar com Kubernetes
- Implementar monitoring e logging
- Automatizar processos de build e deploy

## 📚 Conteúdo

### 1. Docker

#### Dockerfile para ASP.NET Core
```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copiar arquivos de projeto e restaurar dependências
COPY ["MeuApp/MeuApp.csproj", "MeuApp/"]
RUN dotnet restore "MeuApp/MeuApp.csproj"

# Copiar todo o código e compilar
COPY . .
WORKDIR "/src/MeuApp"
RUN dotnet build "MeuApp.csproj" -c Release -o /app/build

# Publish stage
FROM build AS publish
RUN dotnet publish "MeuApp.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
EXPOSE 80
EXPOSE 443

COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MeuApp.dll"]
```

#### Docker Compose
```yaml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "5000:80"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__Default=Server=db;Database=MeuDb;User=sa;Password=SenhaForte123!
    depends_on:
      - db
    networks:
      - app-network

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=SenhaForte123!
    ports:
      - "1433:1433"
    volumes:
      - sqldata:/var/opt/mssql
    networks:
      - app-network

volumes:
  sqldata:

networks:
  app-network:
    driver: bridge
```

#### Comandos Docker Essenciais
```bash
# Build image
docker build -t meuapp:latest .

# Run container
docker run -d -p 5000:80 --name meuapp meuapp:latest

# Ver logs
docker logs meuapp

# Executar comando no container
docker exec -it meuapp bash

# Parar e remover container
docker stop meuapp
docker rm meuapp

# Docker Compose
docker-compose up -d
docker-compose down
docker-compose logs -f
```

### 2. GitHub Actions

#### CI/CD Pipeline
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  DOTNET_VERSION: '8.0.x'
  
jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}
    
    - name: Restore dependencies
      run: dotnet restore
    
    - name: Build
      run: dotnet build --no-restore --configuration Release
    
    - name: Test
      run: dotnet test --no-build --configuration Release --verbosity normal
    
    - name: Publish
      run: dotnet publish -c Release -o ./publish
    
    - name: Upload artifact
      uses: actions/upload-artifact@v3
      with:
        name: app
        path: ./publish

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Download artifact
      uses: actions/download-artifact@v3
      with:
        name: app
        path: ./publish
    
    - name: Deploy to Azure
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ./publish
```

#### Docker Build and Push
```yaml
name: Docker Build and Push

on:
  push:
    branches: [ main ]
    tags:
      - 'v*'

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: |
          meuusuario/meuapp:latest
          meuusuario/meuapp:${{ github.sha }}
```

### 3. Azure DevOps

#### Pipeline YAML
```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
    - main
    - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '8.0.x'

stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - task: UseDotNet@2
      inputs:
        version: $(dotnetVersion)
        
    - task: DotNetCoreCLI@2
      displayName: 'Restore'
      inputs:
        command: 'restore'
        
    - task: DotNetCoreCLI@2
      displayName: 'Build'
      inputs:
        command: 'build'
        arguments: '--configuration $(buildConfiguration)'
        
    - task: DotNetCoreCLI@2
      displayName: 'Test'
      inputs:
        command: 'test'
        arguments: '--configuration $(buildConfiguration) --collect:"XPlat Code Coverage"'
        
    - task: DotNetCoreCLI@2
      displayName: 'Publish'
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
        
    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'

- stage: Deploy
  dependsOn: Build
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployJob
    environment: 'Production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Azure-Connection'
              appName: 'meu-app-web'
              package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

### 4. Kubernetes

#### Deployment Manifest
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: meuapp-deployment
  labels:
    app: meuapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: meuapp
  template:
    metadata:
      labels:
        app: meuapp
    spec:
      containers:
      - name: meuapp
        image: meuusuario/meuapp:latest
        ports:
        - containerPort: 80
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: "Production"
        - name: ConnectionStrings__Default
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: connection-string
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: meuapp-service
spec:
  selector:
    app: meuapp
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

#### ConfigMap e Secrets
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  appsettings.json: |
    {
      "Logging": {
        "LogLevel": {
          "Default": "Information"
        }
      }
    }
---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  connection-string: "Server=db;Database=MeuDb;User=sa;Password=SenhaForte123!"
```

#### Comandos Kubectl
```bash
# Apply manifests
kubectl apply -f deployment.yaml
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml

# Ver recursos
kubectl get pods
kubectl get services
kubectl get deployments

# Logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>

# Describe
kubectl describe pod <pod-name>

# Scale
kubectl scale deployment meuapp-deployment --replicas=5

# Delete
kubectl delete deployment meuapp-deployment
```

### 5. Logging

#### Serilog
```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
dotnet add package Serilog.Sinks.Seq
```

```csharp
// Program.cs
using Serilog;

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
    .Enrich.FromLogContext()
    .Enrich.WithMachineName()
    .Enrich.WithEnvironmentName()
    .WriteTo.Console()
    .WriteTo.File("logs/app-.log", rollingInterval: RollingInterval.Day)
    .WriteTo.Seq("http://localhost:5341")
    .CreateLogger();

try
{
    Log.Information("Starting web application");
    
    var builder = WebApplication.CreateBuilder(args);
    builder.Host.UseSerilog();
    
    // Resto da configuração...
    
    var app = builder.Build();
    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}
```

### 6. Monitoring

#### Health Checks
```csharp
builder.Services.AddHealthChecks()
    .AddDbContextCheck<AppDbContext>()
    .AddUrlGroup(new Uri("https://api.externa.com"), "API Externa")
    .AddRedis(configuration["Redis:ConnectionString"]);

app.MapHealthChecks("/health");
app.MapHealthChecks("/ready");
```

#### Application Insights
```csharp
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = builder.Configuration["ApplicationInsights:ConnectionString"];
});

// Tracking customizado
public class MyService
{
    private readonly TelemetryClient _telemetry;
    
    public MyService(TelemetryClient telemetry)
    {
        _telemetry = telemetry;
    }
    
    public void ProcessOrder(Order order)
    {
        var sw = Stopwatch.StartNew();
        try
        {
            // Processar pedido
            _telemetry.TrackEvent("OrderProcessed", 
                new Dictionary<string, string> 
                {
                    { "OrderId", order.Id.ToString() }
                });
        }
        catch (Exception ex)
        {
            _telemetry.TrackException(ex);
            throw;
        }
        finally
        {
            sw.Stop();
            _telemetry.TrackMetric("OrderProcessingTime", sw.ElapsedMilliseconds);
        }
    }
}
```

### 7. Configuration Management

#### Multiple Environments
```json
// appsettings.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}

// appsettings.Development.json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=DevDb;..."
  }
}

// appsettings.Production.json
{
  "ConnectionStrings": {
    "Default": "Server=prod-server;Database=ProdDb;..."
  }
}
```

#### Environment Variables
```csharp
var connectionString = builder.Configuration.GetConnectionString("Default")
    ?? Environment.GetEnvironmentVariable("DB_CONNECTION_STRING");
```

## 💡 Dicas de Estudo

1. **Aprenda Docker Primeiro**: Base para tudo
2. **Automatize Tudo**: CI/CD economiza muito tempo
3. **Monitore Sempre**: Logs e métricas são essenciais
4. **Infrastructure as Code**: Versione sua infraestrutura
5. **Security First**: Nunca commite secrets

## 📝 Exercícios Práticos

### Básico
1. Crie um Dockerfile para sua aplicação
2. Configure docker-compose com app + banco
3. Configure pipeline básico de CI no GitHub Actions
4. Implemente health checks
5. Configure logging com Serilog

### Intermediário
6. Configure pipeline completo de CI/CD
7. Deploy para Azure App Service
8. Configure Application Insights
9. Crie manifests Kubernetes básicos
10. Implemente blue-green deployment

### Avançado
11. Configure Kubernetes com Helm Charts
12. Implemente service mesh (Istio/Linkerd)
13. Configure monitoring com Prometheus + Grafana
14. Implemente GitOps com ArgoCD/Flux
15. Configure auto-scaling

### Projeto
**DevOps Pipeline Completo**: Configure:
- Aplicação containerizada
- Pipeline CI/CD (build, test, deploy)
- Deploy em Kubernetes
- Monitoring e logging centralizados
- Health checks e readiness probes
- Secrets management
- Multiple environments (dev, staging, prod)
- Rollback strategy

## 🔗 Recursos Recomendados

### Documentação
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [GitHub Actions](https://docs.github.com/en/actions)

### Vídeos
- [Docker for .NET Developers](https://www.youtube.com/watch?v=gAGEar5HQoU)
- [Kubernetes Tutorial](https://www.youtube.com/watch?v=X48VuDVv0do)

### Cursos
- [Docker Mastery](https://www.udemy.com/course/docker-mastery/)
- [Kubernetes for Developers](https://www.pluralsight.com/courses/kubernetes-developers-core-concepts)

### Livros
- "Docker Deep Dive" - Nigel Poulton
- "Kubernetes in Action" - Marko Lukša
- "The Phoenix Project" - Gene Kim

## ✅ Checklist de Conclusão

Antes de avançar, certifique-se de que você:

- [ ] Sabe criar Dockerfile para .NET
- [ ] Conhece docker-compose
- [ ] Configurou pipeline de CI/CD
- [ ] Implementou logging estruturado
- [ ] Configurou health checks
- [ ] Conhece basics de Kubernetes
- [ ] Sabe fazer deploy em cloud (Azure/AWS)
- [ ] Implementou monitoring
- [ ] Gerencia secrets adequadamente
- [ ] Completou projeto de pipeline completo

## ⏭️ Próximo Passo

Após dominar DevOps, avance para [10-Arquitetura](../10-Arquitetura/README.md) para padrões arquiteturais.

---

*"DevOps is not a goal, but a never-ending process of continual improvement." - Jez Humble*
