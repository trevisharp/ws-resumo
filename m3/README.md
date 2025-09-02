| Artefato                 | Valor |
|--------------------------|-------|
| Entregas + API           | 0.8   |
| Telas                    | 1.2   |
| Login                    | 2.0   |

```cs
using Microsoft.AspNetCore.Connections;
using Microsoft.Data.SqlClient;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.UseAuthorization();

app.MapControllers();

app.MapGet("/auth", (string login, string senha) => 
{
    var db = new DB();
    var consulta = db.Usuario
        .FirstOrDefault(u => u.Login == login && u.SenhaHash == senha);

    if (consulta == null)
        return Results.NotFound();

    var medico = db.Profissional.FirstOrDefault(u => u.Id == consulta.Id);

    if (medico != null)
    {
        var profissional = db.Profissional.FirstOrDefault(u => u.Id == consulta.Id);
        var especialidade = db.Especialidade.FirstOrDefault(u => u.Id == profissional.EspecialidadeId);
        return Results.Ok(new 
        {
            Id = profissional.Id,
            Nome = db.Pessoa.Find(consulta.Id).Nome,
            DataAdesao = profissional.DataAdesao,
            CRM = profissional.CRM,
            Especialidade = especialidade.Nome
        });
    }
    else 
    {
        var carteirinhaPaciente = db.Carteirinha.FirstOrDefault(u => u.Id == consulta.Id);


        return Results.Ok(new
        {
            Id = consulta.Id,
            Convenio = db.Convenios.Find(carteirinhaPaciente.ConvenioId).Nome,
            Nome = db.Pessoa.Find(consulta.Id).Nome,
            CPF = db.Cliente.Find(consulta.Id).CPF,
            DataNascimento = db.Cliente.Find(consulta.Id).DataNascimento
            Validade = carteirinhaPaciente.Validade
        });
    }

});

app.Run();

public class DB : DbContext
{
    public DbSet<Usuario> Usuario { get; set; }
    public DbSet<Pessoa> Pessoa { get; set; }
    public DbSet<Cliente> Cliente { get; set; }
    public DbSet<Profissional> Profissional { get; set; }
    public DbSet<Atendimento> Atendimento  { get; set; }
    public DbSet<Carteirinha> Carteirinha   { get; set; }
    public DbSet<Especialidade> Especialidade { get; set; }
    public DbSet<Convenios> Convenios { get; set; } 


    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        var con = new SqlConnectionStringBuilder
        {
            DataSource = "192.168.9.254,1437",
            InitialCatalog = "NomeDoBanco",
            UserID = "sa",
            Password = "Competidor5@Z627#t4",
            IntegratedSecurity = false
        };

        optionsBuilder.UseSqlServer(con.ToString());
    }
}

public class Convenios
{
    public int Id { get; set; }
    public string Nome { get; set; }
}

public class Especialidade()
{
    public int Id { get; set; }
    public string Nome { get; set; }
}

public class Usuario
{
    public int Id { get; set; }
    public string Login { get; set; }
    public string SenhaHash { get; set; }
}

public class Pessoa
{
    public int Id { get; set; }
    public string Nome { get; set; }
    public string Telefone { get; set; }
}

public class Profissional()
{
    public int Id { get; set; }
    public int EspecialidadeId  { get; set; }
    public string CRM { get; set; }
    public DateTime DataAdesao { get; set; }
}

public class Cliente
{
    public int Id { get; set; }
    public DateTime DataNascimento { get; set; }
    public string CPF { get; set; }
    public int ResponsavelId { get; set; }
}

public class Atendimento
{
    public int Id { get; set; }
    public int ClienteId { get; set; }
    public int ProfissionalId { get; set; }
    public DateTime DataAgendada { get; set; }
    public int StatusId { get; set; }
    public string Observacoes { get; set; }
}

public class Carteirinha
{
    public int Id { get; set; }
    public int PacienteId { get; set; }
    public string Registro { get; set; }
    public int ConvenioId { get; set; }
    public int TipoPlanoId { get; set; }
    public DateTime Validade { get; set; }
}
```
