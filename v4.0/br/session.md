# Sessão

- [Configuração](#configuração)
- [Uso da sessão](#uso-da-sessão)
- [Dados Flash](#dados-flash)
- Sessões de banco de dados [Database Sessions](#database-sessions)
- [Drivers de sessão](#drivers-de-sessão)

<a name="configuração"></a>

## Configuração

Como as aplicações baseadas em HTTP são sem estado, as sessões fornecem uma maneira de armazenar informações sobre o usuário entre os pedidos. O Laravel vem com uma variedade de backends de sessão disponíveis para uso através de uma API limpa e unificada. O suporte para backends populares como [Memcached](http://memcached.org), [Redis](http://redis.io) e bancos de dados está incluído de fábrica.

A configuração da sessão é armazenada em `app/config/session.php`. Certifique-se de revisar as opções bem documentadas disponíveis neste arquivo. Por padrão, o Laravel está configurado para usar o driver de sessão `native`, que funcionará bem para a maioria das aplicações.

<a name="session-usage"></a>

## Uso da sessão

**Armazenando um Item na Sessão**

```
Session::put('key', 'value');
```

**Aplicar um valor a uma sessão de array**

```
Session::push('user.teams', 'developers');
```

**Recuperando um Item da Sessão**

```
$value = Session::get('key');
```

**Recuperando um Item ou Retornando um Valor Padrão**

```
$value = Session::get('key', 'default');

$value = Session::get('key', function() { return 'default'; });
```

**Recuperando todos os dados da sessão**

```
$data = Session::all();
```

**Determinando se um item existe na sessão**

```
if (Session::has('users'))
{
	//
}
```

**Remover um item da sessão**

```
Session::forget('key');
```

**Remover todos os itens da sessão**

```
Session::flush();
```

**Regenerando o ID de Sessão**

```
Session::regenerate();
```

<a name="flash-data"></a>

## Flash Data

Às vezes, você pode querer armazenar itens na sessão apenas para o próximo pedido. Você pode fazer isso usando o método `Session::flash`:

```
Session::flash('key', 'value');
```

**Recarregar os dados de flash atuais para outra solicitação**

```
Session::reflash();
```

**Reflashar é apenas um subconjunto dos dados do flash**

```
Session::keep(array('username', 'email'));
```

<a name="database-sessions"></a>

## Sessões de banco de dados

Ao usar o driver de sessão `database`, você precisará configurar uma tabela para conter os itens da sessão. Abaixo está uma declaração de `Schema` para a tabela:

```
Schema::create('sessions', function($table)
{
	$table->string('id')->unique();
	$table->text('payload');
	$table->integer('last_activity');
});
```

Claro, você pode usar o comando `session:table` do Artisan para gerar essa migração para você!

```
php artisan session:table

composer dump-autoload

php artisan migrate
```

<a name="session-drivers"></a>

## Motoristas de sessão

A sessão "driver" define onde os dados da sessão serão armazenados para cada solicitação. O Laravel vem com vários drivers ótimos prontos para uso:

- `native` - as sessões serão gerenciadas pelas instalações internas de sessão do PHP.
- `cookie` - as sessões serão armazenadas em cookies seguros e criptografados.
- `database` - as sessões serão armazenadas em um banco de dados utilizado pelo seu aplicativo.
- `memcached` / `redis` - as sessões serão armazenadas em uma dessas lojas rápidas e baseadas em cache.
- `array` - as sessões serão armazenadas em um simples array PHP e não serão persistidas entre as solicitações.

> **Nota:**
>
>  O driver de array é normalmente usado para executar testes unitários (/docs/testing), portanto, os dados da sessão não serão persistentes.
