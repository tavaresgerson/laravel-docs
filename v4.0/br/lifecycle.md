# Solicitação do ciclo de vida

- [Visão geral](#visão-geral)
- [Iniciar arquivos](#iniciar-arquivos)
- Eventos de Aplicação [Eventos de Aplicação](#application-events)

<a name="sobreavaliação"></a>

## Visão geral

O ciclo de vida de solicitações do Laravel é bastante simples. Uma solicitação entra na sua aplicação e é enviada para a rota ou controlador apropriados. A resposta dessa rota é então enviada de volta ao navegador e exibida na tela. Às vezes, você pode querer realizar algum processamento antes ou depois de as rotas serem realmente chamadas. Existem várias oportunidades para fazer isso, sendo dois deles os arquivos "start" e os eventos da aplicação.

<a name="start-files"></a>

## Comece com Arquivos

Os arquivos de início da sua aplicação estão armazenados em `app/start`. Por padrão, três deles estão incluídos na sua aplicação: `global.php`, `local.php` e `artisan.php`. Para mais informações sobre `artisan.php`, consulte a documentação sobre o [linha de comando Artisan](/docs/commands#registering-commands).

O arquivo de início `global.php` contém alguns itens básicos por padrão, como o registro do [Logger](/docs/errors) e a inclusão do seu arquivo `app/filters.php`. No entanto, você é livre para adicionar qualquer coisa a este arquivo que desejar. Ele será incluído automaticamente em *cada* pedido para sua aplicação, independentemente do ambiente. O arquivo `local.php`, por outro lado, é chamado apenas quando a aplicação está sendo executada no ambiente `local`. Para mais informações sobre ambientes, consulte a documentação da [configuração](/docs/configuration).

Claro, se você tiver outros ambientes além do `local`, você também pode criar arquivos de inicialização para esses ambientes. Eles serão incluídos automaticamente quando sua aplicação estiver em execução nesse ambiente.

<a name="eventos-de-aplicação"></a>

## Eventos de Aplicação

Você também pode realizar o processamento de solicitações pré e pós-requisito ao registrar eventos de aplicativo `before`, `after`, `close`, `finish` e `shutdown`:

**Registrar eventos de aplicativo**

```
App::before(function($request)
{
	//
});

App::after(function($request, $response)
{
	//
});
```

Os ouvintes desses eventos serão executados `antes` e `depois` de cada solicitação à sua aplicação.
