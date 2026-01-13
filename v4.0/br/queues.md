# Fila de espera

- [Configuração](#configuração)
- [Uso básico](#uso-básico)
- [Fechamento de filas](#fechamento-de-filas)
- [Executando o Listener de Fila](#executando-o-listener-de-fila)
- [Fila de empurrão](#fila-de-empurrao)

<a name="configuração"></a>

## Configuração

O componente Laravel Queue oferece uma API unificada para uma variedade de serviços de filas diferentes. As filas permitem que você adiaste o processamento de uma tarefa que leva tempo, como enviar um e-mail, para um momento posterior, acelerando drasticamente as solicitações da web para sua aplicação.

O arquivo de configuração da fila está armazenado em `app/config/queue.php`. Neste arquivo, você encontrará as configurações de conexão para cada um dos drivers de fila incluídos no framework, que incluem um [Beanstalkd](http://kr.github.com/beanstalkd), [IronMQ](http://iron.io), [Amazon SQS](http://aws.amazon.com/sqs) e o driver síncrono (para uso local).

As seguintes dependências são necessárias para os drivers de fila listados:

- Beanstalkd: `pda/pheanstalk`
- Amazon SQS: `aws/aws-sdk-php`
- IronMQ: `iron-io/iron_mq`

<a name="uso-básico"></a>

## Uso básico

Para adicionar um novo trabalho à fila, use o método `Queue::push`:

**Empurrando um trabalho para a fila**

```
Queue::push('SendEmail', array('message' => $message));
```

O primeiro argumento fornecido ao método `push` é o nome da classe que deve ser usada para processar o trabalho. O segundo argumento é um array de dados que deve ser passado ao manipulador. Um manipulador de trabalho deve ser definido da seguinte forma:

**Definindo um Gestor de Emprego**

```
class SendEmail {

	public function fire($job, $data)
	{
		//
	}

}
```

Observe que o único método necessário é `fire`, que recebe uma instância de `Job`, bem como o array de `data` que foi empurrado para a fila.

Se você quiser que o trabalho use um método diferente de `fire`, você pode especificar o método ao empurrar o trabalho:

**Especificando um método de manipulador personalizado**

```
Queue::push('SendEmail@send', array('message' => $message));
```

Depois de processar um trabalho, ele deve ser excluído da fila, o que pode ser feito por meio do método `delete` na instância `Job`:

**Excluindo um trabalho processado**

```
public function fire($job, $data)
{
	// Process the job...

	$job->delete();
}
```

Se você deseja liberar um trabalho de volta na fila, pode fazê-lo através do método `release`:

**Retirar um trabalho da fila**

```
public function fire($job, $data)
{
	// Process the job...

	$job->release();
}
```

Você também pode especificar o número de segundos para esperar antes que o trabalho seja liberado:

```
$job->release(5);
```

Se uma exceção ocorrer enquanto o trabalho estiver sendo processado, ele será automaticamente liberado de volta na fila. Você pode verificar o número de tentativas que foram feitas para executar o trabalho usando o método `attempts`:

**Verificar o número de tentativas de execução**

```
if ($job->attempts() > 3)
{
	//
}
```

Você também pode acessar o identificador do trabalho:

**Acesse o ID do trabalho**

```
$job->getJobId();
```

<a name="queueing-closures"></a>

## Fechamento de filas

Você também pode empurrar uma Fechamento para a fila. Isso é muito conveniente para tarefas rápidas e simples que precisam ser agendadas:

**Empurrando uma Fechadura para a Fila**

```
Queue::push(function($job) use ($id)
{
	Account::delete($id);

	$job->delete();
});
```

> **Observação:**
>
>  Ao empurrar Closures na fila, as constantes 
>
> `__DIR__`
>
>  e 
>
> `__FILE__`
>
>  não devem ser usadas.

Ao usar o Iron.io [filas de envio](#filas-de-envio), você deve tomar precauções extras ao agendar Closures. O ponto final que recebe suas mensagens da fila deve verificar um token para verificar se o pedido realmente é do Iron.io. Por exemplo, seu ponto final de fila de envio deve ser algo como: `https://yourapp.com/queue/receive?token=SecretToken`. Você pode então verificar o valor do token secreto em seu aplicativo antes de mapear o pedido da fila.

<a name="running-the-queue-listener"></a>

## Executando o Ouvinte de Fila

O Laravel inclui uma tarefa do Artisan que executa novos trabalhos à medida que são empilhados na fila. Você pode executar essa tarefa usando o comando `queue:listen`:

**Iniciando o Ouvidor de Fila**

```
php artisan queue:listen
```

Você também pode especificar qual conexão de fila o ouvinte deve utilizar:

```
php artisan queue:listen connection
```

Observe que, uma vez que essa tarefa tenha sido iniciada, ela continuará a ser executada até que seja interrompida manualmente. Você pode usar um monitor de processos, como [Supervisor](http://supervisord.org/), para garantir que o ouvinte da fila não pare de funcionar.

Você também pode definir o tempo (em segundos) que cada tarefa deve ser permitida para executar:

**Especificando o Parâmetro de Timeout do Trabalho**

```
php artisan queue:listen --timeout=60
```

Além disso, você pode especificar o número de segundos para esperar antes de coletar novos trabalhos:

```
php artisan queue:listen --sleep=5
```

Para processar apenas o primeiro trabalho na fila, você pode usar o comando `queue:work`:

**Processando o Primeiro Trabalho na Fila**

```
php artisan queue:work
```

<a name="push-queues"></a>

## Empurrar filas

As filas de envio permitem que você utilize as poderosas facilidades de filas do Laravel 4 sem precisar executar nenhum daemon ou ouvinte em segundo plano. Atualmente, as filas de envio são suportadas apenas pelo driver [Iron.io](http://iron.io). Antes de começar, crie uma conta do Iron.io e adicione suas credenciais do Iron ao arquivo de configuração `app/config/queue.php`.

Em seguida, você pode usar o comando `queue:subscribe` do Artisan para registrar um ponto final de URL que receberá os trabalhos da fila recém-impulsionados:

**Registrar um assinante da fila de envio**

```
php artisan queue:subscribe queue_name http://foo.com/queue/receive
```

Agora, ao fazer login no seu painel do Iron, você verá sua nova fila de envio, além da URL subscrita. Você pode subscrever quantas URLs quiser a uma determinada fila. Em seguida, crie uma rota para o ponto final `queue/receive` e retorne a resposta do método `Queue::marshal`:

```
Route::post('queue/receive', function()
{
	return Queue::marshal();
});
```

O método `marshal` cuidará de disparar a classe correta do manipulador de tarefas. Para disparar tarefas na fila de empilhamento, basta usar o mesmo método `Queue::push` usado para filas convencionais.
