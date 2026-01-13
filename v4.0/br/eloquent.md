# ORM eloquente

- [Introdução](#introdução)
- [Uso básico](#uso-básico)
- [Atribuição em massa](#atribuicao-em-massa)
- [Inserir, Atualizar, Deletar](#inserir-atualizar-deletar)
- [Excluir Suave](#excluir-suave)
- [Timestamps](#timestamps)
- [Ângulos de consulta](#angulos-de-consulta)
- Relações [Relações](#relações)
- [Consultando Relações](#consultando-relações)
- [Carregamento Agressivo](#carregamento-agressivo)
- [Inserir Modelos Relacionados](#inserir-modelos-relacionados)
- [Tocando em marcadores de tempo dos pais](#tocando-em-marcadores-de-tempo-dos-pais)
- [Trabalhando com Tabelas Dinâmicas](#trabalhando-com-tabelas-dinâmicas)
- Coleções [Coleções](#collections)
- [Acessadores e Mutáveis](#acessadores-e-mutáveis)
- [Mutadores de data](#mutaores-de-data)
- Eventos do modelo]\(#model-events)
- [Observadores do Modelo](#model-observers)
- [Conversão para Arrays/JSON](#conversao-para-arrays-ou-json)

<a name="introdução"></a>

## Introdução

O Eloquent ORM incluído no Laravel oferece uma implementação ActiveRecord bonita e simples para trabalhar com seu banco de dados. Cada tabela do banco de dados tem um "Modelo" correspondente, que é usado para interagir com essa tabela.

Antes de começar, certifique-se de configurar uma conexão de banco de dados em `app/config/database.php`.

<a name="uso-básico"></a>

## Uso básico

Para começar, crie um modelo Eloquent. Os modelos geralmente vivem no diretório `app/models`, mas você pode colocá-los em qualquer lugar que possa ser carregado automaticamente de acordo com seu arquivo `composer.json`.

**Definindo um Modelo Eloquente**

```
class User extends Eloquent {}
```

Observe que não informamos ao Eloquent qual tabela usar para nosso modelo `User`. O nome da classe em minúsculas e plural será usado como nome da tabela, a menos que outro nome seja especificado explicitamente. Portanto, neste caso, o Eloquent assumirá que o modelo `User` armazena registros na tabela `users`. Você pode especificar uma tabela personalizada definindo uma propriedade `table` no seu modelo:

```
class User extends Eloquent {

	protected $table = 'my_users';

}
```

> **Nota:**
>
>  O Eloquent também assumirá que cada tabela tem uma coluna de chave primária chamada 
>
> `id`
>
> . Você pode definir uma propriedade 
>
> `primaryKey`
>
>  para sobrescrever essa convenção. Da mesma forma, você pode definir uma propriedade 
>
> `connection`
>
>  para sobrescrever o nome da conexão de banco de dados que deve ser usada ao utilizar o modelo.

Depois que um modelo é definido, você está pronto para começar a recuperar e criar registros em sua tabela. Observe que você precisará colocar as colunas `updated_at` e `created_at` na sua tabela por padrão. Se você não quiser que essas colunas sejam mantidas automaticamente, defina a propriedade `$timestamps` no seu modelo para `false`.

**Recuperando Todos os Modelos**

```
$users = User::all();
```

**Recuperando um Registro Pela Chave Primária**

```
$user = User::find(1);

var_dump($user->name);
```

> **Nota:**
>
>  Todos os métodos disponíveis no 
>
> [construtor de consultas](/docs/queries)
>
>  também estão disponíveis ao consultar modelos Eloquent.

**Recuperando um modelo por chave primária ou lançando uma exceção**

Às vezes, você pode querer lançar uma exceção se um modelo não for encontrado, permitindo que você capture as exceções usando um manipulador `App::error` e exiba uma página 404.

```
$model = User::findOrFail(1);

$model = User::where('votes', '>', 100)->firstOrFail();
```

Para registrar o manipulador de erros, ouça a `ModelNotFoundException`

```
use Illuminate\Database\Eloquent\ModelNotFoundException;

App::error(function(ModelNotFoundException $e)
{
	return Response::make('Not Found', 404);
});
```

**Consultando com Modelos Eloquent**

```
$users = User::where('votes', '>', 100)->take(10)->get();

foreach ($users as $user)
{
	var_dump($user->name);
}
```

Claro, você também pode usar as funções agregadas do construtor de consultas.

**Agregados Eloquentes**

```
$count = User::where('votes', '>', 100)->count();
```

Se você não conseguir gerar a consulta necessária através da interface fluente, sinta-se à vontade para usar `whereRaw`:

```
$users = User::whereRaw('age > ? and votes = 100', array(25))->get();
```

**Especificando a Conexão da Consulta**

Você também pode especificar qual conexão de banco de dados deve ser usada ao executar uma consulta Eloquent. Basta usar o método `on`:

```
$user = User::on('connection-name')->find(1);
```

<a name="mass-assignment"></a>

## Atribuição em massa

Ao criar um novo modelo, você passa um array de atributos para o construtor do modelo. Esses atributos são então atribuídos ao modelo por meio de atribuição em massa. Isso é conveniente; no entanto, pode ser uma **séria** preocupação de segurança quando a entrada do usuário é passada cegamente para um modelo. Se a entrada do usuário for passada cegamente para um modelo, o usuário é livre para modificar **qualquer** e **todos** os atributos do modelo. Por essa razão, todos os modelos Eloquent protegem contra a atribuição em massa por padrão.

Para começar, defina as propriedades `fillable` ou `guarded` no seu modelo.

A propriedade `fillable` especifica quais atributos devem ser atribuídos em massa. Isso pode ser definido no nível de classe ou de instância.

**Definindo Atributos Preenchíveis em um Modelo**

```
class User extends Eloquent {

	protected $fillable = array('first_name', 'last_name', 'email');

}
```

Neste exemplo, apenas os três atributos listados serão atribuídos em massa.

O inverso de `fillable` é `guarded`, e serve como uma "lista negra" em vez de uma "lista branca":

**Definindo Atributos Protegidos em um Modelo**

```
class User extends Eloquent {

	protected $guarded = array('id', 'password');

}
```

No exemplo acima, os atributos `id` e `password` **não podem** ser atribuídos em massa. Todos os outros atributos poderão ser atribuídos em massa. Você também pode bloquear **todos** os atributos da atribuição em massa usando o método de guarda:

**Bloquear todos os atributos da atribuição em massa**

```
protected $guarded = array('*');
```

<a name="inserir-atualizar-deletar"></a>

## Inserir, Atualizar, Deletar

Para criar um novo registro no banco de dados a partir de um modelo, basta criar uma nova instância do modelo e chamar o método `save`.

**Salvar um novo modelo**

```
$user = new User;

$user->name = 'John';

$user->save();
```

> **Observação:**
>
>  Normalmente, seus modelos Eloquent terão chaves de autoincremento. No entanto, se você deseja especificar suas próprias chaves, defina a propriedade 
>
> `incrementing`
>
>  no seu modelo para 
>
> `false`
>
> .

Você também pode usar o método `create` para salvar um novo modelo em uma única linha. A instância do modelo inserida será devolvida para você pelo método. No entanto, antes de fazer isso, você precisará especificar um atributo `fillable` ou `guarded` no modelo, pois todos os modelos Eloquent protegem contra a associação em massa.

Depois de salvar ou criar um novo modelo que utiliza IDs com autoincremento, você pode recuperar o ID acessando o atributo `id` do objeto:

```
$insertedId = $user->id;
```

**Definindo os atributos protegidos no modelo**

```
class User extends Eloquent {

	protected $guarded = array('id', 'account_id');

}
```

**Usando o Método de Criação do Modelo**

```
$user = User::create(array('name' => 'John'));
```

Para atualizar um modelo, você pode recuperá-lo, alterar um atributo e usar o método `save`:

**Atualizando um Modelo Recuperado**

```
$user = User::find(1);

$user->email = 'john@foo.com';

$user->save();
```

Às vezes, você pode querer salvar não apenas um modelo, mas também todas as suas relações. Para fazer isso, você pode usar o método `push`:

**Salvar um modelo e relacionamentos**

```
$user->push();
```

Você também pode executar atualizações como consultas em um conjunto de modelos:

```
$affectedRows = User::where('votes', '>', 100)->update(array('status' => 2));
```

Para excluir um modelo, basta chamar o método `delete` na instância:

**Excluindo um Modelo Existente**

```
$user = User::find(1);

$user->delete();
```

**Excluindo um Modelo Existente por Chave**

```
User::destroy(1);

User::destroy(array(1, 2, 3));

User::destroy(1, 2, 3);
```

Claro, você também pode executar uma consulta de exclusão em um conjunto de modelos:

```
$affectedRows = User::where('votes', '>', 100)->delete();
```

Se você deseja simplesmente atualizar os timestamps de um modelo, você pode usar o método `touch`:

**Atualizando apenas os timestamps do modelo**

```
$user->touch();
```

<a name="soft-deleting"></a>

## Apagar com suavidade

Ao excluir um modelo de forma suave, ele não é realmente removido do seu banco de dados. Em vez disso, um timestamp `deleted_at` é definido no registro. Para habilitar exclusões sutis para um modelo, especifique a propriedade `softDelete` no modelo:

```
class User extends Eloquent {

	protected $softDelete = true;

}
```

Para adicionar uma coluna `deleted_at` à sua tabela, você pode usar o método `softDeletes` de uma migração:

```
$table->softDeletes();
```

Agora, quando você chamar o método `delete` no modelo, a coluna `deleted_at` será definida pelo timestamp atual. Ao consultar um modelo que usa apagamentos suaves, os modelos "apaga" não serão incluídos nos resultados da consulta. Para forçar que os modelos apagados apareçam em um conjunto de resultados, use o método `withTrashed` na consulta:

**Forçando Modelos Aparentemente Excluídos a Aparecer nos Resultados**

```
$users = User::withTrashed()->where('account_id', 1)->get();
```

Se você deseja **apenas** receber modelos excluídos temporariamente em seus resultados, você pode usar o método `onlyTrashed`:

```
$users = User::onlyTrashed()->where('account_id', 1)->get();
```

Para restaurar um modelo excluído com suavidade para um estado ativo, use o método `restore`:

```
$user->restore();
```

Você também pode usar o método `restore` em uma consulta:

```
User::withTrashed()->where('account_id', 1)->restore();
```

O método `restore` também pode ser usado em relacionamentos:

```
$user->posts()->restore();
```

Se você deseja realmente remover um modelo do banco de dados, você pode usar o método `forceDelete`:

```
$user->forceDelete();
```

O método `forceDelete` também funciona em relacionamentos:

```
$user->posts()->forceDelete();
```

Para determinar se uma instância de modelo foi excluída temporariamente, você pode usar o método `trashed`:

```
if ($user->trashed())
{
	//
}
```

<a name="timestamps"></a>

## Datas e horários

Por padrão, o Eloquent manterá as colunas `created_at` e `updated_at` na sua tabela do banco de dados automaticamente. Basta adicionar essas colunas `timestamp` à sua tabela e o Eloquent cuidará do resto. Se você não quiser que o Eloquent mantenha essas colunas, adicione a seguinte propriedade ao seu modelo:

**Desativando Cronometragens Automáticas**

```
class User extends Eloquent {

	protected $table = 'users';

	public $timestamps = false;

}
```

Se você deseja personalizar o formato dos seus timestamps, você pode sobrescrever o método `getDateFormat` em seu modelo:

**Fornecer um formato de marcação de tempo personalizado**

```
class User extends Eloquent {

	protected function getDateFormat()
	{
		return 'U';
	}

}
```

<a name="query-scopes"></a>

## Ângulos de consulta

Os escopos permitem que você reutilize facilmente a lógica de consulta em seus modelos. Para definir um escopo, basta prefixar um método de modelo com `scope`:

**Definindo o escopo de uma consulta**

```
class User extends Eloquent {

	public function scopePopular($query)
	{
		return $query->where('votes', '>', 100);
	}

	public function scopeWomen($query)
	{
		return $query->whereGender('W');
	}

}
```

**Utilizando um escopo de consulta**

```
$users = User::popular()->women()->orderBy('created_at')->get();
```

**Ângulos Dinâmicos**

Às vezes, você pode querer definir um escopo que aceite parâmetros. Basta adicionar seus parâmetros à sua função de escopo:

```
class User extends Eloquent {

	public function scopeOfType($query, $type)
	{
		return $query->whereType($type);
	}

}
```

Em seguida, passe o parâmetro para a chamada de escopo:

```
$users = User::ofType('member')->get();
```

<a name="relacionamentos"></a>

## Relações

Claro, suas tabelas de banco de dados provavelmente estão relacionadas entre si. Por exemplo, um post de blog pode ter muitos comentários, ou uma ordem pode estar relacionada ao usuário que a fez. O Eloquent facilita a gestão e o trabalho com essas relações. O Laravel suporta quatro tipos de relações:

- [One To One](#one-to-one)
- [Um para muitos](#um-para-muitos)
- [Muitos para muitos](#muitos-para-muitos)
- Relações Policrônicas (#relações\_policrônicas)

<a name="um-para-um"> </a>

### One To One

Uma relação um-para-um é uma relação muito básica. Por exemplo, um modelo `User` pode ter um `Phone`. Podemos definir essa relação no Eloquent:

**Definindo uma Relação Um-para-Um**

```
class User extends Eloquent {

	public function phone()
	{
		return $this->hasOne('Phone');
	}

}
```

O primeiro argumento passado para o método `hasOne` é o nome do modelo relacionado. Uma vez que a relação é definida, podemos recuperá-la usando as propriedades dinâmicas do Eloquent:

```
$phone = User::find(1)->phone;
```

O SQL executado por esta declaração será o seguinte:

```
select * from users where id = 1

select * from phones where user_id = 1
```

Tenha em mente que o Eloquent assume a chave estrangeira da relação com base no nome do modelo. Neste caso, o modelo `Phone` é suposto usar uma chave estrangeira `user_id`. Se você deseja sobrescrever essa convenção, pode passar um segundo argumento para o método `hasOne`:

```
return $this->hasOne('Phone', 'custom_key');
```

Para definir a inversão da relação no modelo `Phone`, usamos o método `belongsTo`:

**Definindo o Inverso de uma Relação**

```
class Phone extends Eloquent {

	public function user()
	{
		return $this->belongsTo('User');
	}

}
```

No exemplo acima, o Eloquent procurará uma coluna `user_id` na tabela `phones`. Se você deseja definir uma coluna de chave estrangeira diferente, você pode passá-la como o segundo argumento para o método `belongsTo`:

```
class Phone extends Eloquent {

	public function user()
	{
		return $this->belongsTo('User', 'custom_key');
	}

}
```

<a name="um-para-muitos"></a>

### Um para muitos

Um exemplo de uma relação de um para muitos é um post de blog que "tem muitos" comentários. Podemos modelar essa relação da seguinte forma:

```
class Post extends Eloquent {

	public function comments()
	{
		return $this->hasMany('Comment');
	}

}
```

Agora podemos acessar os comentários do post através da propriedade dinâmica [dynamic property](#dynamic-properties):

```
$comments = Post::find(1)->comments;
```

Se você precisar adicionar mais restrições para os comentários que serão recuperados, você pode chamar o método `comments` e continuar concatenando condições:

```
$comments = Post::find(1)->comments()->where('title', '=', 'foo')->first();
```

Novamente, você pode substituir a chave estrangeira convencional passando um segundo argumento para o método `hasMany`:

```
return $this->hasMany('Comment', 'custom_key');
```

Para definir a inversão da relação no modelo `Comment`, usamos o método `belongsTo`:

**Definindo o Inverso de uma Relação**

```
class Comment extends Eloquent {

	public function post()
	{
		return $this->belongsTo('Post');
	}

}
```

<a name="many-to-many"></a>

### Muitos para muitos

As relações muitos-para-muitos são um tipo de relação mais complexo. Um exemplo dessa relação é um usuário com muitos papéis, onde os papéis também são compartilhados por outros usuários. Por exemplo, muitos usuários podem ter o papel de "Administrador". Três tabelas de banco de dados são necessárias para essa relação: `users`, `roles` e `role_user`. A tabela `role_user` é derivada da ordem alfabética dos nomes dos modelos relacionados e deve ter as colunas `user_id` e `role_id`.

Podemos definir uma relação muitos-para-muitos usando o método `belongsToMany`:

```
class User extends Eloquent {

	public function roles()
	{
		return $this->belongsToMany('Role');
	}

}
```

Agora, podemos recuperar os papéis através do modelo `User`:

```
$roles = User::find(1)->roles;
```

Se você deseja usar um nome de tabela não convencional para sua tabela de pivô, você pode passá-lo como o segundo argumento para o método `belongsToMany`:

```
return $this->belongsToMany('Role', 'user_roles');
```

Você também pode substituir as chaves associadas convencionais:

```
return $this->belongsToMany('Role', 'user_roles', 'user_id', 'foo_id');
```

Claro, você também pode definir o inverso da relação no modelo `Role`:

```
class Role extends Eloquent {

	public function users()
	{
		return $this->belongsToMany('User');
	}

}
```

<a name="relações_polimórficas"></a>

### Relações Polimórficas

As relações polimórficas permitem que um modelo pertença a mais de um outro modelo, em uma única associação. Por exemplo, você pode ter um modelo de foto que pertence a um modelo de equipe ou a um modelo de pedido. Definiríamos essa relação da seguinte forma:

```
class Photo extends Eloquent {

	public function imageable()
	{
		return $this->morphTo();
	}

}

class Staff extends Eloquent {

	public function photos()
	{
		return $this->morphMany('Photo', 'imageable');
	}

}

class Order extends Eloquent {

	public function photos()
	{
		return $this->morphMany('Photo', 'imageable');
	}

}
```

Agora, podemos recuperar as fotos de um funcionário ou de um pedido:

**Recuperando uma Relação Polimórfica**

```
$staff = Staff::find(1);

foreach ($staff->photos as $photo)
{
	//
}
```

No entanto, a verdadeira magia "polimórfica" ocorre quando você acessa o staff ou a ordem do modelo `Photo`:

**Recuperando o Proprietário de uma Relação Polimórfica**

```
$photo = Photo::find(1);

$imageable = $photo->imageable;
```

A relação `imageable` no modelo `Photo` retornará uma instância de `Staff` ou `Order`, dependendo do tipo de modelo que possui a foto.

Para ajudar a entender como isso funciona, vamos explorar a estrutura do banco de dados para uma relação polimórfica:

**Estrutura da Tabela de Relação Polimórfica**

```
staff
	id - integer
	name - string

orders
	id - integer
	price - integer

photos
	id - integer
	path - string
	imageable_id - integer
	imageable_type - string
```

Os campos principais a serem observados aqui são `imageable_id` e `imageable_type` na tabela `photos`. O ID conterá o valor do ID, neste exemplo, do funcionário ou da ordem proprietária, enquanto o tipo conterá o nome da classe do modelo proprietário. Isso permite que o ORM determine qual tipo de modelo proprietário deve ser retornado ao acessar a relação `imageable`.

<a name="consultando-relações"></a>

## Consultando Relações

Ao acessar os registros de um modelo, você pode querer limitar seus resultados com base na existência de uma relação. Por exemplo, você deseja extrair todas as postagens de blog que tenham pelo menos um comentário. Para fazer isso, você pode usar o método `has`:

**Verificando relações ao selecionar**

```
$posts = Post::has('comments')->get();
```

Você também pode especificar um operador e um número:

```
$posts = Post::has('comments', '>=', 3)->get();
```

<a name="propriedades_dinâmicas"></a>

### Propriedades Dinâmicas

O Eloquent permite que você acesse suas relações por meio de propriedades dinâmicas. O Eloquent carregará automaticamente a relação para você e será inteligente o suficiente para saber se deve chamar o método `get` (para relações um-para-muitos) ou `first` (para relações um-para-um). Ele então será acessível por meio de uma propriedade dinâmica com o mesmo nome da relação. Por exemplo, com o seguinte modelo `$phone`:

```
class Phone extends Eloquent {

	public function user()
	{
		return $this->belongsTo('User');
	}

}

$phone = Phone::find(1);
```

Em vez de ecoar o e-mail do usuário assim:

```
echo $phone->user()->first()->email;
```

Pode ser resumido para simplesmente:

```
echo $phone->user->email;
```

> **Observação:**
>
>  Relacionamentos que retornam muitos resultados retornarão uma instância da classe 
>
> `Illuminate\Database\Eloquent\Collection`
>
> .

<a name="eager-loading"></a>

## Carregamento ansioso

O carregamento ágil existe para aliviar o problema da consulta N + 1. Por exemplo, considere um modelo `Livro` que está relacionado a `Autor`. A relação é definida da seguinte forma:

```
class Book extends Eloquent {

	public function author()
	{
		return $this->belongsTo('Author');
	}

}
```

Agora, considere o seguinte código:

```
foreach (Book::all() as $book)
{
	echo $book->author->name;
}
```

Esse loop executará uma consulta para recuperar todos os livros da tabela, e depois outra consulta para cada livro para recuperar o autor. Portanto, se tivermos 25 livros, esse loop executará 26 consultas.

Felizmente, podemos usar o carregamento ansioso para reduzir drasticamente o número de consultas. As relações que devem ser carregadas ansiosamente podem ser especificadas via o método `with`:

```
foreach (Book::with('author')->get() as $book)
{
	echo $book->author->name;
}
```

No loop acima, apenas duas consultas serão executadas:

```
select * from books

select * from authors where id in (1, 2, 3, 4, 5, ...)
```

O uso inteligente do carregamento ansioso pode aumentar drasticamente o desempenho do seu aplicativo.

Claro, você pode carregar vários relacionamentos de uma só vez:

```
$books = Book::with('author', 'publisher')->get();
```

Você pode até carregar com entusiasmo as relações aninhadas:

```
$books = Book::with('author.contacts')->get();
```

No exemplo acima, a relação `author` será carregada com agilidade, e a relação `contacts` do autor também será carregada.

### Restrições de Carregamento Precoce

Às vezes, você pode querer carregar um relacionamento com urgência, mas também especificar uma condição para o carregamento com urgência. Aqui está um exemplo:

```
$users = User::with(array('posts' => function($query)
{
	$query->where('title', 'like', '%first%');
}))->get();
```

Neste exemplo, estamos carregando ansiosamente as postagens do usuário, mas apenas se a coluna "Título" da postagem contiver a palavra "primeiro".

### Carregamento Preocupante Preguiçoso

Também é possível carregar modelos relacionados diretamente de uma coleção de modelos já existente. Isso pode ser útil quando você decide dinamicamente se deve ou não carregar modelos relacionados, ou em combinação com o cache.

```
$books = Book::all();

$books->load('author', 'publisher');
```

<a name="inserindo-modelos-relacionados"></a>

## Inserindo Modelos Relacionados

Você precisará frequentemente inserir novos modelos relacionados. Por exemplo, você pode querer inserir um novo comentário para um post. Em vez de definir manualmente a chave estrangeira `post_id` no modelo, você pode inserir o novo comentário diretamente do modelo `Post` pai:

**Anexar um modelo relacionado**

```
$comment = new Comment(array('message' => 'A new comment.'));

$post = Post::find(1);

$comment = $post->comments()->save($comment);
```

Neste exemplo, o campo `post_id` será definido automaticamente no comentário inserido.

### Associar Modelos (Pertence a)

Ao atualizar uma relação `belongsTo`, você pode usar o método `associate`. Esse método definirá a chave estrangeira no modelo filho:

```
$account = Account::find(10);

$user->account()->associate($account);

$user->save();
```

### Inserindo Modelos Relacionados (Muitos para Muitos)

Você também pode inserir modelos relacionados ao trabalhar com relações muitos-para-muitos. Vamos continuar usando nossos modelos `User` e `Role` como exemplos. Podemos facilmente anexar novos papéis a um usuário usando o método `attach`:

**Atribuindo muitos a muitos modelos**

```
$user = User::find(1);

$user->roles()->attach(1);
```

Você também pode passar um array de atributos que devem ser armazenados na tabela pivot para a relação:

```
$user->roles()->attach(1, array('expires' => $expires));
```

Claro, o oposto de "attach" é "detach":

```
$user->roles()->detach(1);
```

Você também pode usar o método `sync` para vincular modelos relacionados. O método `sync` aceita um array de IDs para colocar na tabela de pivô. Após essa operação ser concluída, apenas os IDs no array estarão na tabela intermediária para o modelo:

**Usando Sincronização para Vincular Modelos Muitos-Para-Muitos**

```
$user->roles()->sync(array(1, 2, 3));
```

Você também pode associar outros valores da tabela dinâmica com os IDs fornecidos:

**Adicionar dados de pivô ao sincronizar**

```
$user->roles()->sync(array(1 => array('expires' => true)));
```

Às vezes, você pode querer criar um novo modelo relacionado e anexá-lo em um único comando. Para essa operação, você pode usar o método `save`:

```
$role = new Role(array('name' => 'Editor'));

User::find(1)->roles()->save($role);
```

Neste exemplo, o novo modelo `Role` será salvo e anexado ao modelo de usuário. Você também pode passar um array de atributos para colocar na tabela de junção para essa operação:

```
User::find(1)->roles()->save($role, array('expires' => $expires));
```

<a name="touching-parent-timestamps"></a>

## Marcações de tempo dos pais tocando

Quando um modelo `belongsTo` outro modelo, como um `Comment` que pertence a um `Post`, muitas vezes é útil atualizar o timestamp do pai quando o modelo filho é atualizado. Por exemplo, quando um modelo `Comment` é atualizado, você pode querer tocar automaticamente o timestamp `updated_at` do `Post` proprietário. O Eloquent facilita isso. Basta adicionar uma propriedade `touches` contendo os nomes das relações para o modelo filho:

```
class Comment extends Eloquent {

	protected $touches = array('post');

	public function post()
	{
		return $this->belongsTo('Post');
	}

}
```

Agora, ao atualizar um `Comentário`, o `Post` proprietário terá sua coluna `updated_at` atualizada:

```
$comment = Comment::find(1);

$comment->text = 'Edit to this comment!';

$comment->save();
```

<a name="trabalhando-com-tabelas-pivô"></a>

## Trabalhando com Tabelas Dinâmicas

Como você já aprendeu, trabalhar com relações muitos-para-muitos requer a presença de uma tabela intermediária. O Eloquent oferece algumas maneiras muito úteis de interagir com essa tabela. Por exemplo, vamos assumir que nosso objeto `User` tem muitos objetos `Role` com os quais está relacionado. Após acessar essa relação, podemos acessar a tabela `pivot` nos modelos:

```
$user = User::find(1);

foreach ($user->roles as $role)
{
	echo $role->pivot->created_at;
}
```

Observe que cada modelo `Role` que recuperamos é automaticamente atribuído um atributo `pivot`. Esse atributo contém um modelo que representa a tabela intermediária e pode ser usado como qualquer outro modelo Eloquent.

Por padrão, apenas as chaves estarão presentes no objeto `pivot`. Se sua tabela pivot contiver atributos extras, você deve especificá-los ao definir a relação:

```
return $this->belongsToMany('Role')->withPivot('foo', 'bar');
```

Agora, os atributos `foo` e `bar` estarão acessíveis no nosso objeto `pivot` para o modelo `Role`.

Se você deseja que sua tabela pivot tenha os timestamps `created_at` e `updated_at` mantidos automaticamente, use o método `withTimestamps` na definição da relação:

```
return $this->belongsToMany('Role')->withTimestamps();
```

Para excluir todos os registros da tabela dinâmica de um modelo, você pode usar o método `detach`:

**Excluindo registros em uma tabela dinâmica**

```
User::find(1)->roles()->detach();
```

Observe que essa operação não exclui registros da tabela `roles`, mas apenas da tabela de pivô.

<a name="coleções"></a>

## Coleções

Todos os conjuntos de resultados múltiplos retornados pelo Eloquent, seja pelo método `get` ou por uma `relação`, retornarão um objeto de coleção. Esse objeto implementa a interface `IteratorAggregate` do PHP, para que possa ser iterado como uma matriz. No entanto, esse objeto também possui uma variedade de outros métodos úteis para trabalhar com conjuntos de resultados.

Por exemplo, podemos determinar se um conjunto de resultados contém uma chave primária específica usando o método `contains`:

**Verificando se uma coleção contém uma chave**

```
$roles = User::find(1)->roles;

if ($roles->contains(2))
{
	//
}
```

As coleções também podem ser convertidas em um array ou JSON:

```
$roles = User::find(1)->roles->toArray();

$roles = User::find(1)->roles->toJson();
```

Se uma coleção for convertida em uma string, ela será retornada como JSON:

```
$roles = (string) User::find(1)->roles;
```

As coleções eloquentes também contêm alguns métodos úteis para fazer o looping e filtrar os itens que contêm:

**Iteração de Coleções**

```
$roles = $user->roles->each(function($role)
{
	//
});
```

**Coleções de Filtragem**

Ao filtrar coleções, o callback fornecido será usado como callback para [array\_filter](http://php.net/manual/en/function.array-filter.php).

```
$users = $user->filter(function($user)
{
	if($user->isAdmin())
	{
		return $user;
	}
});
```

> **Observação:**
>
>  Ao filtrar uma coleção e convertê-la em JSON, tente chamar primeiro a função 
>
> `values`
>
>  para reiniciar as chaves do array.

**Aplicando um callback a cada objeto de coleção**

```
$roles = User::find(1)->roles;

$roles->each(function($role)
{
	//
});
```

**Ordenando uma Coleção por um Valor**

```
$roles = $roles->sortBy(function($role)
{
	return $role->created_at;
});
```

Às vezes, você pode querer retornar um objeto personalizado da coleção com seus próprios métodos adicionados. Você pode especificar isso no seu modelo Eloquent sobrescrevendo o método `newCollection`:

**Retornando um tipo de coleção personalizada**

```
class User extends Eloquent {

	public function newCollection(array $models = array())
	{
		return new CustomCollection($models);
	}

}
```

<a name="acessadores-e-mutáveis"></a>

## Atributos e Mutáveis

O Eloquent oferece uma maneira conveniente de transformar os atributos do seu modelo ao obtê-los ou defini-los. Basta definir um método `getFooAttribute` no seu modelo para declarar um acessador. Tenha em mente que os métodos devem seguir a formatação camelCase, mesmo que as colunas do banco de dados sejam em snake\_case:

**Definindo um Acessório**

```
class User extends Eloquent {

	public function getFirstNameAttribute($value)
	{
		return ucfirst($value);
	}

}
```

No exemplo acima, a coluna `first_name` tem um acessor. Observe que o valor do atributo é passado para o acessor.

Os mutantes são declarados de maneira semelhante:

**Definindo um Mutator**

```
class User extends Eloquent {

	public function setFirstNameAttribute($value)
	{
		$this->attributes['first_name'] = strtolower($value);
	}

}
```

<a name="date-mutators"></a>

## Mutantes de data

Por padrão, o Eloquent converterá as colunas `created_at`, `updated_at` e `deleted_at` em instâncias do [Carbon](https://github.com/briannesbitt/Carbon), que oferece uma variedade de métodos úteis e estende a classe nativa PHP `DateTime`.

Você pode personalizar quais campos são mutados automaticamente e até mesmo desativar completamente essa mutação, sobrescrevendo o método `getDates` do modelo:

```
public function getDates()
{
	return array('created_at');
}
```

Quando uma coluna é considerada uma data, você pode definir seu valor para um timestamp UNIX, uma string de data (`Y-m-d`), uma string de data e hora, e, claro, uma instância de `DateTime` / `Carbon`.

Para desativar totalmente as mutações de data, basta retornar um array vazio do método `getDates`:

```
public function getDates()
{
	return array();
}
```

<a name="model-events"></a>

## Eventos Modelo

Modelos eloquentes acionam vários eventos, permitindo que você interaja com vários pontos no ciclo de vida do modelo usando os seguintes métodos: `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`.

Sempre que um novo item é salvo pela primeira vez, os eventos `creating` e `created` serão acionados. Se um item não for novo e o método `save` for chamado, os eventos `updating` / `updated` serão acionados. Em ambos os casos, os eventos `saving` / `saved` serão acionados.

Se `false` for retornado nos eventos `creating`, `updating`, `saving` ou `deleting`, a ação será cancelada:

**Cancelamento de operações de salvar por meio de eventos**

```
User::creating(function($user)
{
	if ( ! $user->isValid()) return false;
});
```

Os modelos eloquentes também contêm um método `boot` estático, que pode fornecer um local conveniente para registrar suas associações de eventos.

**Definindo um Método de Inicialização de Modelo**

```
class User extends Eloquent {

	public static function boot()
	{
		parent::boot();

		// Setup event bindings...
	}

}
```

<a name="modelo-observadores"></a>

## Observadores do modelo

Para consolidar o gerenciamento de eventos de modelo, você pode registrar um observador de modelo. Uma classe de observador pode ter métodos que correspondem aos vários eventos de modelo. Por exemplo, os métodos `criando`, `atualizando`, `salvando` podem estar em um observador, além de qualquer outro nome de evento de modelo.

Então, por exemplo, um modelo observador poderia parecer assim:

```
class UserObserver {

	public function saving($model)
	{
		//
	}

	public function saved($model)
	{
		//
	}

}
```

Você pode registrar uma instância de observador usando o método `observe`:

```
User::observe(new UserObserver);
```

<a name="convertendo-para-arrays-ou-json"></a>

## Conversão para Arrays/JSON

Ao criar APIs JSON, você pode precisar converter seus modelos e relacionamentos em arrays ou JSON. Portanto, o Eloquent inclui métodos para isso. Para converter um modelo e seu relacionamento carregado em um array, você pode usar o método `toArray`:

**Convertendo um Modelo em um Array**

```
$user = User::with('roles')->first();

return $user->toArray();
```

Observe que coleções inteiras de modelos também podem ser convertidas em arrays:

```
return User::all()->toArray();
```

Para converter um modelo para JSON, você pode usar o método `toJson`:

**Convertendo um Modelo para JSON**

```
return User::find(1)->toJson();
```

Observe que, quando um modelo ou coleção é convertido para uma string, ele será convertido para JSON, o que significa que você pode retornar objetos Eloquent diretamente das rotas da sua aplicação!

**Devolvendo um modelo de uma rota**

```
Route::get('users', function()
{
	return User::all();
});
```

Às vezes, você pode querer limitar os atributos que estão incluídos na matriz ou no formulário JSON do seu modelo, como senhas. Para fazer isso, adicione uma definição de propriedade `hidden` ao seu modelo:

**Ocultar atributos da conversão de array ou JSON**

```
class User extends Eloquent {

	protected $hidden = array('password');

}
```

> **Observação:**
>
>  Ao ocultar relações, use o nome do 
>
> **método**
>
>  da relação, e não o nome do acessador dinâmico.

Alternativamente, você pode usar a propriedade `visible` para definir uma lista branca:

```
protected $visible = array('first_name', 'last_name');
```

<a name="array-appends"></a>
Às vezes, você pode precisar adicionar atributos de array que não têm uma coluna correspondente em seu banco de dados. Para fazer isso, basta definir um acessor para o valor:

```
public function getIsAdminAttribute()
{
	return $this->attributes['admin'] == 'yes';
}
```

Depois de criar o acessório, basta adicionar o valor à propriedade `appends` no modelo:

```
protected $appends = array('is_admin');
```

Depois que o atributo for adicionado à lista `appends`, ele será incluído tanto na matriz quanto nas formas JSON do modelo.
