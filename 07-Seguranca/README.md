# 🔒 Segurança em .NET

## 🎯 Objetivos de Aprendizado

Ao completar este módulo, você será capaz de:
- Implementar autenticação e autorização
- Trabalhar com JWT (JSON Web Tokens)
- Usar ASP.NET Core Identity
- Implementar OAuth 2.0 e OpenID Connect
- Proteger APIs contra ataques comuns
- Gerenciar secrets e configurações sensíveis

## 📚 Conteúdo

### 1. Conceitos Fundamentais

#### Autenticação vs Autorização
- **Autenticação**: Quem você é? (Login)
- **Autorização**: O que você pode fazer? (Permissões)

#### Princípios de Segurança
- Princípio do Menor Privilégio
- Defesa em Profundidade
- Fail Secure (Falhar com Segurança)
- Não confiar em dados de entrada

### 2. ASP.NET Core Identity

```bash
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
```

#### Configuração
```csharp
// Models/ApplicationUser.cs
public class ApplicationUser : IdentityUser
{
    public string NomeCompleto { get; set; }
    public DateTime DataCadastro { get; set; }
}

// Data/ApplicationDbContext.cs
public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
}

// Program.cs
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString));

builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options =>
{
    // Configurações de senha
    options.Password.RequireDigit = true;
    options.Password.RequireLowercase = true;
    options.Password.RequireUppercase = true;
    options.Password.RequireNonAlphanumeric = true;
    options.Password.RequiredLength = 8;
    
    // Lockout
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(5);
    options.Lockout.MaxFailedAccessAttempts = 5;
    
    // User
    options.User.RequireUniqueEmail = true;
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders();

app.UseAuthentication();
app.UseAuthorization();
```

#### Registro e Login
```csharp
public class AccountController : Controller
{
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;
    
    public AccountController(
        UserManager<ApplicationUser> userManager,
        SignInManager<ApplicationUser> signInManager)
    {
        _userManager = userManager;
        _signInManager = signInManager;
    }
    
    [HttpPost]
    public async Task<IActionResult> Register(RegisterDto model)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
        
        var user = new ApplicationUser
        {
            UserName = model.Email,
            Email = model.Email,
            NomeCompleto = model.NomeCompleto,
            DataCadastro = DateTime.UtcNow
        };
        
        var result = await _userManager.CreateAsync(user, model.Password);
        
        if (result.Succeeded)
        {
            await _signInManager.SignInAsync(user, isPersistent: false);
            return Ok(new { message = "Usuário registrado com sucesso" });
        }
        
        return BadRequest(result.Errors);
    }
    
    [HttpPost]
    public async Task<IActionResult> Login(LoginDto model)
    {
        var result = await _signInManager.PasswordSignInAsync(
            model.Email,
            model.Password,
            model.RememberMe,
            lockoutOnFailure: true);
        
        if (result.Succeeded)
            return Ok(new { message = "Login bem-sucedido" });
        
        if (result.IsLockedOut)
            return BadRequest("Conta bloqueada temporariamente");
        
        return Unauthorized("Credenciais inválidas");
    }
    
    [HttpPost]
    [Authorize]
    public async Task<IActionResult> Logout()
    {
        await _signInManager.SignOutAsync();
        return Ok();
    }
}
```

### 3. JWT (JSON Web Tokens)

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

#### Configuração JWT
```csharp
// appsettings.json
{
  "JwtSettings": {
    "SecretKey": "sua-chave-super-secreta-com-pelo-menos-32-caracteres",
    "Issuer": "https://meusite.com",
    "Audience": "https://meusite.com",
    "ExpirationInMinutes": 60
  }
}

// Program.cs
var jwtSettings = builder.Configuration.GetSection("JwtSettings");

builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true,
        ValidIssuer = jwtSettings["Issuer"],
        ValidAudience = jwtSettings["Audience"],
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(jwtSettings["SecretKey"]))
    };
});
```

