# Setup .NET Core 2.2 Web Api with EF Core

This is a guide on how to create a Web API project with .NET Core 2.2 CLI and Entity Framework Core.

## Setting up the general project structure

Open your terminal and write the following commands

```bash
mkdir MyNewProject
cd MyNewProject
dotnet new sln
git init
touch .gitignore
```

Add files and folders to gitignore (see [my suggestions](suggestedGitignore.txt)). Then add a new project to the solution:

```bash
dotnet new webapi -o MyNewProject.Api
dotnet sln add MyNewProject.Api/MyNewProject.Api.csproj
```

Now commit and push

```bash
git add -Av
git commit -m "Initial commit; add solution and project"
git remote add origin [remote address]
git push -u origin master
```

## Setup EF Core with MySQL

Navigate to the API project and add the EF Core NuGet package with MySql

```bash
cd MyNewProject/
dotnet add pakcage MySql.Data.EntityFrameworkCore
```

Create a model `Data/Models/MyModel.cs`:

```csharp
namespace MyNewProject.Api.Data.Models
{
    public class MyModel
    {
        public string Id { get; set; }
        public string Title { get; set; }
    }
}
```

Create a configuration class for the model `Data/EntityConfigurations/MyModelConfiguration.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using MyNewProject.Api.Data.Models;
namespace MyNewProject.Api.Data.EntityConfigurations
{
    public class MyModelConfiguration : IEntityTypeConfiguration<MyModel>
    {
        public void Configure(EntityTypeBuilder<MyModel> builder)
        {
            builder.HasIndex(m => m.Id);
            builder.Property(m => m.Title).IsRequired();
        }
    }
}
```

Now create the db context `Data/ApplicationDbContext.cs`:

```csharp
using MyNewProject.Data.Models;
using MyNewProject.Data.EntityConfigurations;
using Microsoft.EntityFrameworkCore;
namespace MyNewProject.Data
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options) { }

        public DbSet<Address> Addresses { get; set; }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            builder.ApplyConfiguration(new MyModelConfiguration());
        }
    }
}
```

In `Startup.cs` we can now configure entity framework:

```csharp
[...]
public void ConfigureServices(IServiceCollection services)
{
    var dbHost = Configuration["DB_HOST"] ?? "localhost";
    var dbPort = Configuration["DB_PORT"] ?? "3306";
    var dbName = Configuration["DB_NAME"] ?? "my_database";
    var dbUser = Configuration["DB_USER"] ?? "root";
    var dbPass = Configuration["DB_PASS"] ?? "root";

    var connectionString = $"server={dbHost};port={dbPort};database={dbName};uid={dbUser};password={dbPass}";

    services.AddEntityFrameworkSqlServer().AddDbContext<ApplicationDbContext>(opt => 
        opt.UseMySQL(connectionString)
    );

    [...]
}
[...]
```

Now simply migrate and update the databae:

```bash
dotnet ef migrations add "MyNewMigration"
dotnet ef database update
```
