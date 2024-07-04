# EFCoreDataWorkshop

## Herramienta que utilizaremos

**Entity Framework Core tools reference - .NET Core CLI**

https://learn.microsoft.com/en-us/ef/core/cli/dotnet

Comando para instalación

```csharp
dotnet tool install --global dotnet-ef
```

Verificación de instalación

```csharp
$ dotnet-ef --version
Entity Framework Core .NET Command-line Tools
8.0.6
```

## Creación del proyecto

```csharp
$ dotnet new webapi -n IntroduccionEFCore
$ cd IntroduccionEFCore
$ code .
```

Necesitaremos dos paquetes

- Instalar la extensión NuGet Package Managuer GUI
  VS Marketplace Link: https://marketplace.visualstudio.com/items?itemName=aliasadidev.nugetpackagemanagergui - Después ingresar a la paleta de comandos `Ctrl+Shift+P` escribir la opción **`NuGet Package Manager GUI`** - Elegir la opción `Install New Package` - Buscar **`Microsoft.EntityFrameworkCore.SqlServer`** hacer click en Install - Buscar [\*\*`Microsoft.EntityFrameworkCore.Design`](http://Microsoft.EntityFrameworkCore.Design)\*\* hacer click en Install

## Creación de entidad

Creación de clase `ApplicationDbContext` heredando de `DbContext`

1.- Creamos un archivo llamado `ApplicationDbContext.cs`

```csharp
using Microsoft.EntityFrameworkCore;

namespace IntroduccionEFCore
{
  public class ApplicationDbContext : DbContext
  {
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
  }
}
```

2.- Hacemos la configuración en `Program.cs`

```csharp
using IntroduccionEFCore;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddDbContext<ApplicationDbContext>(opciones => opciones.UseSqlServer("name=DefaultConnection"));
```

3.- Desarrollamos nuestra conexión a la base de datos en `appsettings.Development.json`

```csharp
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=PAZTEDDY;Database=IntroduccionEFCore;Integrated Security=True;TrustServerCertificate=True;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}

```

4.- Creamos nuestra primera entidad `Genero.cs` en un nuevo directorio `Entidades` la ruta sería la siguiente `Entidades\Genero.cs`

```csharp
namespace IntroduccionEFCore.Entidades
{
  public class Genero
  {
    public int Id { get; set; }
    public string? Nombre { get; set; }
  }
}

```

5.- Para que realmente sea una entidad tenemos que configurarla en `ApplicationDbContext.cs`

```csharp
using IntroduccionEFCore.Entidades;
using Microsoft.EntityFrameworkCore;

namespace IntroduccionEFCore
{
  public class ApplicationDbContext : DbContext
  {
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    public DbSet<Genero> Generos => Set<Genero>();
  }
}
```

6.- Crearemos una migración para poder afectar a nuestra base de datos

- Para poder crear las migraciones tendremos que correr el siguiente comando este deberá correr desde la carpeta donde se tiene el archivo `.csproj` (Esto porque estamos usando EF Core CLI)
  ```csharp
  dotnet ef migrations add Inicial
  ```
  Nos creará un directorio `Migrations` y una clase con el nombre que mandamos como parámetro `Migrations\20240701022852_Inicial.cs`
- Ahora aplicaremos la migración al proyecto

      ```csharp
      dotnet ef database update
      ```

      ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7f819dcc-d011-4d16-9542-07e9ca77e445/f6b59605-b7af-4b88-97ef-0f9ecbce6cf5/Untitled.png)

  7.- Migraciones con valores por defecto

- Para que tengamos datos de prueba podemos hacer lo siguiente en `ApplicationDbContext.cs`

  ```csharp
  using IntroduccionEFCore.Entidades;
  using Microsoft.EntityFrameworkCore;

  namespace IntroduccionEFCore
  {
    public class ApplicationDbContext : DbContext
    {
      public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
          : base(options)
      {
      }
      protected override void OnModelCreating(ModelBuilder modelBuilder)
      {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<Genero>().HasData(new Genero { Id = 1, Nombre = "Acción" }, new Genero { Id = 2, Nombre = "Ciencia Ficción" });
      }

      public DbSet<Genero> Generos => Set<Genero>();
    }
  }
  ```

