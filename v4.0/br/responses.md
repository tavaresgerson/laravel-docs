# Visões e Respostas

- [Respostas Básicas](#respostas-básicas)
- [Redireciona](#redirecionamentos)
- [Visões](#visões)
- [Ver Compositores](#ver-compositores)
- [Respostas Especiais](#respostas-especiais)

<a name="respostas-básicas"></a>

## Respostas básicas

**Retorno de cadeias de caracteres de rotas**

```
Route::get('/', function()
{
	return 'Hello World';
});
```

**Criando Respostas Personalizadas**

Uma instância de `Response` herda da classe `Symfony\Component\HttpFoundation\Response`, fornecendo uma variedade de métodos para criar respostas HTTP.

```
$response = Response::make($contents, $statusCode);

$response->header('Content-Type', $value);

return $response;
```

**Anexar cookies às respostas**

```
$cookie = Cookie::make('name', 'value');

return Response::make($content)->withCookie($cookie);
```

<a name="redirects"></a>

## Redirecionamentos

**Retorno de um redirecionamento**

```
return Redirect::to('user/login');
```

**Retornando um redirecionamento com dados Flash**

```
return Redirect::to('user/login')->with('message', 'Login Failed');
```

> **Observação:**
>
>  Como o método 
>
> `with`
>
>  atualiza os dados na sessão, você pode recuperar os dados usando o método típico 
>
> `Session::get`
>
> .

**Retorno de um redirecionamento para uma rota nomeada**

```
return Redirect::route('login');
```

**Retornando um redirecionamento para uma rota nomeada com parâmetros**

```
return Redirect::route('profile', array(1));
```

**Retornando um redirecionamento para uma rota nomeada usando parâmetros nomeados**

```
return Redirect::route('profile', array('user' => 1));
```

**Retornando um redirecionamento para uma ação do controlador**

```
return Redirect::action('HomeController@index');
```

**Retornando um redirecionamento para uma ação de controle com parâmetros**

```
return Redirect::action('UserController@profile', array(1));
```

**Retornando um redirecionamento para uma ação de controle usando parâmetros nomeados**

```
return Redirect::action('UserController@profile', array('user' => 1));
```

<a name="visões"></a>

## Visões

As views geralmente contêm o HTML da sua aplicação e fornecem uma maneira conveniente de separar a lógica do controlador e do domínio da lógica de apresentação. As views são armazenadas no diretório `app/views`.

Uma visão simples poderia parecer algo assim:

```
<!-- View stored in app/views/greeting.php -->

<html>
	<body>
		<h1>Hello, <?php echo $name; ?></h1>
	</body>
</html>
```

Essa visualização pode ser devolvida ao navegador da seguinte forma:

```
Route::get('/', function()
{
	return View::make('greeting', array('name' => 'Taylor'));
});
```

O segundo argumento passado para `View::make` é um array de dados que deve ser disponibilizado à vista.

**Passagem de dados para visualizações**

```
$view = View::make('greeting')->with('name', 'Steve');
```

No exemplo acima, a variável `$name` seria acessível a partir da visualização e conteria `Steve`.

Se desejar, você pode passar um array de dados como o segundo parâmetro para o método `make`:

```
$view = View::make('greetings', $data);
```

Você também pode compartilhar um pedaço de dados em todas as visualizações:

```
View::share('name', 'Steve');
```

**Passar uma subvisualização para uma visualização**

Às vezes, você pode querer passar uma vista para outra. Por exemplo, dado uma sub-vista armazenada em `app/views/child/view.php`, podemos passá-la para outra vista da seguinte forma:

```
$view = View::make('greeting')->nest('child', 'child.view');

$view = View::make('greeting')->nest('child', 'child.view', $data);
```

A subvisualização pode então ser renderizada a partir da visualização principal:

```
<html>
	<body>
		<h1>Hello!</h1>
		<?php echo $child; ?>
	</body>
</html>
```

<a name="view-composers"></a>

## Ver Compositores

Os compositores de visualização são métodos de chamada ou métodos de classe que são acionados quando uma visualização é renderizada. Se você tem dados que deseja vincular a uma visualização específica cada vez que essa visualização for renderizada em toda a sua aplicação, um compositor de visualização pode organizar esse código em um único local. Portanto, os compositores de visualização podem funcionar como "modelos de visualização" ou "presentes".

**Definindo um Compilador de Visualizações**

```
View::composer('profile', function($view)
{
	$view->with('count', User::count());
});
```

Agora, cada vez que a visualização `profile` for renderizada, os dados `count` serão vinculados à visualização.

Você também pode anexar um criador de visualizações a várias visualizações de uma só vez:

```
View::composer(array('profile','dashboard'), function($view)
{
    $view->with('count', User::count());
});
```

Se você prefere usar um compósito baseado em classes, que proporcionará os benefícios de ser resolvido através do container de IoC [IoC Container](/docs/ioc), você pode fazê-lo:

```
View::composer('profile', 'ProfileComposer');
```

Uma classe de compositor de visualização deve ser definida da seguinte forma:

```
class ProfileComposer {

	public function compose($view)
	{
		$view->with('count', User::count());
	}

}
```

Observe que não há uma convenção sobre onde as classes de composer podem ser armazenadas. Você pode armazená-las em qualquer lugar, desde que possam ser carregadas automaticamente usando as diretivas no seu arquivo `composer.json`.

### Ver Criadores

As views **criadoras** funcionam quase exatamente como os compositores de visualizações; no entanto, elas são removidas imediatamente quando a visualização é instanciada. Para registrar uma view criadora, basta usar o método `creator`:

```
View::creator('profile', function($view)
{
	$view->with('count', User::count());
});
```

<a name="respostas-especiais"></a>

## Respostas Especiais

**Criando uma Resposta JSON**

```
return Response::json(array('name' => 'Steve', 'state' => 'CA'));
```

**Criando uma Resposta JSONP**

```
return Response::json(array('name' => 'Steve', 'state' => 'CA'))->setCallback(Input::get('callback'));
```

**Criando uma resposta para download de arquivo**

```
return Response::download($pathToFile);

return Response::download($pathToFile, $name, $headers);
```
