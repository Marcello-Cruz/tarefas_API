# tarefas-cs-web
Uma aplicação em C# (web) com Minimal APIs, EFCore 6, MySQL e Pomelo

## OpenAPI (Swagger)
Pacote:
```
dotnet add package Swashbuckle.AspNetCore
```

Código:
```cs
builder.Services.AddSwaggerGen();
builder.Services.AddEndpointsApiExplorer();
...
app.UseSwagger();
app.UseSwaggerUI();
```

## _Arquivos estáticos_
Código:
```cs
app.UseDefaultFiles();
app.UseStaticFiles();
```

## EntityFramework Core 6 (MySQL com Pomelo)
Banco: https://github.com/ermogenes/tarefas-mysql

Para subir o MySQL com Docker:
```
docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1234 mysql:8.0.28
```

Foi utilizada a lib [Pomelo](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql) (`Pomelo.EntityFrameworkCore.MySql`) em vez do Connector/NET oficial da Oracle, devido ao suporte simplificado a diferentes versões do MySQL.

Comandos utilizados para fazer o _scaffolding_:

```
dotnet tool install --global dotnet-ef
dotnet tool update --global dotnet-ef

dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Pomelo.EntityFrameworkCore.MySql

dotnet ef dbcontext scaffold "server=localhost;port=3306;uid=root;pwd=1234;database=tarefas" Pomelo.EntityFrameworkCore.MySql -o db -f --no-pluralize
```

String de conexão transferida de `tarefasContext.OnConfiguring` para `appsettings.json`:

```json
"ConnectionStrings": {
    "tarefasConnection": "server=localhost;port=3306;uid=root;pwd=1234;database=tarefas"
}
```

Exemplo:
```cs
...
builder.Services.AddDbContext<tarefasContext>(opt =>
{
    string connectionString = builder.Configuration.GetConnectionString("tarefasConnection");
    var serverVersion = ServerVersion.AutoDetect(connectionString);
    opt.UseMySql(connectionString, serverVersion);
});
...
app.MapGet("/api/tarefas", ([FromServices] tarefasContext _db) =>
{
    return Results.Ok(_db.Tarefa.ToList<Tarefa>());
});
...
```