- Para que se ejecute el cambio creamos una nueva migración

      ```csharp
      dotnet ef migrations add AgregarDatosGenero
      ```

      Luego el comando:

      ```csharp
      dotnet ef database update
      ```

  8.- Queremos validar nuestros campos lo haremos en el modelo a través de las anotaciones de datos `DataAnnotations` en `Entidades\Genero.cs`

```csharp
using System.ComponentModel.DataAnnotations;

namespace IntroduccionEFCore.Entidades
{
  public class Genero
  {
    public int Id { get; set; }
    [StringLength(maximumLength: 150)]
    public string? Nombre { get; set; }
  }
}

```

- Para que se ejecute el cambio creamos una nueva migración
  ```csharp
  dotnet ef migrations add GeneroNombre
  ```
- Actualizamos nuestra base de datos

      ```csharp
      dotnet ef database update
      ```

  9.- Si queremos revertir la ultima migración podemos usar la siguiente sentencia `dotnet ef database update <previous-migration-name>` en nuestro caso:

```csharp
dotnet ef database update AgregarDatosGenero
```

## Creación EndPoint y operaciones CRUD

1.- Una vez creada nuestra entidad y tabla en la base de datos podemos exponer nuestro endpoint `api/genero`

- Tenemos que configurar nuestro `Program.cs` (Borramos el ejemplo `weatherforecast` )

```csharp
using IntroduccionEFCore;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
// Añade esta línea para registrar los controladores
builder.Services.AddControllers();

builder.Services.AddDbContext<ApplicationDbContext>(opciones => opciones.UseSqlServer("name=DefaultConnection"));

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
  app.UseSwagger();
  app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API v1"));
}
app.UseRouting();
app.UseHttpsRedirection();
app.MapControllers();
app.Run();

```

- Creamos el directorio para controladores y también el controlador `Controllers\GeneroController.cs` con las operaciones CRUD

```csharp
using Microsoft.AspNetCore.Mvc;
using IntroduccionEFCore.Entidades;
using Microsoft.EntityFrameworkCore;

namespace IntroduccionEFCore.Controllers
{
  [ApiController]
  [Route("/api/[controller]")]
  public class GeneroController : ControllerBase
  {
    private readonly ApplicationDbContext _context;

    public GeneroController(ApplicationDbContext context)
    {
      _context = context;
    }

    // GET: api/Genero
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Genero>>> GetGeneros()
    {
      return await _context.Generos.ToListAsync();
    }

    // GET: api/Genero/5
    [HttpGet("{id}")]
    public async Task<ActionResult<Genero>> GetGenero(int id)
    {
      var genero = await _context.Generos.FindAsync(id);

      if (genero == null)
      {
        return NotFound();
      }

      return genero;
    }

    // PUT: api/Genero/5
    [HttpPut("{id}")]
    public async Task<IActionResult> PutGenero(int id, Genero generoPartial)
    {
      var genero = await _context.Generos.FindAsync(id);
      if (genero == null)
      {
        return NotFound();
      }

      // Actualizar solo los campos que fueron enviados en la solicitud
      genero.Nombre = generoPartial.Nombre; // Asumiendo que solo el nombre puede ser actualizado

      try
      {
        await _context.SaveChangesAsync();
      }
      catch (DbUpdateConcurrencyException)
      {
        if (!GeneroExists(id))
        {
          return NotFound();
        }
        else
        {
          throw;
        }
      }

      return NoContent();
    }

    // POST: api/Genero
    [HttpPost]
    public async Task<ActionResult<Genero>> PostGenero(Genero genero)
    {
      _context.Generos.Add(genero);
      await _context.SaveChangesAsync();

      return CreatedAtAction(nameof(GetGenero), new { id = genero.Id }, genero);
    }

    // DELETE: api/Genero/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteGenero(int id)
    {
      var genero = await _context.Generos.FindAsync(id);
      if (genero == null)
      {
        return NotFound();
      }

      _context.Generos.Remove(genero);
      await _context.SaveChangesAsync();

      return NoContent();
    }

    private bool GeneroExists(int id)
    {
      return _context.Generos.Any(e => e.Id == id);
    }
  }
}
```

