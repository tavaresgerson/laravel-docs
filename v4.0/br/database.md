# Uso básico do banco de dados

- [Configuração](#configuração)
- [Executar consultas](#executar-consultas)
- [Transações de banco de dados](#transacoes-de-banco-de-dados)
- [Acesse as conexões](#acessando-conexoes)
- [Registro de consultas](#registro-de-consultas)

<a name="configuração"></a>

## Configuração

O Laravel torna a conexão com bancos de dados e a execução de consultas extremamente simples. O arquivo de configuração do banco de dados está em `app/config/database.php`. Neste arquivo, você pode definir todas as suas conexões de banco de dados, além de especificar qual conexão deve ser usada por padrão. Exemplos para todos os sistemas de banco de dados suportados estão fornecidos neste arquivo.

Atualmente, o Laravel suporta quatro sistemas de banco de dados: MySQL, Postgres, SQLite e SQL Server.

<a name="running-queries"></a>

## Executar consultas

Depois de configurar a conexão do banco de dados, você pode executar consultas usando a classe `DB`.

**Executando uma consulta seletiva**

```
$results = DB::select('select * from users where id = ?', array(1));
```

O método `select` sempre retornará um `array` de resultados.

**Executando uma declaração de inserção**

```
DB::insert('insert into users (id, name) values (?, ?)', array(1, 'Dayle'));
```

**Executando uma declaração de atualização**

```
DB::update('update users set votes = 100 where name = ?', array('John'));
```

**Executando uma declaração de exclusão**

```
DB::delete('delete from users');
```

> **Observação:**
>
>  As instruções 
>
> `update`
>
>  e 
>
> `delete`
>
>  retornam o número de linhas afetadas pela operação.

**Executando uma declaração geral**

```
DB::statement('drop table users');
```

Você pode ouvir eventos de consulta usando o método `DB::listen`:

**Ouvir para eventos de consulta**

```
DB::listen(function($sql, $bindings, $time)
{
	//
});
```

<a name="transações-do-banco-de-dados"></a>

## Transações de banco de dados

Para executar um conjunto de operações dentro de uma transação de banco de dados, você pode usar o método `transaction`:

```
DB::transaction(function()
{
	DB::table('users')->update(array('votes' => 1));

	DB::table('posts')->delete();
});
```

<a name="acessando-conexões"></a>

## Acesse Conexões

Ao usar múltiplas conexões, você pode acessá-las via o método `DB::connection`:

```
$users = DB::connection('foo')->select(...);
```

Você também pode acessar a instância PDO subjacente em bruto:

```
$pdo = DB::connection()->getPdo();
```

Às vezes, você pode precisar reconectar-se a um banco de dados específico:

```
DB::reconnect('foo');
```

<a name="query-logging"></a>

## Registro de consultas

Por padrão, o Laravel mantém um log na memória de todas as consultas executadas para o pedido atual. No entanto, em alguns casos, como ao inserir um grande número de linhas, isso pode fazer com que o aplicativo use memória em excesso. Para desabilitar o log, você pode usar o método `disableQueryLog`:

```
DB::connection()->disableQueryLog();
```

Para obter uma matriz das consultas executadas, você pode usar o método `getQueryLog`:

```
   $queries = DB::getQueryLog();
```
