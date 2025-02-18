# JWT Authentication in a .NET MVC App with Entity Framework

This guide explains how to generate, add, and use a JWT (JSON Web Token) for securing a .NET MVC API using Entity Framework (EF) Core.

## Prerequisites

- .NET SDK installed
- ASP.NET Core MVC project setup
- `Microsoft.AspNetCore.Authentication.JwtBearer` package installed
- `Microsoft.EntityFrameworkCore` and `Microsoft.EntityFrameworkCore.SqlServer` installed

## Step 1: Install Dependencies

Run the following commands in the terminal:

```sh
 dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
 dotnet add package Microsoft.EntityFrameworkCore
 dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

## Step 2: Configure Entity Framework and JWT Authentication in `Program.cs`

Modify `Program.cs` to include EF Core and JWT authentication:

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.EntityFrameworkCore;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

// Configure Entity Framework Core
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Configure JWT Authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"])),
            ValidateIssuer = false,
            ValidateAudience = false
        };
    });

builder.Services.AddControllersWithViews();
var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

## Step 3: Create User Model and DbContext

Create a `Models` folder and add `User.cs`:

```csharp
namespace MyApi.Models
{
    public class User
    {
        public int Id { get; set; }
        public string Username { get; set; }
        public string PasswordHash { get; set; }
    }
}
```

Create `AppDbContext.cs`:

```csharp
using Microsoft.EntityFrameworkCore;

namespace MyApi.Models
{
    public class AppDbContext : DbContext
    {
        public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

        public DbSet<User> Users { get; set; }
    }
}
```

## Step 4: Generate a JWT Token in `AuthController.cs`

Create a controller named `AuthController.cs` and add the following code:

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.IdentityModel.Tokens;
using MyApi.Models;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

[Route("api/auth")]
[ApiController]
public class AuthController : ControllerBase
{
    private readonly IConfiguration _config;
    private readonly AppDbContext _context;

    public AuthController(IConfiguration config, AppDbContext context)
    {
        _config = config;
        _context = context;
    }

    [HttpPost("login")]
    public IActionResult Login([FromBody] User user)
    {
        var existingUser = _context.Users.FirstOrDefault(u => u.Username == user.Username);
        if (existingUser == null || existingUser.PasswordHash != user.PasswordHash)
        {
            return Unauthorized();
        }

        var token = GenerateToken(existingUser);
        return Ok(new { token });
    }

    private string GenerateToken(User user)
    {
        var securityKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:Key"]));
        var credentials = new SigningCredentials(securityKey, SecurityAlgorithms.HmacSha256);
        var claims = new[] { new Claim(ClaimTypes.Name, user.Username) };
        var token = new JwtSecurityToken(
            claims: claims,
            expires: DateTime.UtcNow.AddHours(1),
            signingCredentials: credentials
        );
        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

## Step 5: Secure API Endpoints

In a protected controller, add authentication and authorization:

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

[Route("api/protected")]
[ApiController]
[Authorize]
public class ProtectedController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new { message = "You have accessed a protected route!" });
    }
}
```

## Step 6: Apply Migrations and Seed Database

1. **Add Migrations:**
   ```sh
   dotnet ef migrations add InitialCreate
   ```

2. **Apply Migrations:**
   ```sh
   dotnet ef database update
   ```

3. **Seed Users (Optional):** Modify `Program.cs` to seed a test user:
   ```csharp
   using (var scope = app.Services.CreateScope())
   {
       var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();
       if (!context.Users.Any())
       {
           context.Users.Add(new User { Username = "testUser", PasswordHash = "password" });
           context.SaveChanges();
       }
   }
   ```

## Step 7: Testing the API

1. **Start the server:**
   ```sh
   dotnet run
   ```

2. **Get a Token:** Send a POST request to `/api/auth/login` with JSON body:
   ```json
   {
       "username": "testUser",
       "password": "password"
   }
   ```

3. **Access Protected Route:**
   Use the token received in step 2 and send a GET request to `/api/protected` with the header:
   ```sh
   Authorization: Bearer <your-token>
   ```