- Iniciamos el servidor

      ```csharp
      dotnet run
      ```

      Comprobamos el puerto en el que se inicia nuestro servidor  y probamos las siguientes rutas:

      - http://localhost:[puerto]/api/genero
      - http://localhost:[puerto]/swagger/index.html

  2.- (Opcional) Al revisar los Endpoints se ve que tenemos que nos pide enviar el “id” en el objeto JSON cuando tratamos de hacer un `POST` creación de un genero, para poder modificar la generación con ese campo tendremos que hacer lo siguiente:

- Crear un archivo `Entidades\GeneroPostModel.cs` donde solo se especificará la propiedad `Nombre`
  ```csharp
  public class GeneroPostModel
  {
    public string? Nombre { get; set; }
  }
  ```
- Modificamos nuestro `POST` en `Controllers\GeneroController.cs`

  ```csharp
  // POST: api/Genero
  [HttpPost]
  public async Task<ActionResult<Genero>> PostGenero(GeneroPostModel generoPostModel)
   {
    var genero = new Genero { Nombre = generoPostModel.Nombre };
    _context.Generos.Add(genero);
    await _context.SaveChangesAsync();

    return CreatedAtAction(nameof(GetGenero), new { id = genero.Id }, genero);
  }
  ```

Probamos la funcionalidad `http://localhost:[puerto]/swagger/index.html`

## Creación de entidad Actor

- Crearemos una nueva clase `Entidades/Actor.cs`

```csharp
namespace IntroduccionEFCore.Entidades
{
  public class Actor
  {
    public int Id { get; set; }
    public required string Nombre { get; set; }
    public DateTime? FechaNacimiento { get; set; }
  }
}
```

- Validación y tipo de dato con **Fluent API** configuración en **`ApplicationDbContext.cs`**

```csharp
using IntroduccionEFCore.Entidades;
using Microsoft.EntityFrameworkCore;

namespace IntroduccionEFCore
{
  public class ApplicationDbContext : DbContext
  {
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
      base.OnModelCreating(modelBuilder);

      modelBuilder.Entity<Genero>().HasData(new Genero { Id = 1, Nombre = "Acción" }, new Genero { Id = 2, Nombre = "Ciencia Ficción" });

      modelBuilder.Entity<Actor>().Property(a => a.Nombre).IsRequired();
      modelBuilder.Entity<Actor>().Property(a => a.Nombre).HasMaxLength(150);
      modelBuilder.Entity<Actor>().Property(a => a.FechaNacimiento).HasColumnType("date");
      modelBuilder.Entity<Actor>().HasData(new Actor { Id = 1, Nombre = "John Doe", FechaNacimiento = new DateTime(1980, 1, 1) }, new Actor { Id = 2, Nombre = "Jane Doe", FechaNacimiento = new DateTime(1990, 2, 2) });

    }

    public DbSet<Genero> Generos => Set<Genero>();
    public DbSet<Actor> Actores => Set<Actor>();
  }
}
```

## Creación de entidad Pelicula

- Crearemos una nueva clase `Entidades/Pelicula.cs`

```csharp
namespace IntroduccionEFCore.Entidades
{
  public class Pelicula
  {
    public int Id { get; set; }
    public required string Titulo { get; set; }
    public DateOnly FechaEstreno { get; set; }

  }
}
```

- Validación **Fluent API** configuración en **`ApplicationDbContext.cs`**

```csharp
using IntroduccionEFCore.Entidades;
using Microsoft.EntityFrameworkCore;

namespace IntroduccionEFCore
{
  public class ApplicationDbContext : DbContext
  {
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
      base.OnModelCreating(modelBuilder);

      modelBuilder.Entity<Genero>().HasData(new Genero { Id = 1, Nombre = "Acción" }, new Genero { Id = 2, Nombre = "Ciencia Ficción" });

      modelBuilder.Entity<Actor>().Property(a => a.Nombre).IsRequired();
      modelBuilder.Entity<Actor>().Property(a => a.Nombre).HasMaxLength(150);
      modelBuilder.Entity<Actor>().Property(a => a.FechaNacimiento).HasColumnType("date");
      modelBuilder.Entity<Actor>().HasData(new Actor { Id = 1, Nombre = "John Doe", FechaNacimiento = new DateTime(1980, 1, 1) }, new Actor { Id = 2, Nombre = "Jane Doe", FechaNacimiento = new DateTime(1990, 2, 2) });

      modelBuilder.Entity<Pelicula>().Property(p => p.Titulo).IsRequired();
      modelBuilder.Entity<Pelicula>().Property(p => p.Titulo).HasMaxLength(150);
    }

    public DbSet<Genero> Generos => Set<Genero>();
    public DbSet<Actor> Actores => Set<Actor>();
    public DbSet<Pelicula> Peliculas => Set<Pelicula>();
  }
}
```

