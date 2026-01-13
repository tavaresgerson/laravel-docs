# Correio

- [Configuração](#configuração)
- [Uso básico](#uso-básico)
- [Incorporação de anexos inline](#incorporacao-de-anexos-inline)
- [Encaminhar e-mail para fila](#encaminhar-e-mail-para-fila)
- [Correio e Desenvolvimento Local](#correio-e-desenvolvimento-local)

<a name="configuração"></a>

## Configuração

O Laravel oferece uma API limpa e simples sobre a popular biblioteca [SwiftMailer](http://swiftmailer.org). O arquivo de configuração de e-mail está em `app/config/mail.php` e contém opções que permitem alterar o host, a porta e as credenciais do SMTP, além de definir um endereço `from` global para todas as mensagens entregues pela biblioteca. Você pode usar qualquer servidor SMTP que desejar. Se você deseja usar a função `mail` do PHP para enviar e-mails, pode alterar o `driver` para `mail` no arquivo de configuração. Um driver `sendmail` também está disponível.

<a name="uso-básico"></a>

## Uso básico

O método `Mail::send` pode ser usado para enviar uma mensagem de e-mail:

```
Mail::send('emails.welcome', $data, function($message)
{
	$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
});
```

O primeiro argumento passado para o método `send` é o nome da vista que deve ser usado como o corpo do e-mail. O segundo é o `$data` que deve ser passado para a vista, e o terceiro é uma Closure que permite especificar várias opções na mensagem de e-mail.

> **Observação:**
>
>  Uma variável 
>
> `$message`
>
>  é sempre passada para as visualizações de e-mail e permite a incorporação em linha de anexos. Portanto, é melhor evitar passar uma variável 
>
> `message`
>
>  no payload da visualização.

Você também pode especificar uma visualização de texto simples para usar em adição a uma visualização HTML:

```
Mail::send(array('html.view', 'text.view'), $data, $callback);
```

Ou você pode especificar apenas um tipo de visualização usando as chaves `html` ou `text`:

```
Mail::send(array('text' => 'view'), $data, $callback);
```

Você pode especificar outras opções na mensagem de e-mail, como cópias de carbono ou anexos:

```
Mail::send('emails.welcome', $data, function($message)
{
	$message->from('us@example.com', 'Laravel');

	$message->to('foo@example.com')->cc('bar@example.com');

	$message->attach($pathToFile);
});
```

Ao anexar arquivos a uma mensagem, você também pode especificar um tipo MIME e/ou um nome de exibição:

```
$message->attach($pathToFile, array('as' => $display, 'mime' => $mime));
```

> **Nota:**
>
>  A instância da mensagem passada para um Closure 
>
> `Mail::send`
>
>  estende a classe de mensagem SwiftMailer, permitindo que você chame qualquer método dessa classe para criar suas mensagens de e-mail.

<a name="incorporação-de-anexos-inline"></a>

## Incorporação de anexos inline

Integrar imagens em linha nos seus e-mails é geralmente trabalhoso; no entanto, o Laravel oferece uma maneira conveniente de anexar imagens aos seus e-mails e recuperar o CID apropriado.

**Colocando uma Imagem em uma Visualização de E-mail**

```
<body>
	Here is an image:

	<img src="<?php echo $message->embed($pathToFile); ?>">
</body>
```

**Incorporação de Dados Brutos em uma Visualização de E-mail**

```
<body>
	Here is an image from raw data:

	<img src="<?php echo $message->embedData($data, $name); ?>">
</body>
```

Observe que a variável `$message` é sempre passada para as visualizações de e-mail pela classe `Mail`.

<a name="queueing-mail"></a>

## Fila de Correio

Como o envio de mensagens de e-mail pode prolongar drasticamente o tempo de resposta do seu aplicativo, muitos desenvolvedores optam por colocar as mensagens de e-mail em fila para envio em segundo plano. O Laravel facilita isso usando sua API de fila unificada embutida (/docs/queues). Para colocar uma mensagem de e-mail em fila, basta usar o método `queue` na classe `Mail`:

**Aguardar uma mensagem de e-mail**

```
Mail::queue('emails.welcome', $data, function($message)
{
	$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
});
```

Você também pode especificar o número de segundos que deseja adiar o envio da mensagem de e-mail usando o método `later`:

```
Mail::later(5, 'emails.welcome', $data, function($message)
{
	$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
});
```

Se você deseja especificar uma fila ou "tubo" específico para empurrar a mensagem, você pode fazer isso usando os métodos `queueOn` e `laterOn`:

```
Mail::queueOn('queue-name', 'emails.welcome', $data, function($message)
{
	$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
});
```

<a name="mail-e-desenvolvimento-local"></a>

## Correio e Desenvolvimento Local

Ao desenvolver uma aplicação que envia e-mails, geralmente é desejável desabilitar o envio de mensagens do seu ambiente local ou de desenvolvimento. Para fazer isso, você pode chamar o método `Mail::pretend` ou definir a opção `pretend` no arquivo de configuração `app/config/mail.php` para `true`. Quando o remetente estiver no modo `pretend`, as mensagens serão escritas nos arquivos de log da sua aplicação em vez de serem enviadas ao destinatário.

**Habilitar o Modo de Correio Fictício**

```
Mail::pretend();
```