#### Gerando Tokens
```csharp
public class TokenService
{
    private readonly IConfiguration _configuration;
    
    public TokenService(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public string GenerateToken(ApplicationUser user, IList<string> roles)
    {
        var jwtSettings = _configuration.GetSection("JwtSettings");
        var secretKey = jwtSettings["SecretKey"];
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(secretKey));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
        
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id),
            new Claim(ClaimTypes.Name, user.UserName),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
        };
        
        // Adicionar roles como claims
        foreach (var role in roles)
        {
            claims.Add(new Claim(ClaimTypes.Role, role));
        }
        
        var token = new JwtSecurityToken(
            issuer: jwtSettings["Issuer"],
            audience: jwtSettings["Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(
                int.Parse(jwtSettings["ExpirationInMinutes"])),
            signingCredentials: credentials
        );
        
        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

### 4. Autorização

#### Role-Based Authorization
```csharp
[Authorize(Roles = "Admin")]
public class AdminController : Controller
{
    public IActionResult Dashboard() => View();
}

[Authorize(Roles = "Admin,Manager")]
public IActionResult Reports() => View();
```

#### Policy-Based Authorization
```csharp
// Program.cs
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("RequireAdminRole", policy =>
        policy.RequireRole("Admin"));
    
    options.AddPolicy("RequireAge18", policy =>
        policy.RequireClaim("Age", "18", "19", "20" /* etc */));
    
    options.AddPolicy("RequirePremiumUser", policy =>
        policy.RequireAssertion(context =>
            context.User.HasClaim(c => c.Type == "SubscriptionType" && 
                                      c.Value == "Premium")));
});

// Controller
[Authorize(Policy = "RequireAdminRole")]
public class AdminController : Controller { }

[Authorize(Policy = "RequirePremiumUser")]
public IActionResult PremiumContent() => View();
```

#### Custom Authorization Handler
```csharp
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    
    public MinimumAgeRequirement(int minimumAge)
    {
        MinimumAge = minimumAge;
    }
}

public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        var dateOfBirthClaim = context.User.FindFirst(
            c => c.Type == ClaimTypes.DateOfBirth);
        
        if (dateOfBirthClaim == null)
            return Task.CompletedTask;
        
        var dateOfBirth = Convert.ToDateTime(dateOfBirthClaim.Value);
        var age = DateTime.Today.Year - dateOfBirth.Year;
        
        if (age >= requirement.MinimumAge)
            context.Succeed(requirement);
        
        return Task.CompletedTask;
    }
}

// Registro
builder.Services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AtLeast18", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(18)));
});
```

### 5. OAuth 2.0 / OpenID Connect

```bash
dotnet add package Microsoft.AspNetCore.Authentication.Google
dotnet add package Microsoft.AspNetCore.Authentication.Facebook
```

```csharp
builder.Services.AddAuthentication()
    .AddGoogle(options =>
    {
        options.ClientId = configuration["Google:ClientId"];
        options.ClientSecret = configuration["Google:ClientSecret"];
    })
    .AddFacebook(options =>
    {
        options.AppId = configuration["Facebook:AppId"];
        options.AppSecret = configuration["Facebook:AppSecret"];
    });
```

### 6. Proteção Contra Ataques

#### CSRF (Cross-Site Request Forgery)
```csharp
// Automático em ASP.NET Core para formulários
builder.Services.AddAntiforgery(options =>
{
    options.HeaderName = "X-CSRF-TOKEN";
});
```

#### SQL Injection
```csharp
// ❌ ERRADO - Vulnerável a SQL Injection
var query = $"SELECT * FROM Users WHERE Username = '{username}'";

// ✅ CORRETO - Usar parametrização (EF Core faz isso automaticamente)
var user = await _context.Users
    .FirstOrDefaultAsync(u => u.Username == username);

// ✅ CORRETO - Query parametrizada
var user = await _context.Users
    .FromSqlRaw("SELECT * FROM Users WHERE Username = {0}", username)
    .FirstOrDefaultAsync();
```

#### XSS (Cross-Site Scripting)
```razor
@* Razor automaticamente codifica HTML *@
<div>@Model.UserInput</div>

@* Para HTML não codificado (USE COM CUIDADO) *@
<div>@Html.Raw(Model.TrustedHtml)</div>
```

#### HTTPS e HSTS
```csharp
// Program.cs
app.UseHttpsRedirection();
app.UseHsts();

