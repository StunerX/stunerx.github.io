# 🚀 Logging Estruturado no .NET com Serilog (Parte 1)

## 📌 Introdução

O logging estruturado é uma prática essencial para aplicações modernas, especialmente em arquiteturas distribuídas e microsserviços. Com logs bem estruturados, conseguimos rastrear eventos importantes, diagnosticar falhas e monitorar o comportamento da aplicação com mais eficiência.

Nesta série de artigos, vamos explorar o uso de **Serilog** no .NET, sua configuração e integração com ferramentas como **Elasticsearch, Kibana e Seq**. Este primeiro artigo foca exclusivamente no **Serilog**, explicando sua importância, como configurá-lo e boas práticas para obter logs úteis e organizados.

## 🛠️ O que é Serilog?

O **Serilog** é uma biblioteca de logging estruturado para .NET que permite armazenar logs de maneira flexível e organizada. Ele suporta múltiplos destinos (**sinks**) como:
- **Console** (logs no terminal)
- **Arquivos** (logs persistentes)
- **Banco de dados** (SQL Server, PostgreSQL, MongoDB)
- **Elasticsearch** (logs indexados e pesquisáveis)
- **Seq** (ferramenta de análise de logs estruturados)

Serilog é **altamente configurável**, permitindo enriquecer logs com metadados úteis, como IDs de requisição, nome do servidor e outras informações relevantes.

## 🔧 Instalando o Serilog

Para começar, adicione as seguintes dependências ao seu projeto .NET:

```xml
<PackageReference Include="Serilog" Version="4.2.0" />
<PackageReference Include="Serilog.Enrichers.CorrelationId" Version="3.0.1" />
<PackageReference Include="Serilog.Sinks.Console" Version="6.0.0" />
<PackageReference Include="Serilog.Sinks.File" Version="5.0.0" />
<PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />

```

Se estiver usando o **.NET CLI**, instale os pacotes com:

```sh
dotnet add package Serilog
dotnet add package Serilog.Enrichers.CorrelationId
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
dotnet add package Serilog.AspNetCore

```

## ⚙️ Configurando Serilog no .NET

Agora, vamos configurar o Serilog no `Program.cs` para capturar logs e exibi-los no console e em um arquivo.

### 1️⃣ Configuração Simples no Código

```csharp
using Serilog;
using Serilog.Events;
using Serilog.Sinks.Elasticsearch;

var builder = WebApplication.CreateBuilder(args);

// Configuração do Serilog diretamente no código
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
    .MinimumLevel.Override("System", LogEventLevel.Warning)
    .MinimumLevel.Override("Microsoft.EntityFrameworkCore.Database.Command", LogEventLevel.Warning)
    .Enrich.WithCorrelationId()
    .Enrich.WithMachineName()
    .WriteTo.Console()
    .WriteTo.File("logs/log.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();

builder.Host.UseSerilog();

var app = builder.Build();

app.UseSerilogRequestLogging();
app.MapGet("/", () => "Hello World!");
app.Run();
```

### 📌 Quando usar essa abordagem?
✅ **Projetos pequenos e simples** onde o nível de log raramente muda.  
✅ **Ambientes de desenvolvimento** onde recompilar não é um problema.  

### ⚠️ Limitações dessa abordagem
❌ Se precisar mudar o nível de log (ex.: de `Information` para `Debug`), será necessário **alterar o código e recompilar o projeto**.

---

## 🔄 📌 Alternativa: Alterando o Nível de Log e Enrichers via `appsettings.json` (Sem Rebuild)

Para evitar a necessidade de recompilar o código ao mudar o nível de log e enriquecer as informações, podemos configurar o **Serilog** para ler tudo do `appsettings.json`.

### 1️⃣ Configuração no `appsettings.json`

```json
"Serilog": {
  "MinimumLevel": "Information",
  "Override": {
    "Microsoft": "Warning",
    "System": "Warning"
  },
  "Enrich": [
    "FromLogContext",
    "WithMachineName",
    "WithCorrelationId"
  ],
  "WriteTo": [
    {
      "Name": "Console"
    },
    {
      "Name": "File",
      "Args": { "path": "logs/log.txt", "rollingInterval": "Day" }
    }
  ]
}
```

### 2️⃣ Ajustando o `Program.cs`

```csharp
using Serilog;
using Serilog.Settings.Configuration;

var builder = WebApplication.CreateBuilder(args);

Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)
    .CreateLogger();

builder.Host.UseSerilog();

var app = builder.Build();

app.UseSerilogRequestLogging();
app.MapGet("/", () => "Hello World!");
app.Run();
```

### 🎯 Benefícios dessa Abordagem
✅ **Permite alterar o nível de log sem recompilar o código.**  
✅ **Enrichers também podem ser configurados dinamicamente.**  
✅ **Ótima para produção**, onde ajustes rápidos são necessários.  
✅ **Facilidade para ajustes em diferentes ambientes** (Desenvolvimento, Produção, Testes).  
✅ **Melhor prática para DevOps**, pois toda a configuração fica centralizada no `appsettings.json`.

---

## 🛡️ Boas Práticas no Uso de Logs

1. **Defina níveis de log apropriados** (`Information`, `Warning`, `Error`).
2. **Sempre inclua `CorrelationId` e `MachineName`**, para melhor rastreabilidade.
3. **Use logs estruturados com parâmetros** para facilitar buscas.
4. **Evite logar dados sensíveis**, como senhas e tokens.
5. **Rotacione arquivos de log** para evitar crescimento excessivo.
6. **Integre logs com ferramentas de monitoramento**, como Elasticsearch e Seq.

## 🎯 Conclusão

Neste artigo, exploramos os conceitos básicos do **Serilog**, como configurá-lo e registrar logs estruturados no .NET. Também aprendemos a **alterar dinamicamente o nível de log e os Enrichers** sem precisar recompilar o código.

Nos próximos artigos, veremos como integrar o Serilog com:

➡️ **Parte 2:** Armazenando Logs no **Elasticsearch** 🔍  
➡️ **Parte 3:** Visualizando Logs no **Kibana** 📊  
➡️ **Parte 4:** Analisando Logs com **Seq** 🔎  
