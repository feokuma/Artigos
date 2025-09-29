# Testes automatizados

Durante minha carreira, várias vezes me deparei com times em que a separação entre o que é um teste de integração e um teste de unidade não estavam muito clara para as pessoas que estavam ali escrevendo esses testes e isso sempre rende discussões muito interessantes. Neste artigo vou tentar simplificar um pouco essa discussão mas tentar trazer as informações que sempre trago nessas conversas.

Testes automatizados são fundamentais para garantir qualidade, reduzir falhas em produção e permitir evolução contínua do sistema. Dois tipos de testes se destacam nesse processo: testes de unidade e testes de integração.
Apesar de complementares, cada um possui um objetivo diferente e deve ser aplicado em momentos distintos do ciclo de desenvolvimento.

Neste artigo utilizarei exemplos em C# mas podemos os conceitos podem ser considerados para outras tecnologias

## O que são testes de unidade?

Testes de unidade têm como foco avaliar uma parte muito pequena e isolada do código — geralmente uma função, classe ou método então devem isolar a lógica sem depender de serviços externos como banco de dados, APIs, cache ou fila de mensagens. Para isso, usamos mocks.

### Características principais

- Escopo reduzido (testa apenas uma unidade de código).
- Executam rapidamente.
- Facilitam a detecção de erros logo na fase inicial.
- Não dependem de banco de dados, rede ou serviços externos (devem usar mocks ou stubs).

### Cenário

Temos um serviço de pedidos que depende de:

- Um repositório para salvar no banco.
- Um serviço de notificação para avisar o cliente quando o pedido for criado.

```csharp
public class PedidoService
{
    private readonly IPedidoRepository _repo;
    private readonly INotificador _notificador;

    public PedidoService(IPedidoRepository repo, INotificador notificador)
    {
        _repo = repo;
        _notificador = notificador;
    }

    public void CriarPedido(Pedido pedido)
    {
        if (pedido.Valor <= 0)
            throw new ArgumentException("Pedido inválido");

        _repo.Salvar(pedido);
        _notificador.EnviarEmail(pedido.Cliente, "Pedido criado com sucesso!");
    }
}
```

### Problema

Se rodarmos um teste real aqui:

- Ele tentaria gravar no banco de dados real.
- Ele enviaria um e-mail real.

Isso não é um teste de unidade, mas sim um teste de integração.
No teste de unidade, queremos validar apenas a lógica da classe PedidoService, sem executar ações externas.

### Solução: usar Mocks

Neste cenário queremos garantir que se o valor do pedido for maior que zero o os métodos **_repo.Salvar** e o **_notificador.EnviarEmail** são chamados com os parâmetros esperados e, não nos importamos com o que acontece dentro deles, mas somente se são chamados como esperado.

```csharp
using Moq;
using Xunit;

public class PedidoServiceTests
{
    [Fact]
    public void CriarPedido_DeveSalvarPedidoENotificarCliente()
    {
        // Arrange
        var pedido = new Pedido { Cliente = "Maria", Valor = 100, PercentualDesconto = 0.1 };

        var repoMock = new Mock<IPedidoRepository>();
        var notificadorMock = new Mock<INotificador>();

        var service = new PedidoService(repoMock.Object, notificadorMock.Object);

        // Act
        service.CriarPedido(pedido);

        // Assert
        repoMock.Verify(r => r.Salvar(pedido), Times.Once);
        notificadorMock.Verify(n => n.EnviarEmail("Maria", "Pedido criado com sucesso!"), Times.Once);
    }
}
```

### O que esse exemplo mostra

- Banco de dados (IPedidoRepository) foi mockado → não grava nada de verdade.
- Serviço de e-mail (INotificador) foi mockado → não dispara e-mail real.
- O teste só valida a regra da classe PedidoService:
  - Se valor > 0, salva e notifica.
  - Se valor <= 0, lança exceção.

