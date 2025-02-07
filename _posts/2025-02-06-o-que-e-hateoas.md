---
layout: post
title:  "O Que é HATEOAS?"
date:   2025-02-06 19:02:51 -0300
categories: [rest, dotnet, C#]
---


# **HATEOAS (Hypermedia As The Engine Of Application State)**

---

## **1. O Que é HATEOAS?**
- **HATEOAS** é um componente fundamental do estilo arquitetural REST.
- Significa **Hypermedia As The Engine Of Application State**.
- Ele permite que a navegação e as interações dentro de uma API sejam **descobertas dinamicamente** por meio de **links** incluídos nas respostas.

### **Exemplo de uma Resposta HATEOAS:**
```json
{
  "id": 123,
  "name": "John Doe",
  "email": "johndoe@example.com",
  "links": [
    { "rel": "self", "href": "/api/users/123" },
    { "rel": "update", "href": "/api/users/123", "method": "PUT" },
    { "rel": "delete", "href": "/api/users/123", "method": "DELETE" },
    { "rel": "orders", "href": "/api/users/123/orders" }
  ]
}
```

---

## **2. Por Que Usar HATEOAS?**

1. **Descoberta Dinâmica**
   - O cliente não precisa conhecer previamente todas as rotas da API.
   - As rotas relevantes são fornecidas dinamicamente pela própria resposta.

2. **Menos Acoplamento**
   - O cliente não depende de URLs fixas.
   - Isso facilita mudanças na estrutura da API sem quebrar o cliente.

3. **Navegação Intuitiva**
   - Assim como navegamos na web por meio de links, os clientes podem navegar em uma API REST usando os links fornecidos.

4. **Melhor Evolução**
   - As APIs podem evoluir sem grandes mudanças nos clientes, desde que mantenham os links adequados.

---

## **3. Quando Usar HATEOAS?**

- **APIs Públicas ou Complexas:**  
  Se a API precisa oferecer **navegação dinâmica** e **facilidade de extensão**, HATEOAS é uma boa escolha.

- **Cenários de Navegação Multi-Relacional:**  
  Quando as entidades têm muitas relações e é importante guiar o cliente de forma dinâmica (por exemplo, perfis com links para conexões e recomendações).

- **APIs de Longo Prazo:**  
  Quando o objetivo é permitir que a API evolua sem exigir mudanças frequentes nos clientes.

---


## **4. Cuidados ao Usar HATEOAS**

1. **Desempenho**
   - Incluir muitos links nas respostas pode aumentar o tamanho do payload e impactar o desempenho.
   - **Cuidados:** Limite os links a apenas aqueles que são relevantes para a operação atual.

2. **Complexidade**
   - Implementar HATEOAS requer planejamento adicional.
   - **Cuidados:** Certifique-se de que os links são consistentes e representam as ações corretas.

3. **Overhead de Desenvolvimento**
   - Criar e manter links dinâmicos pode adicionar esforço extra ao desenvolvimento.
   - **Cuidados:** Automatize a geração de links sempre que possível.

4. **Consistência dos Links**
   - É crucial garantir que os links sempre estejam corretos e apontem para os recursos certos.
   - **Cuidados:** Use testes automatizados para validar a geração de links.

---

## **5. Exemplos Práticos**

### **Cenário 1: API de Carteiras**
**Requisição:**  
`GET /api/wallet/me`

**Resposta com HATEOAS:**
```json
{
  "userId": "71913492-b587-4823-a5eb-aa47fb4567b3",
  "name": "Jose Silva",
  "balance": 1000,
  "links": [
    {
      "rel": "http://localhost:5232/api/wallets/me",
      "href": "self",
      "method": "GET"
    },
    {
      "rel": "http://localhost:5232/api/wallets/deposit",
      "href": "deposit-funds",
      "method": "POST"
    },
    {
      "rel": "http://localhost:5232/api/wallets/withdrawn",
      "href": "withdrawn-funds",
      "method": "POST"
    },
    {
      "rel": "http://localhost:5232/api/wallets/transfer",
      "href": "transfer-funds",
      "method": "POST"
    }
  ]
}
```

### **Cenário 2: API de Usuários**
**Requisição:**  
`GET /api/users/d97db76f-a10b-4a89-a28f-3926b06cb6b6`

**Resposta com HATEOAS:**
```json
{
  "id": "d97db76f-a10b-4a89-a28f-3926b06cb6b6",
  "firstName": "Cleyton",
  "lastName": "Santos",
  "email": "cleyton.santos@example.com",
  "createdAt": "2025-02-06T18:11:19.360577Z",
  "links": [
    {
      "rel": "http://localhost:5232/api/users/d97db76f-a10b-4a89-a28f-3926b06cb6b6",
      "href": "self",
      "method": "GET"
    },
    {
      "rel": "http://localhost:5232/api/users/d97db76f-a10b-4a89-a28f-3926b06cb6b6",
      "href": "update-user",
      "method": "PUT"
    },
    {
      "rel": "http://localhost:5232/api/users/d97db76f-a10b-4a89-a28f-3926b06cb6b6",
      "href": "delete-user",
      "method": "DELETE"
    }
  ]
}
```

---

## **6. Ferramentas e Bibliotecas Úteis**

- **ASP.NET Core:**  
  O ASP.NET Core permite adicionar facilmente links HATEOAS usando bibliotecas como o **`Microsoft.AspNetCore.Mvc.Routing`**.

```c#
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Infrastructure;
using Microsoft.AspNetCore.Mvc.Routing;

namespace Codepride.Wallet.Presentation.Hateoas;

public class LinkGeneratorService(IUrlHelperFactory urlHelperFactory, IActionContextAccessor actionContextAccessor)
{
    private readonly IUrlHelper _urlHelper = urlHelperFactory.GetUrlHelper(actionContextAccessor.ActionContext!);
    
    private const string UsersRouteName = "users";
    private const string WalletsRouteName = "wallets";

    public List<Link> GenerateLinksForUser(Guid userId)
    {
        return
        [
            new Link(_urlHelper.Link(UsersRouteName, new { id = userId })!, "self", HttpMethods.Get),
            new Link(_urlHelper.Link(UsersRouteName, new { id = userId })!, "update-user", HttpMethods.Put),
            new Link(_urlHelper.Link(UsersRouteName, new { id = userId })!, "delete-user", HttpMethods.Delete)
        ];
    }
    
    public List<Link> GenerateLinksForWallet(Guid userId)
    {
        return
        [
            new Link(_urlHelper.Link("GetWalletBalance", null)!, "self", HttpMethods.Get),
            new Link(_urlHelper.Link("DepositFunds", null)!, "deposit-funds", HttpMethods.Post),
            new Link(_urlHelper.Link("WithdrawnFunds", null)!, "withdrawn-funds", HttpMethods.Post),
            new Link(_urlHelper.Link("TransferFunds", new { id = userId })!, "transfer-funds", HttpMethods.Post)
        ];
    }
}   
```
  E no Controller

```c#
using AutoMapper;
using MediatR;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Codepride.Wallet.Application.UseCases.Users.GetUserById;
using Codepride.Wallet.Application.UseCases.Users.Register;
using Codepride.Wallet.Presentation.ApiContracts.Users.GetUserById;
using Codepride.Wallet.Presentation.ApiContracts.Users.Register;
using Codepride.Wallet.Presentation.Hateoas;

namespace Codepride.Wallet.Presentation.Controllers;

[ApiController]
[Route("api/users")]
[Authorize]
public class UsersController(IMediator mediator, IMapper mapper, LinkGeneratorService linkGeneratorService) : ControllerBase
{
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound, Type = typeof(ProblemDetails))]
    [ProducesResponseType(StatusCodes.Status500InternalServerError, Type = typeof(ProblemDetails))]
    [AllowAnonymous]
    public async Task<ActionResult<RegisterUserResponse>> Register(RegisterUserRequest request)
    {
        var command = mapper.Map<RegisterUserCommand>(request);
        var result = await mediator.Send(command);
        var response = mapper.Map<RegisterUserResponse>(result);
        
        response.Links = linkGeneratorService.GenerateLinksForUser(response.Id);
        
        return CreatedAtRoute("users", new { id = response.Id }, response);
    }
    
    [HttpGet("{id:guid}", Name = "users")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound, Type = typeof(ProblemDetails))]
    [ProducesResponseType(StatusCodes.Status500InternalServerError, Type = typeof(ProblemDetails))]
    public async Task<ActionResult<GetUserByIdResponse>> GetUserById(Guid id)
    {
        var query = new GetUserByIdQuery { Id = id };
        var result = await mediator.Send(query);
        var response = mapper.Map<GetUserByIdResponse>(result);
        
        response.Links = linkGeneratorService.GenerateLinksForUser(response.Id);
        
        return Ok(response);
    }
    
}
```

```c#
using AutoMapper;
using MediatR;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Codepride.Wallet.Application.Contracts.Services;
using Codepride.Wallet.Application.UseCases.Wallets.GetWalletBalance;
using Codepride.Wallet.Presentation.ApiContracts.Wallets.GetWalletBalance;
using Codepride.Wallet.Presentation.Hateoas;

namespace Codepride.Wallet.Presentation.Controllers;

[ApiController]
[Route("api/wallets")]
[Authorize]
public class WalletsController(IMediator mediator, IMapper mapper, LinkGeneratorService linkGeneratorService, ICurrentUserService currentUserService) : ControllerBase
{
    [HttpGet("me", Name = "GetWalletBalance")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound, Type = typeof(ProblemDetails))]
    [ProducesResponseType(StatusCodes.Status500InternalServerError, Type = typeof(ProblemDetails))]
    public async Task<ActionResult<GetWalletBalanceResponse>> GetWalletBalance()
    {
        var query = new GetWalletBalanceQuery { UserId = currentUserService.UserId.GetValueOrDefault() };
        var result = await mediator.Send(query);
        var response = mapper.Map<GetWalletBalanceResponse>(result);
        
        response.Links = linkGeneratorService.GenerateLinksForWallet(response.UserId);
        
        return Ok(response);
    }
}
```

---

## **7. Conclusão**

- **Quando usar:**  
  HATEOAS é ideal para APIs complexas, públicas ou de longo prazo, onde é importante reduzir o acoplamento e permitir evolução.

- **O que ele melhora:**  
  Flexibilidade, navegação dinâmica e facilidade de evolução.

- **Cuidados:**  
  Limite os links às operações relevantes, mantenha os links consistentes e valide o impacto no desempenho.

---

## **8. Referências**
- [Fielding, Roy. REST: Architectural Styles and the Design of Network-based Software Architectures](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
- [Microsoft Docs - ASP.NET Core Web API Best Practices](https://learn.microsoft.com/en-us/aspnet/core/web-api/)
