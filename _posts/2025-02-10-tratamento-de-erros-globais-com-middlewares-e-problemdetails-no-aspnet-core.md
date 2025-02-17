---
layout: post
title:  "Tratamento de erros globais com Middleware e ProblemDetails no ASP.NET Core"
date:   2025-02-10 18:03:51 -0300
categories: [rest, dotnet, C#]
---


# Tratamento de erros globais com Middleware e ProblemDetails no ASP.NET Core

## 1. Introdução

Quando APIs RESTful encontram erros, é importante fornecer respostas padronizadas e úteis para o cliente. Mas, em vez de tratar erros individualmente em cada controlador, uma abordagem mais elegante é capturar todas as exceções em um ponto central usando middlewares. Neste artigo, você aprenderá como configurar um **middleware global de tratamento de erros**, retornando mensagens padronizadas utilizando o `ProblemDetails` no ASP.NET Core.

---

## 2. O que é `ProblemDetails`?

Para garantir que APIs RESTful forneçam mensagens de erro detalhadas e padronizadas, precisamos de uma solução robusta. É aqui que o `ProblemDetails` entra como uma ferramenta essencial.

`ProblemDetails` é uma classe do ASP.NET Core baseada na **[Especificação RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807)**, que define um formato padrão para respostas de erro em APIs RESTful. O objetivo é garantir que todas as respostas de erro sigam uma estrutura consistente, facilitando a depuração e automatização pelos clientes.

### **Exemplo visual de uma resposta de erro:**
```json
{
  "type": "https://example.com/errors/validation-error",
  "title": "Validation Failed",
  "status": 400,
  "detail": "One or more validation errors occurred.",
  "instance": "/api/orders/1"
}
```

### **Campos importantes:**
- **`type`**: Link que explica o tipo do erro.
- **`title`**: Um resumo do problema.
- **`status`**: O código de status HTTP.
- **`detail`**: Detalhes adicionais do erro.
- **`instance`**: O recurso que gerou o erro.

---

## 3. Implementando o middleware global

Abaixo está o código completo do middleware, que intercepta exceções, define mensagens de erro apropriadas e retorna uma resposta padronizada utilizando o `ProblemDetails`.

### **Código do middleware:**
```csharp
using System.Net;
using System.Text.Json;
using Microsoft.AspNetCore.Http.Features;
using Microsoft.AspNetCore.Mvc;
using ApplicationException = System.ApplicationException;

namespace WlConsultings.Wallet.Presentation.Middlewares;

public class ExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;

    public ExceptionHandlingMiddleware(RequestDelegate next, ILogger<ExceptionHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            await HandleExceptionAsync(context, ex);
        }
    }

    private async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        _logger.LogError(exception, "An error occurred while processing the request.");

        var statusCode = (int)HttpStatusCode.InternalServerError;
        var title = "An unexpected error occurred.";
        var detail = exception.Message;
        var type = "https://example.com/errors/internal-server-error"; // Valor padrão

        switch (exception)
        {
            case DomainException:
                statusCode = (int)HttpStatusCode.BadRequest;
                title = "A domain validation error occurred.";
                break;

            case NotFoundException:
                statusCode = (int)HttpStatusCode.NotFound;
                title = "The requested resource was not found.";
                break;

            case AlreadyExistsException:
                statusCode = (int)HttpStatusCode.Conflict;
                title = "The resource already exists.";
                break;

            case UnauthorizedAccessException:
                statusCode = (int)HttpStatusCode.Unauthorized;
                title = "Unauthorized access.";
                type = "https://example.com/errors/unauthorized-access";
                break;

            case ApplicationException:
                statusCode = (int)HttpStatusCode.BadRequest;
                title = "An application error occurred.";
                break;
        }

        var activity = context.Features.Get<IHttpActivityFeature>()?.Activity;
        var problemDetails = new ProblemDetails
        {
            Type = exception.GetType().Name,
            Status = statusCode,
            Title = title,
            Detail = detail,
            Type = type,
            Instance = context.Request.Path.ToString(),
            Extensions = new Dictionary<string, object?>
            {
                { "requestId", context.TraceIdentifier },
                { "traceId", activity?.Id },
            }
        };

        context.Response.ContentType = "application/json";
        context.Response.StatusCode = statusCode;

        await context.Response.WriteAsJsonAsync(problemDetails);
    }
}
```

---

## 4. Registrando o middleware no pipeline da aplicação
Para que o middleware seja executado em todas as requisições, precisamos registrá-lo no `Program.cs`. O pipeline do ASP.NET Core é baseado em middleware, onde cada componente pode processar solicitações e respostas. Isso garante que o middleware de exceção capture erros antes de a resposta ser enviada ao cliente.

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

// Adicione o middleware personalizado
app.UseMiddleware<ExceptionHandlingMiddleware>();

app.MapControllers();
app.Run();
```

Ao adicionar o middleware no pipeline, ele será executado para todas as requisições, garantindo que qualquer exceção seja tratada e uma resposta padronizada seja enviada.

---

## 5. Teste do middleware
Vamos simular um erro para verificar o comportamento do middleware. Durante os testes, você também pode depurar os logs de erro gerados pelo middleware usando o console ou uma ferramenta de monitoramento configurada.

### **No `ProductsController`, adicione uma exceção para teste:**
```csharp
[HttpGet("{id}")]
public IActionResult GetProduct(int id)
{
    if (id == 0)
    {
        throw new NotFoundException("The product was not found.");
    }

    return Ok(new { Id = id, Name = "Sample Product" });
}
```

### **Faça uma requisição GET para `/api/products/0` usando o Postman:**

A API deve retornar uma resposta JSON como esta:
```json
{
  "status": 404,
  "title": "The requested resource was not found.",
  "type": "https://example.com/errors/resource-not-found",
  "detail": "The product was not found.",
  "instance": "/api/products/0",
  "requestId": "0HNAA1TL59OVD:00000003",
  "traceId": "00-27c97bc619fcd41e48b522f67f140408-795561222be111e5-00"
}
```

---

## 6. Resumo sobre `requestId` e `traceId`

- **`requestId`**: Identifica a requisição específica e deve ser usado para depuração e suporte ao cliente. Você pode correlacionar erros dessa requisição nos logs do servidor.

- **`traceId`**: Rastreia a requisição em todo o sistema distribuído, sendo ideal para monitoramento em arquiteturas de microserviços ou fluxos complexos. Ferramentas de monitoramento como **Application Insights** e **Jaeger** podem utilizá-lo para fornecer

---
## 7. Conclusão

Usar um middleware global de tratamento de erros com `ProblemDetails` é uma abordagem elegante e eficaz para padronizar erros em APIs RESTful. Isso melhora a experiência do cliente, facilita a depuração e permite a customização das mensagens de erro conforme as necessidades da aplicação.

Agora que você aprendeu essa técnica, experimente integrar o middleware com logs estruturados e ferramentas de monitoramento, como **Serilog**, **Application Insights** ou **Seq**. Essas ferramentas podem fornecer insights detalhados sobre erros e falhas no sistema.

---