Assim, garantimos que o comportamento da classe está correto, sem depender de infraestrutura externa.

#### O que estamos fazendo aqui?

Neste gráfico é possível entender visualmente onde as dependências são substituídas por mocks.
![Testes de unidade](Testes%20de%20unidade.svg)

### Mas o que é um Mock afinal?

Um mock é uma versão falsa ou simulada de uma dependência real usada em um teste de unidade.
Ele substitui objetos que, no mundo real, fariam operações externas — como acessar banco de dados, chamar uma API, enviar e-mails ou gravar em arquivos.

A ideia é que, em vez de usar a implementação de verdade (que poderia ser lenta, instável ou até gerar efeitos colaterais indesejados), criamos uma cópia controlada, que apenas registra interações ou retorna valores pré-programados.

#### Características de um Mock

- Isolamento: impede que o teste dependa de sistemas externos.
- Controle: podemos configurar o que ele deve retornar.
- Verificação: permite checar se um método foi chamado, quantas vezes e com quais parâmetros.

#### Exemplo prático em pseudocódigo

```plaintext
Dado: um serviço de pedidos que envia e-mail
Quando: eu criar um pedido válido
Então: o mock do e-mail deve registrar que "EnviarEmail" foi chamado 1 vez
```

**Resumindo**: Um mock não implementa a lógica real, mas sim simula o comportamento esperado para que possamos testar apenas a lógica da classe em foco.

---

## O que são os testes de integração?

Testes de integração são aqueles que verificam se diferentes partes de um sistema funcionam corretamente quando trabalham juntas.
Enquanto os testes de unidade isolam pequenas partes do código (ex.: uma função ou classe), os testes de integração validam a colaboração entre componentes — como serviços, repositórios, banco de dados, APIs externas e filas de mensagens.

### Objetivos principais

- Garantir que módulos independentes se comuniquem corretamente.
- Detectar problemas que só aparecem quando há troca de dados entre camadas.
- Validar configurações reais (conexões, credenciais, serialização, etc.).

### Caracteristicas dos testes de integração

- Mais lentos que testes de unidade (dependem de banco, rede, containers).
- Mais complexos (exigem preparar ambientes de teste).
- Menos numerosos: geralmente escrevemos menos testes de integração do que de unidade, porque eles são mais caros de manter.
- Mais próximos da realidade: mostram se o sistema funciona como esperado quando as peças se juntam.

### Exemplo da aplicação de pedidos

Vou utilizar aqui exemplo da aplicação de pedidos, mas considerando somente que vamos gravar o pedido do banco de dados.

#### Implementação a aplicação de pedidos

Vou mostrar aqui os principais trechos de código para permitir que a aplicação tenha um endpoint para gravar um novo pedido, uma service e um repositório, em memória, que compõe o básico da aplicação.

1. Repositório, e sua interface, responsável pela persistência dos pedidos
    - Esta implementação é feita com um **List** para simplificar o exemplo

    ```csharp
    public interface IPedidoRepository
    {
        void Salvar(Pedido pedido);
        Pedido? BuscarPorCliente(string cliente);
    }

    public class PedidoRepository : IPedidoRepository
    {
        private readonly List<Pedido> _pedidos;

        public PedidoRepository(List<Pedido> pedidos)
        {
            _pedidos = pedidos;
        }

        public void Salvar(Pedido pedido) => _pedidos.Add(pedido);
        public Pedido? BuscarPorCliente(string cliente) =>
            _pedidos.FirstOrDefault(p => p.Cliente == cliente);
    }
    ```

1. Implementação do PedidoService onde o repositório é utilizado

    ```csharp
    public class PedidoService
    {
        private readonly IPedidoRepository _repo;

        public PedidoService(IPedidoRepository repo)
        {
            _repo = repo;
        }

        public void CriarPedido(Pedido pedido)
        {
            if (pedido.Valor <= 0)
                throw new ArgumentException("Pedido inválido");

            _repo.Salvar(pedido);
        }
    }
    ```

