# Construtor de Esquema

- [Introdução](#introdução)
- [Criar e Remover Tabelas](#criando-e-removendo-tabelas)
- [Adicionar Colunas](#adicionando-colunas)
- Renomear Colunas (#renaming-columns)
- [Remover Colunas](#remover-colunas)
- [Verificar existência](#verificar-existência)
- [Adicionar índices](#adicionando-índices)
- [Chaves Estrangeiras](#chaves-estrangeiras)
- [Índice de queda](#índice-de-queda)
- Motores de Armazenamento (#storage-engines)

<a name="introdução"></a>

## Introdução

A classe `Schema` do Laravel oferece uma maneira de manipular tabelas que é independente do banco de dados. Ela funciona bem com todos os bancos de dados suportados pelo Laravel e possui uma API unificada em todos esses sistemas.

<a name="criando-e-excluindo-tabelas"></a>

## Criar e excluir tabelas

Para criar uma nova tabela de banco de dados, o método `Schema::create` é utilizado:

```
Schema::create('users', function($table)
{
	$table->increments('id');
});
```

O primeiro argumento passado para o método `create` é o nome da tabela, e o segundo é um `Closure` que receberá um objeto `Blueprint`, que pode ser usado para definir a nova tabela.

Para renomear uma tabela de banco de dados existente, o método `rename` pode ser usado:

```
Schema::rename($from, $to);
```

Para especificar em qual conexão a operação do esquema deve ocorrer, use o método `Schema::connection`:

```
Schema::connection('foo')->create('users', function($table)
{
	$table->increments('id');
});
```

Para descartar uma tabela, você pode usar o método `Schema::drop`:

```
Schema::drop('users');

Schema::dropIfExists('users');
```

<a name="adicionando-colunas"></a>

## Adicionar Colunas

Para atualizar uma tabela existente, usaremos o método `Schema::table`:

```
Schema::table('users', function($table)
{
	$table->string('email');
});
```

O construtor de tabelas contém uma variedade de tipos de colunas que você pode usar ao criar suas tabelas:

| Comando                                         | Descrição                                                          |
| ----------------------------------------------- | ------------------------------------------------------------------ |
| `$table->increments('id');`                     | Incrementar o ID para a tabela (chave primária).                   |
| `$table->bigIncrements('id');`                  | Incremento de ID usando um equivalente de "grande número inteiro". |
| `$table->string('email');`                      | Coluna equivalente a VARCHAR                                       |
| `$table->string('nome', 100);`                  | Equivalente VARCHAR com comprimento                                |
| `$table->integer('votos');`                     | INTEIRO equivalente à tabela                                       |
| `$table->bigInteger('votos');`                  | BIGINT equivalente à tabela                                        |
| `$table->smallInteger('votos');`                | SMALLINT equivalente à tabela                                      |
| `$table->float('valor');`                       | FLOAT equivalente à tabela                                         |
| `$table->double('column', 15, 8);`              | Equivalente a DOUBLE com precisão                                  |
| `$table->decimal('valor', 5, 2);`               | Equivalente decimal com precisão e escala                          |
| `$table->boolean('confirmado');`                | É equivalente a uma tabela BOOLEAN                                 |
| `$tabela->data('created_at');`                  | DATA equivalente à tabela                                          |
| `$table->dateTime('created_at');`               | DATETIME equivalente à tabela                                      |
| `$table->time('nascer do sol');`                | TEMPO equivalente à tabela                                         |
| `$table->timestamp('added_on');`                | TEMPO DE GRAVURA equivalente à tabela                              |
| `$table->timestamps();`                         | Adiciona as colunas **created\_at** e **updated\_at**              |
| `$table->softDeletes();`                        | Adiciona a coluna **deleted\_at** para apagamentos suaves          |
| `$table->text('descrição');`                    | TEXTO equivalente à tabela                                         |
| `$table->binary('data');`                       | BLOB equivalente à tabela                                          |
| `$table->enum('choices', array('foo', 'bar'));` | ENUM equivalente à tabela                                          |
| `->nullable()`                                  | Designe que a coluna permita valores NULL                          |
| `->default($value)`                             | Declarar um valor padrão para uma coluna                           |
| `->unsigned()`                                  | Defina INTEGER para UNSIGNED                                       |

Se você estiver usando o banco de dados MySQL, pode usar o método `after` para especificar a ordem das colunas:

**Usando After On no MySQL**

```
$table->string('name')->after('email');
```

<a name="renaming-columns"></a>

## Renomear Colunas

Para renomear uma coluna, você pode usar o método `renameColumn` no construtor de Schema:

**Renomear uma Coluna**

```
Schema::table('users', function($table)
{
	$table->renameColumn('from', 'to');
});
```

> **Observação:**
>
>  O renomeamento dos tipos de colunas 
>
> `enum`
>
>  não é suportado.

<a name="colunas-a-cair"></a>

## Colunas em queda

**Excluindo uma coluna de uma tabela de banco de dados**

```
Schema::table('users', function($table)
{
	$table->dropColumn('votes');
});
```

**Excluindo Múltiplos Colunas de uma Tabela de Banco de Dados**

```
Schema::table('users', function($table)
{
	$table->dropColumn('votes', 'avatar', 'location');
});
```

<a name="verificando-a-existência"></a>

## Verificar a existência

Você pode facilmente verificar a existência de uma tabela ou coluna usando os métodos `hasTable` e `hasColumn`:

**Verificando a existência de uma tabela**

```
if (Schema::hasTable('users'))
{
	//
}
```

**Verificação da Existência de Colunas**

```
if (Schema::hasColumn('users', 'email'))
{
	//
}
```

<a name="adicionando-índices"></a>

## Adicionar índices

O construtor de esquemas suporta vários tipos de índices. Existem duas maneiras de adicioná-los. Primeiro, você pode defini-los de forma fluente em uma definição de coluna, ou você pode adicioná-los separadamente:

**Criando uma coluna e um índice com fluidez**

```
$table->string('email')->unique();
```

Ou você pode optar por adicionar os índices em linhas separadas. Abaixo está uma lista de todos os tipos de índice disponíveis:

| Comando                                         | Descrição                     |
| ----------------------------------------------- | ----------------------------- |
| `$table->primary('id');`                        | Adicionar uma chave primária  |
| `$table->primary(array('primeiro', 'ultimo'));` | Adicionar chaves compostas    |
| `$table->unique('email');`                      | Adicionar um índice exclusivo |
| `$table->index('estado');`                      | Adicionar um índice básico    |

<a name="foreign-keys"></a>

## Chaves Estrangeiras

O Laravel também oferece suporte para adicionar restrições de chave estrangeira às suas tabelas:

**Adicionando uma chave estrangeira a uma tabela**

```
$table->foreign('user_id')->references('id')->on('users');
```

Neste exemplo, estamos afirmando que a coluna `user_id` faz referência à coluna `id` na tabela `users`.

Você também pode especificar opções para as ações "ao excluir" e "ao atualizar" da restrição:

```
$table->foreign('user_id')
      ->references('id')->on('users')
      ->onDelete('cascade');
```

Para descartar uma chave estrangeira, você pode usar o método `dropForeign`. Uma convenção de nomenclatura semelhante é usada para chaves estrangeiras, assim como para outros índices:

```
$table->dropForeign('posts_user_id_foreign');
```

> **Observação:**
>
>  Ao criar uma chave estrangeira que faz referência a um inteiro incrementado, lembre-se de sempre tornar a coluna da chave estrangeira como 
>
> `unsigned`
>
> .

<a name="dropping-indexes"></a>

## Índice de queda

Para excluir um índice, você deve especificar o nome do índice. O Laravel atribui um nome razoável aos índices por padrão. Basta concatenar o nome da tabela, os nomes das colunas no índice e o tipo do índice. Aqui estão alguns exemplos:

| Comando                                     | Descrição                                    |
| ------------------------------------------- | -------------------------------------------- |
| `$table->dropPrimary('users_id_primary');`  | Remover uma chave primária da tabela "users" |
| `$table->dropUnique('users_email_unique');` | Remover um índice único da tabela "users"    |
| `$table->dropIndex('geo_state_index');`     | Remover um índice básico da tabela "geo"     |

<a name="storage-engines"></a>

## Motores de Armazenamento

Para definir o mecanismo de armazenamento para uma tabela, defina a propriedade `engine` no construtor do esquema:

```
Schema::create('users', function($table)
{
    $table->engine = 'InnoDB';

    $table->string('email');
});
```
