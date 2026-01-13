# Construtor de consultas

- [Introdução](#introdução)
- [Seleciona](#seleciona)
- [Junte-se](#joins)
- [Advanced Wheres](#advanced-wheres)
- [Agregados](#agregados)
- Expressões Brutas [Raw Expressions](#raw-expressions)
- [Insere](#inserir)
- [Atualizações](#atualizações)
- [Exclui](#exclui)
- Uniões (#unions)
- [Cache de consultas](#cache-de-consultas)

<a name="introdução"></a>

## Introdução

O construtor de consultas de banco de dados oferece uma interface conveniente e fluida para criar e executar consultas de banco de dados. Ele pode ser usado para realizar a maioria das operações de banco de dados em sua aplicação e funciona em todos os sistemas de banco de dados suportados.

> **Nota:**
>
>  O construtor de consultas do Laravel usa a vinculação de parâmetros PDO em toda a aplicação para proteger seu aplicativo contra ataques de injeção SQL. Não há necessidade de limpar as strings que são passadas como vinculações.

<a name="seleciona"></a>

## Seleciona

**Recuperando todas as linhas de uma tabela**

```
$users = DB::table('users')->get();

foreach ($users as $user)
{
	var_dump($user->name);
}
```

**Recuperando uma única linha de uma tabela**

```
$user = DB::table('users')->where('name', 'John')->first();

var_dump($user->name);
```

**Recuperando uma única coluna de uma linha**

```
$name = DB::table('users')->where('name', 'John')->pluck('name');
```

**Recuperando uma lista de valores de coluna**

```
$roles = DB::table('roles')->lists('title');
```

Esse método retornará um array de títulos de papel. Você também pode especificar uma coluna de chave personalizada para o array retornado:

```
$roles = DB::table('roles')->lists('title', 'name');
```

**Especificando uma cláusula de seleção**

```
$users = DB::table('users')->select('name', 'email')->get();

$users = DB::table('users')->distinct()->get();

$users = DB::table('users')->select('name as user_name')->get();
```

**Adicionando uma cláusula de seleção a uma consulta existente**

```
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
```

**Usando Operadores Onde**

```
$users = DB::table('users')->where('votes', '>', 100)->get();
```

**Ou declarações**

```
$users = DB::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'John')
                    ->get();
```

**Usando Onde Entre**

```
$users = DB::table('users')
                    ->whereBetween('votes', array(1, 100))->get();
```

**Usando Where com um Array**

```
$users = DB::table('users')
                    ->whereIn('id', array(1, 2, 3))->get();

$users = DB::table('users')
                    ->whereNotIn('id', array(1, 2, 3))->get();
```

**Usando Where Null para encontrar registros com valores não definidos**

```
$users = DB::table('users')
                    ->whereNull('updated_at')->get();
```

**Ordenar por, Agrupar por e Ter**

```
$users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->groupBy('count')
                    ->having('count', '>', 100)
                    ->get();
```

**Deslocamento e Limite**

```
$users = DB::table('users')->skip(10)->take(5)->get();
```

<a name="liga-se"></a>

## Conexões

O construtor de consultas também pode ser usado para escrever instruções de junção. Veja os exemplos a seguir:

**Declaração de Conexão Básica**

```
DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.id', 'contacts.phone', 'orders.price');
```

**Instrução de Conjunção Esquerda**

```
DB::table('users')
	    ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
	    ->get();
```

Você também pode especificar cláusulas de junção mais avançadas:

```
DB::table('users')
        ->join('contacts', function($join)
        {
        	$join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();
```

<a name="advanced-wheres"></a>

## Advanced Wheres

Às vezes, você pode precisar criar cláusulas WHERE mais avançadas, como "where exists" ou agrupamentos de parâmetros aninhados. O construtor de consultas do Laravel também pode lidar com isso:

**Agrupamento de parâmetros**

```
DB::table('users')
            ->where('name', '=', 'John')
            ->orWhere(function($query)
            {
            	$query->where('votes', '>', 100)
            	      ->where('title', '<>', 'Admin');
            })
            ->get();
```

A consulta acima produzirá o seguinte SQL:

```
select * from users where name = 'John' or (votes > 100 and title <> 'Admin')
```

**Declarações existentes**

```
DB::table('users')
            ->whereExists(function($query)
            {
            	$query->select(DB::raw(1))
            	      ->from('orders')
            	      ->whereRaw('orders.user_id = users.id');
            })
            ->get();
```

A consulta acima produzirá o seguinte SQL:

```
select * from users
where exists (
	select 1 from orders where orders.user_id = users.id
)
```

<a name="agregados"></a>

## Agregados

O construtor de consultas também oferece uma variedade de métodos agregados, como `count`, `max`, `min`, `avg` e `sum`.

**Usando Métodos Agregados**

```
$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');

$price = DB::table('orders')->min('price');

$price = DB::table('orders')->avg('price');

$total = DB::table('users')->sum('votes');
```

<a name="expressões-bruta"></a>

## Expressões Bruxas

Às vezes, você pode precisar usar uma expressão bruta em uma consulta. Essas expressões serão inseridas na consulta como strings, então tenha cuidado para não criar pontos de injeção SQL! Para criar uma expressão bruta, você pode usar o método `DB::raw`:

**Usando uma expressão direta**

```
$users = DB::table('users')
                     ->select(DB::raw('count(*) as user_count, status'))
                     ->where('status', '<>', 1)
                     ->groupBy('status')
                     ->get();
```

**Aumentar ou diminuir um valor de uma coluna**

```
DB::table('users')->increment('votes');

DB::table('users')->increment('votes', 5);

DB::table('users')->decrement('votes');

DB::table('users')->decrement('votes', 5);
```

Você também pode especificar colunas adicionais para atualizar:

```
DB::table('users')->increment('votes', 1, array('name' => 'John'));
```

<a name="inserts"></a>

## Insertos

**Inserindo registros em uma tabela**

```
DB::table('users')->insert(
	array('email' => 'john@example.com', 'votes' => 0)
);
```

Se a tabela tiver um ID com autoincremento, use `insertGetId` para inserir um registro e recuperar o ID:

**Inserindo registros em uma tabela com um ID autoincrementado**

```
$id = DB::table('users')->insertGetId(
	array('email' => 'john@example.com', 'votes' => 0)
);
```

> **Observação:**
>
>  Ao usar o PostgreSQL, o método insertGetId espera que a coluna de autoincremento seja chamada de "id".

**Inserindo vários registros em uma tabela**

```
DB::table('users')->insert(array(
	array('email' => 'taylor@example.com', 'votes' => 0),
	array('email' => 'dayle@example.com', 'votes' => 0),
));
```

<a name="atualizações"></a>

## Atualizações

**Atualizando registros em uma tabela**

```
DB::table('users')
            ->where('id', 1)
            ->update(array('votes' => 1));
```

<a name="deleta"></a>

## Exclui

**Excluindo registros em uma tabela**

```
DB::table('users')->where('votes', '<', 100)->delete();
```

**Excluindo todos os registros de uma tabela**

```
DB::table('users')->delete();
```

**Cortando uma Tabela**

```
DB::table('users')->truncate();
```

<a name="sindicatos"></a>

## Sindicatos

O construtor de consultas também oferece uma maneira rápida de "unir" duas consultas:

**Executando uma união de consultas**

```
$first = DB::table('users')->whereNull('first_name');

$users = DB::table('users')->whereNull('last_name')->union($first)->get();
```

O método `unionAll` também está disponível e tem a mesma assinatura de método que `union`.

<a name="caching-queries"></a>

## Cacheamento de consultas

Você pode facilmente armazenar os resultados de uma consulta usando o método `remember`:

**Cacheamento de um Resultado de Consulta**

```
$users = DB::table('users')->remember(10)->get();
```

Neste exemplo, os resultados da consulta serão armazenados em cache por dez minutos. Enquanto os resultados estiverem em cache, a consulta não será executada no banco de dados e os resultados serão carregados a partir do driver de cache padrão especificado para sua aplicação.
