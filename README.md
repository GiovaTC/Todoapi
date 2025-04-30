# app Todoapi
# 🔧 Tecnologías

- **Backend:** ASP.NET Core Web API (.NET 6 o .NET 8)  
- **Frontend:** Razor Pages o Blazor (alternativamente, HTML/CSS/JS)  
- **Base de datos:** MySQL o SQL Server  
- **IDE:** Visual Studio 2022  

---

# 🎯 Ejemplo de App: Gestor de Tareas (ToDo App)

## Funcionalidades:

- Crear tarea  
- Leer lista de tareas  
- Actualizar tarea  
- Eliminar tarea  

---

# ✅ Paso a Paso

## 1. Crear el proyecto Web API

1. Abre **Visual Studio 2022**.  
2. Crea un nuevo proyecto.  
3. Selecciona **ASP.NET Core Web API**.  
4. Nombre: `TodoApi`  
5. Marca la opción **Enable OpenAPI Support (Swagger)**.  
6. Elige **.NET 6** o **.NET 8**.  

### Crea el modelo `TodoItem.cs`:

```csharp
public class TodoItem
{
    public int Id { get; set; }
    public string Title { get; set; }
    public bool IsDone { get; set; }
}
Agrega el DbContext TodoContext.cs:
csharp
Copiar
Editar
using Microsoft.EntityFrameworkCore;

public class TodoContext : DbContext
{
    public TodoContext(DbContextOptions<TodoContext> options) : base(options) { }

    public DbSet<TodoItem> TodoItems { get; set; }
}
Configura el contexto en Program.cs:
csharp
Copiar
Editar
builder.Services.AddDbContext<TodoContext>(options =>
    options.UseInMemoryDatabase("TodoList")); // o UseMySQL(...) si deseas MySQL
Crea el controlador TodoController.cs:
csharp
Copiar
Editar
[ApiController]
[Route("api/[controller]")]
public class TodoController : ControllerBase
{
    private readonly TodoContext _context;

    public TodoController(TodoContext context)
    {
        _context = context;
    }

    [HttpGet]
    public async Task<ActionResult<IEnumerable<TodoItem>>> Get() =>
        await _context.TodoItems.ToListAsync();

    [HttpPost]
    public async Task<ActionResult<TodoItem>> Post(TodoItem item)
    {
        _context.TodoItems.Add(item);
        await _context.SaveChangesAsync();
        return CreatedAtAction(nameof(Get), new { id = item.Id }, item);
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> Put(int id, TodoItem item)
    {
        if (id != item.Id) return BadRequest();
        _context.Entry(item).State = EntityState.Modified;
        await _context.SaveChangesAsync();
        return NoContent();
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var item = await _context.TodoItems.FindAsync(id);
        if (item == null) return NotFound();

        _context.TodoItems.Remove(item);
        await _context.SaveChangesAsync();
        return NoContent();
    }
}
2. Crear el Frontend (Razor Pages)
En la misma solución, agrega un nuevo proyecto: ASP.NET Core Web App (Razor Pages)

Llámalo TodoApp.Client

Agrega una clase TodoItem.cs en este proyecto también (modelo compartido)

Instala HttpClient
En Program.cs agrega:

csharp
Copiar
Editar
builder.Services.AddHttpClient();
Crea la página Pages/Index.cshtml.cs:
csharp
Copiar
Editar
public class IndexModel : PageModel
{
    private readonly IHttpClientFactory _clientFactory;

    public IndexModel(IHttpClientFactory clientFactory)
    {
        _clientFactory = clientFactory;
    }

    public List<TodoItem> Todos { get; set; } = new();

    public async Task OnGetAsync()
    {
        var client = _clientFactory.CreateClient();
        var response = await client.GetFromJsonAsync<List<TodoItem>>("https://localhost:5001/api/todo");
        if (response != null)
            Todos = response;
    }
}
En Pages/Index.cshtml:
html
Copiar
Editar
@page
@model IndexModel

<h1>Lista de Tareas</h1>

<ul>
@foreach (var todo in Model.Todos)
{
    <li>
        <input type="checkbox" @(todo.IsDone ? "checked" : "") disabled />
        @todo.Title
    </li>
}
</ul>
🧪 Ejecutar la App
Establece el proyecto API como Startup Project y ejecútalo.

Luego, establece el proyecto Razor como Startup Project y ejecútalo.

Asegúrate de que el puerto del backend esté correcto en la URL del frontend.

🛠️ Conectar el proyecto ASP.NET Core Web API a MySQL
1. 📦 Instalar paquetes NuGet necesarios
Abre la consola de NuGet (o usa el administrador de paquetes en Visual Studio) y ejecuta:

bash
Copiar
Editar
Install-Package Pomelo.EntityFrameworkCore.MySql
Este paquete es una de las opciones más populares y bien mantenidas para MySQL en .NET.

2. ⚙️ Configurar la cadena de conexión en appsettings.json
json
Copiar
Editar
{
  "ConnectionStrings": {
    "DefaultConnection": "server=localhost;port=3306;database=todo_db;user=root;password=TU_CONTRASEÑA;"
  }
}
3. 🧠 Modificar Program.cs para usar MySQL
csharp
Copiar
Editar
builder.Services.AddDbContext<TodoContext>(options =>
    options.UseMySql(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        ServerVersion.AutoDetect(builder.Configuration.GetConnectionString("DefaultConnection"))
    ));
4. 🗃️ Crear la base de datos en MySQL (una vez)
Puedes ejecutar esto desde tu cliente MySQL (MySQL Workbench, DBeaver o línea de comandos):

sql
Copiar
Editar
CREATE DATABASE todo_db;
5. 🧱 Crear e instalar la base de datos desde EF Core (Migraciones)
Desde la Consola del Administrador de Paquetes de Visual Studio:

bash
Copiar
Editar
Add-Migration InitialCreate
Update-Database
Esto creará las tablas automáticamente en MySQL.

6. ✅ Verifica que el controlador ya esté funcionando
Ejecuta la API y prueba con Swagger:
https://localhost:5001/swagger
Asegúrate de poder realizar correctamente GET, POST, PUT y DELETE.
Los datos deben guardarse y recuperarse desde MySQL.

------------//------------//----------//-----------//----------//-------
Aviso de Copyright
© 2025 TuNombre o TuEmpresa. Todos los derechos reservados.

Esta aplicación y su contenido están protegidos por las leyes de derechos de autor.
Queda prohibida su reproducción, distribución o modificación total o parcial sin la autorización expresa del titular.

Desarrollado con Visual Studio 2022 utilizando tecnologías .NET.