## Creación de entidad Comentario

- Crearemos una nueva clase `Entidades/Comentario.cs`

```csharp
namespace IntroduccionEFCore.Entidades
{
  public class Comentario
  {
    public int Id { get; set; }
    public required string Contenido { get; set; }
  }
}
```

- Creación de la entidad en **`ApplicationDbContext.cs`**

```csharp
using IntroduccionEFCore.Entidades;
using Microsoft.EntityFrameworkCore;

namespace IntroduccionEFCore
{
  public class ApplicationDbContext : DbContext
  {
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
      base.OnModelCreating(modelBuilder);

      modelBuilder.Entity<Genero>().HasData(new Genero { Id = 1, Nombre = "Acción" }, new Genero { Id = 2, Nombre = "Ciencia Ficción" });

      modelBuilder.Entity<Actor>().Property(a => a.Nombre).IsRequired();
      modelBuilder.Entity<Actor>().Property(a => a.Nombre).HasMaxLength(150);
      modelBuilder.Entity<Actor>().Property(a => a.FechaNacimiento).HasColumnType("date");
      modelBuilder.Entity<Actor>().HasData(new Actor { Id = 1, Nombre = "John Doe", FechaNacimiento = new DateTime(1980, 1, 1) }, new Actor { Id = 2, Nombre = "Jane Doe", FechaNacimiento = new DateTime(1990, 2, 2) });

      modelBuilder.Entity<Pelicula>().Property(p => p.Titulo).IsRequired();
      modelBuilder.Entity<Pelicula>().Property(p => p.Titulo).HasMaxLength(150);
    }

    public DbSet<Genero> Generos => Set<Genero>();
    public DbSet<Actor> Actores => Set<Actor>();
    public DbSet<Pelicula> Peliculas => Set<Pelicula>();
    public DbSet<Comentario> Comentarios => Set<Comentario>();
  }
}
```

## Migración

- Ejecutamos en la terminal para crear la migración con las nuevas tablas

```csharp
dotnet ef migrations add EntidadesActorComentarioPelicula
```

- Ejecutamos los cambios de la migración anterior en la base de datos

```csharp
dotnet ef database update
```

## Ordenemos nuestras configuraciones del método OnModelCreating

- En el archivo `ApplicationDbContext.cs` tenemos mezcladas configuraciones de varias entidades, con el paso del tiempo podría ser inmanejable
- Por lo tanto creamos un nuevo directorio en `Entidades\Configuraciones`

  - Creamos la configuraciones de Actor con el nombre de archivo `Entidades\Configuraciones\ActorConfig.cs`

    ```csharp
    using Microsoft.EntityFrameworkCore;
    using Microsoft.EntityFrameworkCore.Metadata.Builders;

    namespace IntroduccionEFCore.Entidades.Configuraciones
    {
      public class ActorConfig : IEntityTypeConfiguration<Actor>
      {
        public void Configure(EntityTypeBuilder<Actor> builder)
        {
          builder.Property(a => a.Nombre).IsRequired();
          builder.Property(a => a.Nombre).HasMaxLength(150);
          builder.Property(a => a.FechaNacimiento).HasColumnType("date");
          builder.HasData(new Actor { Id = 1, Nombre = "John Doe", FechaNacimiento = new DateTime(1980, 1, 1) }, new Actor { Id = 2, Nombre = "Jane Doe", FechaNacimiento = new DateTime(1990, 2, 2) });
        }
      }
    }
    ```

  - Creamos la configuraciones de Pelicula con el nombre de archivo `Entidades\Configuraciones\PeliculaConfig.cs`

    ```csharp
    using Microsoft.EntityFrameworkCore;
    using Microsoft.EntityFrameworkCore.Metadata.Builders;

    namespace IntroduccionEFCore.Entidades.Configuraciones
    {
      public class PeliculaConfig : IEntityTypeConfiguration<Pelicula>
      {
        public void Configure(EntityTypeBuilder<Pelicula> builder)
        {
          builder.Property(p => p.Titulo).IsRequired();
          builder.Property(p => p.Titulo).HasMaxLength(150);
        }
      }
    }
    ```

  - Creamos la configuraciones de Genero con el nombre de archivo
    `Entidades\Configuraciones\GeneroCofig.cs`

    ```csharp
    using Microsoft.EntityFrameworkCore;
    using Microsoft.EntityFrameworkCore.Metadata.Builders;

    namespace IntroduccionEFCore.Entidades.Configuraciones
    {
      public class GeneroCofig : IEntityTypeConfiguration<Genero>
      {
        public void Configure(EntityTypeBuilder<Genero> builder)
        {
          builder.HasData(new Genero { Id = 1, Nombre = "Acción" }, new Genero { Id = 2, Nombre = "Ciencia Ficción" });
        }
      }
    }
    ```

