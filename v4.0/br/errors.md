# Erros e registro

- [Detalhe do erro](#detalhe-do-erro)
- Erros de Manuseio]\(#erros-de-manuseio)
- [Exceções HTTP](#http-exceptions)
- [Tratamento de erros 404](#tratamento-de-erros-404)
- [Registro](#registro)

<a name="error-detail"></a>

## Detalhe do erro

Por padrão, o detalhamento de erros está ativado para sua aplicação. Isso significa que, quando um erro ocorrer, você será exibido uma página de erro com uma depuração detalhada e uma mensagem de erro. Você pode desativar o detalhamento de erros configurando a opção `debug` em seu arquivo `app/config/app.php` para `false`. **É altamente recomendável que você desative o detalhamento de erros em um ambiente de produção.**

<a name="gerenciamento-de-erros">gerenciamento de erros</a>

## Erros de Manuseio

Por padrão, o arquivo `app/start/global.php` contém um manipulador de erros para todas as exceções:

```
App::error(function(Exception $exception)
{
	Log::error($exception);
});
```

Este é o manipulador de erros mais básico. No entanto, você pode especificar mais manipuladores, se necessário. Os manipuladores são chamados com base na dica de tipo da exceção que eles manipulam. Por exemplo, você pode criar um manipulador que trate apenas instâncias de `RuntimeException`:

```
App::error(function(RuntimeException $exception)
{
	// Handle the exception...
});
```

Se um manipulador de exceção retornar uma resposta, essa resposta será enviada ao navegador e nenhum outro manipulador de erro será acionado:

```
App::error(function(InvalidUserException $exception)
{
	Log::error($exception);

	return 'Sorry! Something is wrong with this account!';
});
```

Para ouvir erros fatais do PHP, você pode usar o método `App::fatal`:

```
App::fatal(function($exception)
{
	//
});
```

Se você tiver vários manipuladores de exceções, eles devem ser definidos da forma mais genérica para a mais específica. Por exemplo, um manipulador que lida com todas as exceções do tipo `Exception` deve ser definido antes de um tipo de exceção personalizado, como `Illuminate\Encryption\DecryptException`.

<a name="http-exceptions"></a>

## Exceptions HTTP

As exceções em relação ao HTTP referem-se a erros que podem ocorrer durante uma solicitação do cliente. Isso pode ser um erro de página não encontrada (404), um erro não autorizado (401) ou até mesmo um erro gerado 500. Para retornar uma resposta desse tipo, use o seguinte:

```
App::abort(404, 'Page not found');
```

O primeiro argumento é o código de status HTTP, com a seguinte mensagem personalizada que você gostaria de exibir junto com o erro.

Para levantar uma exceção 401 Unauthorized, basta fazer o seguinte:

```
App::abort(401, 'You are not authorized.');
```

Essas exceções podem ser executadas a qualquer momento durante o ciclo de vida da solicitação.

<a name="gerenciamento-de-erros-404"></a>

## Tratamento de erros 404

Você pode registrar um manipulador de erros que trate todos os erros "404 Not Found" em sua aplicação, permitindo que você retorne páginas de erro 404 personalizadas:

```
App::missing(function($exception)
{
	return Response::view('errors.missing', array(), 404);
});
```

<a name="logging"></a>

## Atividade de registro

As facilidades de registro do Laravel fornecem uma camada simples sobre o poderoso [Monolog](http://github.com/seldaek/monolog). Por padrão, o Laravel é configurado para criar arquivos de registro diários para sua aplicação, e esses arquivos são armazenados em `app/storage/logs`. Você pode escrever informações nesses logs da seguinte forma:

```
Log::info('This is some useful information.');

Log::warning('Something could be going wrong.');

Log::error('Something is really going wrong.');
```

O registro fornece os sete níveis de registro definidos no [RFC 5424](http://tools.ietf.org/html/rfc5424): **debug**, **info**, **notice**, **warning**, **error**, **critical** e **alert**.

Uma série de dados contextuais também pode ser passada para os métodos de log:

```
Log::info('Log message', array('context' => 'Other helpful information'));
```

O Monolog possui uma variedade de manipuladores adicionais que você pode usar para registro. Se necessário, você pode acessar a instância subjacente do Monolog que está sendo usada pelo Laravel:

```
$monolog = Log::getMonolog();
```

Você também pode registrar um evento para capturar todas as mensagens passadas para o log:

**Registrando um Ouvinte de Log**

```
Log::listen(function($level, $message, $context)
{
	//
});
```
