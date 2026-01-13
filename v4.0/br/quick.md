# Laravel Quickstart

- [Instalação](#instalação)
- [Roteamento](#roteamento)
- [Criando uma visualização](#criando-uma-visualizacao)
- [Criando uma Migração](#criando-uma-migracao)
- [ORM Eloquent](#orm-eloquent)
- [Exibir dados](#exibir-dados)

<a name="instalação"></a>

## Instalação

O framework Laravel utiliza o [Composer](http://getcomposer.org) para instalação e gerenciamento de dependências. Se ainda não o fez, comece instalando o [Composer](http://getcomposer.org/doc/00-intro.md).

Agora você pode instalar o Laravel executando o seguinte comando no seu terminal:

```
composer create-project laravel/laravel=4.0.* your-project-name --prefer-dist
```

Esse comando baixará e instalará uma cópia fresca do Laravel em uma nova pasta `seu-nome-do-projeto` dentro do seu diretório atual.

Se preferir, você pode, como alternativa, baixar manualmente uma cópia do [repositório do Laravel do Github](https://github.com/laravel/laravel/archive/master.zip). Em seguida, execute o comando `composer install` no diretório raiz do seu projeto criado manualmente. Esse comando baixará e instalará as dependências do framework.

<a name="permissões"></a>

### Permissões

Após instalar o Laravel, você pode precisar conceder permissões de escrita ao servidor web para os diretórios `app/storage`. Consulte a documentação de [Instalação](/docs/installation) para mais detalhes sobre a configuração.

<a name="diretórios"></a>

### Estrutura do diretório

Após instalar o framework, dê uma olhada pelo projeto para se familiarizar com a estrutura de diretórios. O diretório `app` contém pastas como `views`, `controllers` e `models`. A maior parte do código da sua aplicação estará em algum lugar neste diretório. Você também pode querer explorar o diretório `app/config` e as opções de configuração disponíveis para você.

<a name="roteamento">roteamento</a>

## Roteamento

Para começar, vamos criar nossa primeira rota. No Laravel, a rota mais simples é uma rota para uma Closure. Abra o arquivo `app/routes.php` e adicione a seguinte rota no final do arquivo:

```
Route::get('users', function()
{
	return 'Users!';
});
```

Agora, se você acessar a rota `/users` no seu navegador da web, você deve ver "Users!" exibido como a resposta. Ótimo! Você acabou de criar sua primeira rota.

As rotas também podem ser anexadas a classes de controle. Por exemplo:

```
Route::get('users', 'UserController@getIndex');
```

Esta rota informa que as requisições para a rota `/users` devem chamar o método `getIndex` na classe `UserController`. Para obter mais informações sobre o roteamento de controladores, consulte a [documentação do controlador](/docs/controllers).

<a name="criando-uma-visualização"></a>

## Criando uma visão

Em seguida, criaremos uma visão simples para exibir nossos dados de usuário. As visões vivem no diretório `app/views` e contêm o HTML da sua aplicação. Vamos colocar duas novas visões neste diretório: `layout.blade.php` e `users.blade.php`. Primeiro, vamos criar nosso arquivo `layout.blade.php`:

```
<html>
	<body>
		<h1>Laravel Quickstart</h1>

		@yield('content')
	</body>
</html>
```

Em seguida, criaremos nossa vista `users.blade.php`:

```
@extends('layout')

@section('content')
	Users!
@stop
```

Algumas dessas sintaxes provavelmente parecerão estranhas para você. Isso ocorre porque estamos usando o sistema de modelagem do Laravel: o Blade. O Blade é muito rápido, pois é simplesmente um conjunto de expressões regulares que são executadas em suas templates para compilá-las em PHP puro. O Blade oferece funcionalidades poderosas, como herança de templates, além de alguns açúcares sintáticos em estruturas de controle típicas do PHP, como `if` e `for`. Confira a [documentação do Blade](/docs/templates) para mais detalhes.

Agora que temos nossas visualizações, vamos devolvê-las a partir da nossa rota `/users`. Em vez de retornar `Users!` da rota, retorne a visualização em vez disso:

```
Route::get('users', function()
{
	return View::make('users');
});
```

Ótimo! Agora você configurou uma visualização simples que estende um layout. Em seguida, vamos começar a trabalhar na camada de banco de dados.

<a name="criando-uma-migração"></a>

## Criando uma Migração

Para criar uma tabela para armazenar nossos dados, usaremos o sistema de migração do Laravel. As migrações permitem que você defina expressamente as modificações no seu banco de dados e compartilhe facilmente com o resto da sua equipe.

Primeiro, vamos configurar uma conexão de banco de dados. Você pode configurar todas as suas conexões de banco de dados no arquivo `app/config/database.php`. Por padrão, o Laravel está configurado para usar MySQL, e você precisará fornecer as credenciais de conexão no arquivo de configuração do banco de dados. Se desejar, você pode alterar a opção `driver` para `sqlite` e ele usará o banco de dados SQLite incluído no diretório `app/database`.

Em seguida, para criar a migração, usaremos o [Artisan CLI](/docs/artisan). Do diretório raiz do seu projeto, execute o seguinte no terminal:

```
php artisan migrate:make create_users_table
```

Em seguida, encontre o arquivo de migração gerado na pasta `app/database/migrations`. Este arquivo contém uma classe com dois métodos: `up` e `down`. No método `up`, você deve fazer as alterações desejadas nas tabelas do banco de dados, e no método `down`, você simplesmente inverte-as.

Vamos definir uma migração que tenha a seguinte aparência:

```
public function up()
{
	Schema::create('users', function($table)
	{
		$table->increments('id');
		$table->string('email')->unique();
		$table->string('name');
		$table->timestamps();
	});
}

public function down()
{
	Schema::drop('users');
}
```

Em seguida, podemos executar nossas migrações no nosso terminal usando o comando `migrate`. Basta executar este comando a partir da raiz do seu projeto:

```
php artisan migrate
```

Se você deseja reverter uma migração, pode emitir o comando `migrate:rollback`. Agora que temos uma tabela de banco de dados, vamos começar a extrair alguns dados!

<a name="eloquente-orm"></a>

## ORM eloquente

O Laravel vem com um ORM excelente: Eloquent. Se você já usou o framework Ruby on Rails, vai achar o Eloquent familiar, pois segue o estilo de interação com o banco de dados do ORM ActiveRecord.

Primeiro, vamos definir um modelo. Um modelo Eloquent pode ser usado para consultar uma tabela de banco de dados associada, bem como para representar uma determinada linha dentro dessa tabela. Não se preocupe, tudo ficará claro em breve! Os modelos são normalmente armazenados no diretório `app/models`. Vamos definir um modelo `User.php` nesse diretório da seguinte forma:

```
class User extends Eloquent {}
```

Observe que não precisamos dizer ao Eloquent qual tabela usar. O Eloquent tem várias convenções, uma das quais é usar a forma plural do nome do modelo como a tabela do banco de dados do modelo. Conveniente!

Use sua ferramenta de administração de banco de dados preferida para inserir algumas linhas na sua tabela `users`, e usaremos Eloquent para recuperá-las e passá-las para nossa visualização.

Agora, vamos modificar nossa rota `/users` para ficar assim:

```
Route::get('users', function()
{
	$users = User::all();

	return View::make('users')->with('users', $users);
});
```

Vamos percorrer essa rota. Primeiro, o método `all` no modelo `User` recuperará todas as linhas da tabela `users`. Em seguida, passamos esses registros para a visualização usando o método `with`. O método `with` aceita uma chave e um valor e é usado para tornar um pedaço de dados disponível para uma visualização.

Ótimo. Agora estamos prontos para exibir os usuários em nossa visualização!

<a name="exibir-dados"></a>

## Exibir dados

Agora que disponibilizamos os `users` para nossa visualização, podemos exibí-los da seguinte forma:

```
@extends('layout')

@section('content')
	@foreach($users as $user)
		<p>{{ $user->name }}</p>
	@endforeach
@stop
```

Você pode estar se perguntando onde encontrar nossas declarações `echo`. Ao usar o Blade, você pode exibir dados ao envolvê-los em chaves duplas. É fácil. Agora, você deve ser capaz de acessar a rota `/users` e ver os nomes dos seus usuários exibidos na resposta.

Isso é apenas o começo. Neste tutorial, você viu os conceitos básicos do Laravel, mas há muito mais coisas interessantes para aprender. Continue lendo a documentação e aprofunde-se nas poderosas funcionalidades disponíveis para você em [Eloquent](/docs/eloquent) e [Blade](/docs/templates). Ou, talvez você esteja mais interessado em [Queues](/docs/queues) e [Testes Unitários](/docs/testing). Depois, talvez você queira exercitar seus músculos de arquitetura com o [Container IoC](/docs/ioc). A escolha é sua!