- Una vez creados todos los archivos de configuración tenemos que aplicar esas configuraciones `ApplicationDbContext.cs`

```csharp
using System.Reflection;
using IntroduccionEFCore.Entidades;
using Microsoft.EntityFrameworkCore;

namespace IntroduccionEFCore
{
  public class ApplicationDbContext : DbContext
  {
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
      base.OnModelCreating(modelBuilder);
      // Buscamos todas las clases que hereden de IEntityTypeConfiguration<T> y las aplicamos a la base de datos
      modelBuilder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());
    }

    public DbSet<Genero> Generos => Set<Genero>();
    public DbSet<Actor> Actores => Set<Actor>();
    public DbSet<Pelicula> Peliculas => Set<Pelicula>();
    public DbSet<Comentario> Comentarios => Set<Comentario>();
  }
}
```

## Configuración de las relaciones entre los modelos

### Relación uno a mucho entre Pelicula y Comentarios

- En la clase Comentario `Entidades\Comentario.cs`

  ```csharp
  namespace IntroduccionEFCore.Entidades
  {
    public class Comentario
    {
      public int Id { get; set; }
      public required string Contenido { get; set; }

      // Navegación a otras entidades por convención
      // A que Película pertenece el comentario
      public int PeliculaId { get; set; }
      // Propieda de  Navegación desde Comentario a Película
      public required Pelicula Pelicula { get; set; }
    }
  }
  ```

- En la clase Pelicula `Entidades\Pelicula.cs`

```csharp
namespace IntroduccionEFCore.Entidades
{
  public class Pelicula
  {
    public int Id { get; set; }
    public required string Titulo { get; set; }

    public DateOnly FechaEstreno { get; set; }

    // Navegación a Comentarios
    public HashSet<Comentario> Comentarios { get; set; } = new HashSet<Comentario>();

  }
}
```

- Realizamos la migración

```powershell
dotnet ef migrations add RelacionPeliculaComentarios
```

### Relación muchos a muchos entre Peliculas y Generos

- En la clase Pelicula `Entidades\Pelicula.cs`

  ```csharp
  namespace IntroduccionEFCore.Entidades
  {
    public class Pelicula
    {
      public int Id { get; set; }
      public required string Titulo { get; set; }

      public DateOnly FechaEstreno { get; set; }

      // Navegación a Comentarios
      public HashSet<Comentario> Comentarios { get; set; } = new HashSet<Comentario>();

      // Navegación a Generos
      public HashSet<Genero> Generos { get; set; } = new HashSet<Genero>();
    }
  }
  ```

- En la clase Genero `Entidades\Genero.cs`

  ```csharp
  using System.ComponentModel.DataAnnotations;

  namespace IntroduccionEFCore.Entidades
  {
    public class Genero
    {
      public int Id { get; set; }
      [StringLength(maximumLength: 150)]
      public string? Nombre { get; set; }

      // Navegación a Peliculas
      public HashSet<Pelicula> Peliculas { get; set; } = new HashSet<Pelicula>();
    }
  }

  ```

- Creamos la migración
  ```powershell
  dotnet ef migrations add RelacionPeliculasGeneros
  ```

> ⚠️ En caso de que no necesitemos añadir mas campos en la tabla intermedia, podemos hacer los pasos anteriores

### Relación muchos a muchos entre Peliculas y Actores con entidad intermedia

- Crearemos una nueva clase `Entidades\PeliculaActor.cs`

  ```powershell

  ```