1. No Program.cs registramos o repositório e o service para pedidos além é claro da controller que irá consumir utilizar o **PedidoService**
    - O **pedidosList** é registrado é registrado no container de injeção de dependência assim como um DbContext também seria.

    ```csharp
    var builder = WebApplication.CreateBuilder(args);

    // Criamos uma lista em memória para simular o banco
    var pedidosList = new List<Pedido>();

    // Registramos essa lista no DI container
    builder.Services.AddSingleton(pedidosList);
    builder.Services.AddScoped<IPedidoRepository, PedidoRepository>();
    builder.Services.AddScoped<PedidoService>();

    var app = builder.Build();

    app.MapPost("/pedidos", (Pedido pedido, PedidoService service) =>
    {
        service.CriarPedido(pedido);
        return Results.Created($"/pedidos/{pedido.Id}", pedido);
    });

    app.MapGet("/pedidos/{cliente}", (string cliente, IPedidoRepository repo) =>
    {
        var pedido = repo.BuscarPorCliente(cliente);
        return pedido is not null ? Results.Ok(pedido) : Results.NotFound();
    });

    app.Run();

    public partial class Program { } // Necessário para o WebApplicationFactory

    ```

Esse é o mínimo de implementação necessário para permitir que a nossa aplicação AspNet Core possa receber uma requisição de **Post** para gravar o pedido.

#### Implementando o teste de integração

Considerando a nossa aplicação, vou mostrar a seguir como ficaria o setup de a implementação do teste de integração.

```csharp
using System.Net.Http.Json;
using Microsoft.AspNetCore.Mvc.Testing;
using Microsoft.Extensions.DependencyInjection;
using Xunit;

public class PedidoIntegrationTests 
    : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;
    private readonly ServiceProvider _serviceProvider;

    public PedidoIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
        _client = factory.CreateClient();
        _serviceProvider = factory.Services.CreateScope().ServiceProvider;
    }

    [Fact]
    public async Task CriarPedido_DeveSalvarNaLista()
    {
        // Arrange
        var pedido = new Pedido(1, "Maria", 100, PercentualDesconto = 0.1);

        // Act - POST
        var responsePost = await _client.PostAsJsonAsync("/pedidos", pedido);

        // Assert - verifica diretamente na lista em memória
        var pedidos = _serviceProvider.GetRequiredService<List<Pedido>>();

        Assert.Equal(System.Net.HttpStatusCode.Created, responsePost.StatusCode);
        Assert.Contains(pedidos, p => p.Cliente == "Maria" && p.Valor == 100);
    }
}
```

Aqui valem algumas explicações. O **[WebApplicationFactory](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1?view=aspnetcore-9.0)** é uma ferramenta que permite subir nossa aplicação em memória e como pode ser observado, ele disponibiliza um objeto do tipo **HttpClient** onde todas as requisições feitas com este objeto são direcionadas para a nossa aplicação. Isso fabilita muito o setup dos nossos testes.

Ao executar os nossos testes o que estamos fazendo é o seguinte:

1. Criamos um pedido que será enviado para o endpoint para ser gravado.
1. Fazemos a requisição com o **HttpClient** disponibilizado pelo WebApplicationFactory.
1. Obtermos do nosso container de injeção de dependência a instância do nosso repositório (que neste caso é a nossa lista em memória).
1. Validamos se o método utilizado status code da resposta é o esperado.
1. Verificamos diretamente no repositório se o conteúdo foi gravado para esperávamos.
    - Não utilizamos o método **BuscarPorCliente** do repositório para evitar que, caso haja algum erro neste método de busca, isso não afetará o teste do **POST**

**Resumindo**: Testes de integração confirmam que o todo funciona em conjunto, não apenas as partes isoladas.

## Referências

- [Moq](https://github.com/devlooped/moq)
