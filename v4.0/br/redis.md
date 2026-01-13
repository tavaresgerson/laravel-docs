# Redis

- [Introdução](#introdução)
- [Configuração](#configuração)
- [Uso](#uso)
- [Pipelining](#pipelining)

<a name="introdução"></a>

## Introdução

[Redis](http://redis.io) é um repositório de valores chave-valor avançado de código aberto. Ele é frequentemente referido como um servidor de estrutura de dados, pois as chaves podem conter [strings](http://redis.io/topics/data-types#strings), [hashes](http://redis.io/topics/data-types#hashes), [listas](http://redis.io/topics/data-types#lists), [conjuntos](http://redis.io/topics/data-types#sets) e [conjuntos ordenados](http://redis.io/topics/data-types#sorted-sets).

> **Observação:**
>
>  Se você tiver a extensão PHP Redis instalada via PECL, precisará renomear o alias do Redis no arquivo 
>
> `app/config/app.php`
>
> .

<a name="configuração"></a>

## Configuração

A configuração do Redis para sua aplicação é armazenada no arquivo **app/config/database.php**. Dentro deste arquivo, você verá um array **redis** contendo os servidores Redis usados pela sua aplicação:

```
'redis' => array(

	'cluster' => true,

	'default' => array('host' => '127.0.0.1', 'port' => 6379),

),
```

A configuração padrão do servidor deve ser suficiente para o desenvolvimento. No entanto, você pode modificar essa matriz de acordo com o seu ambiente. Basta atribuir um nome a cada servidor Redis e especificar o host e a porta usados pelo servidor.

A opção `cluster` informará o cliente Laravel Redis para realizar a fragmentação no lado do cliente em seus nós Redis, permitindo que você agrupe nós e crie uma grande quantidade de RAM disponível. No entanto, observe que a fragmentação no lado do cliente não gerencia o failover; portanto, é adequada principalmente para dados em cache que estão disponíveis em outro armazenamento de dados primário.

Se o seu servidor Redis exigir autenticação, você pode fornecer uma senha adicionando um par de chaves/valores `password` à sua matriz de configuração do servidor Redis.

<a name="uso"></a>

## Uso

Você pode obter uma instância do Redis chamando o método `Redis::connection`:

```
$redis = Redis::connection();
```

Isso lhe dará uma instância do servidor Redis padrão. Se você não estiver usando o agrupamento de servidores, pode passar o nome do servidor ao método `connection` para obter um servidor específico, conforme definido na configuração do Redis:

```
$redis = Redis::connection('other');
```

Depois de ter uma instância do cliente Redis, podemos emitir qualquer um dos [comandos Redis](http://redis.io/commands) à instância. O Laravel usa métodos mágicos para passar os comandos ao servidor Redis:

```
$redis->set('name', 'Taylor');

$name = $redis->get('name');

$values = $redis->lrange('names', 5, 10);
```

Observe que os argumentos do comando são simplesmente passados para o método mágico. Claro, você não precisa usar os métodos mágicos; você também pode passar comandos para o servidor usando o método `command`:

```
$values = $redis->command('lrange', array(5, 10));
```

Quando você está simplesmente executando comandos contra a conexão padrão, basta usar métodos mágicos estáticos na classe `Redis`:

```
Redis::set('name', 'Taylor');

$name = Redis::get('name');

$values = Redis::lrange('names', 5, 10);
```

> **Nota:**
>
>  Os drivers de cache 
>
> [Redis](/docs/cache)
>
>  e 
>
> [sessão](/docs/session)
>
>  estão incluídos no Laravel.

<a name="pipelining"></a>

## Pipelining

O pipelining deve ser usado quando você precisa enviar muitos comandos para o servidor em uma única operação. Para começar, use o comando `pipeline`:

**Pipeamento de Muitas Comandos para Seus Servidores**

```
Redis::pipeline(function($pipe)
{
	for ($i = 0; $i < 1000; $i++)
	{
		$pipe->set("key:$i", $i);
	}
});
```