builder.Services.AddHsts(options =>
{
    options.MaxAge = TimeSpan.FromDays(365);
    options.IncludeSubDomains = true;
});
```

### 7. Gerenciamento de Secrets

#### User Secrets (Desenvolvimento)
```bash
dotnet user-secrets init
dotnet user-secrets set "JwtSettings:SecretKey" "minha-chave-secreta"
```

#### Azure Key Vault (Produção)
```bash
dotnet add package Azure.Extensions.AspNetCore.Configuration.Secrets
```

```csharp
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{keyVaultName}.vault.azure.net/"),
    new DefaultAzureCredential());
```

#### Environment Variables
```csharp
var secretKey = Environment.GetEnvironmentVariable("JWT_SECRET_KEY");
```

### 8. Data Protection

```csharp
builder.Services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@"./keys"))
    .SetApplicationName("MinhaApp");

public class SecureService
{
    private readonly IDataProtector _protector;
    
    public SecureService(IDataProtectionProvider provider)
    {
        _protector = provider.CreateProtector("MinhaApp.SecureService");
    }
    
    public string Encrypt(string plainText)
    {
        return _protector.Protect(plainText);
    }
    
    public string Decrypt(string cipherText)
    {
        return _protector.Unprotect(cipherText);
    }
}
```

## 💡 Dicas de Estudo

1. **Nunca Armazene Senhas em Texto Plano**: Sempre use hashing
2. **Use HTTPS**: Sempre em produção
3. **Valide no Servidor**: Nunca confie apenas em validação client-side
4. **Principle of Least Privilege**: Dê apenas permissões necessárias
5. **Keep Dependencies Updated**: Vulnerabilidades são descobertas constantemente

## 📝 Exercícios Práticos

### Básico
1. Configure ASP.NET Core Identity
2. Implemente registro e login de usuários
3. Crie roles e atribua a usuários
4. Proteja rotas com [Authorize]

### Intermediário
5. Implemente autenticação JWT
6. Crie políticas de autorização customizadas
7. Configure login social (Google, Facebook)
8. Implemente refresh tokens
9. Configure CORS e HTTPS

### Avançado
10. Crie authorization handler customizado
11. Implemente Two-Factor Authentication (2FA)
12. Configure Azure Key Vault
13. Implemente rate limiting por usuário
14. Adicione auditoria de acessos

### Projeto
**Sistema de Autenticação Completo**: Desenvolva:
- Registro com validação de e-mail
- Login com JWT
- Roles e permissões
- Login social (Google/Facebook)
- Recuperação de senha
- Two-Factor Authentication
- API protegida com diferentes níveis de acesso
- Auditoria de login e acessos

## 🔗 Recursos Recomendados

### Documentação
- [ASP.NET Core Security](https://learn.microsoft.com/aspnet/core/security/)
- [Identity Documentation](https://learn.microsoft.com/aspnet/core/security/authentication/identity)
- [JWT Authentication](https://jwt.io/)

### Vídeos
- [ASP.NET Core Identity Tutorial](https://www.youtube.com/watch?v=egITMrwMOPU)
- [JWT Authentication in ASP.NET Core](https://www.youtube.com/watch?v=mgeuh8k3I4g)

### Artigos
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Security Best Practices](https://learn.microsoft.com/aspnet/core/security/app-secrets)

### Livros
- "ASP.NET Core Security" - Christian Wenz
- "Web Security Testing Cookbook" - Paco Hope

## ✅ Checklist de Conclusão

Antes de avançar, certifique-se de que você:

- [ ] Entende autenticação vs autorização
- [ ] Configurou ASP.NET Core Identity
- [ ] Implementou JWT authentication
- [ ] Conhece role-based e policy-based authorization
- [ ] Sabe proteger contra ataques comuns (XSS, CSRF, SQL Injection)
- [ ] Configurou HTTPS e HSTS
- [ ] Sabe gerenciar secrets adequadamente
- [ ] Implementou login social
- [ ] Conhece data protection
- [ ] Completou projeto de autenticação completo

## ⏭️ Próximo Passo

Após dominar segurança, avance para [08-Performance](../08-Performance/README.md) para otimização.

---

*"Security is not a product, but a process." - Bruce Schneier*
