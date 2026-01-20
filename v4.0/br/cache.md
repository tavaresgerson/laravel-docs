# Cache

## Configuração

O Laravel fornece uma API unificada para vários sistemas de cache. A configuração do cache está localizada em `app/config/cache.php`. Neste arquivo, você pode especificar qual driver de cache você gostaria de usar por padrão em toda a sua aplicação. O Laravel suporta backends de cache populares como [Memcached](http://memcached.org) e [Redis](http://redis.io) prontos para uso.

O arquivo de configuração do cache também contém várias outras opções, que estão documentadas dentro do arquivo, então certifique-se de ler essas opções. Por padrão, o Laravel está configurado para usar o driver de cache `file`, que armazena os objetos serializados e cacheados no sistema de arquivos. Para aplicações maiores, recomenda-se que você use um cache de memória, como Memcached ou APC.

## Uso do cache

**Armazenando um Item no Cache**

```php
Cache::put('key', 'value', $minutes);
```

**Armazenando um Item no Cache se Ele Não Existir**

```php
Cache::add('key', 'value', $minutes);
```

**Verificar a existência no cache**

```php
if (Cache::has('key'))
{
	//
}
```

**Recuperando um Item do Cache**

```php
$value = Cache::get('key');
```

**Recuperando um Item ou Retornando um Valor Padrão**

```php
$value = Cache::get('key', 'default');

$value = Cache::get('key', function() { return 'default'; });
```

**Armazenando um Item no Cache Permanentemente**

```php
Cache::forever('key', 'value');
```

Às vezes, você pode querer recuperar um item do cache, mas também armazenar um valor padrão se o item solicitado não existir. Você pode fazer isso usando o método `Cache::remember`:

```php
$value = Cache::remember('users', $minutes, function()
{
	return DB::table('users')->get();
});
```

Você também pode combinar os métodos `remember` e `forever`:

```php
$value = Cache::rememberForever('users', function()
{
	return DB::table('users')->get();
});
```

Observe que todos os itens armazenados no cache são serializados, então você pode armazenar qualquer tipo de dado.

**Remover um item do cache**

```php
Cache::forget('key');
```

## Aumentos e Reduções

Todos os drivers, exceto `file` e `database`, suportam as operações `increment` e `decrement`:

**Aumentando um Valor**

```php
Cache::increment('key');

Cache::increment('key', $amount);
```

**Reduzindo um Valor**

```php
Cache::decrement('key');

Cache::decrement('key', $amount);
```

## Seções de cache

> **Observação:**
>
>  As seções de cache não são suportadas ao usar os controladores de cache 
>
> `file`
>
>  ou 
>
> `database`
>
> .

As seções de cache permitem agrupar itens relacionados no cache e, em seguida, limpar toda a seção. Para acessar uma seção, use o método `section`:

**Acessando uma seção de cache**

```
Cache::section('people')->put('John', $john, $minutes);

Cache::section('people')->put('Anne', $anne, $minutes);
```

Você também pode acessar itens armazenados em cache na seção, além de usar outros métodos de cache, como `increment` e `decrement`:

**Acessando itens em uma seção de cache**

```
$anne = Cache::section('people')->get('Anne');
```

Em seguida, você pode limpar todos os itens na seção:

```
Cache::section('people')->flush();
```

<a name="cache-de-banco-de-dados"></a>

## Cache de banco de dados

Ao usar o driver de cache `database`, você precisará configurar uma tabela para conter os itens de cache. Abaixo está uma declaração de `Schema` para a tabela:

```
Schema::create('cache', function($table)
{
	$table->string('key')->unique();
	$table->text('value');
	$table->integer('expiration');
});
```
