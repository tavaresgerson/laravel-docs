# Roteamento

- [Roteamento Básico](#roteamento-básico)
- [Parâmetros da rota](#parâmetros-da-rota)
- Filtros de rota [Filtros de rota](#route-filters)
- [Rotas Nomeadas](#rotas-nomeadas)
- [Grupos de rotas](#grupos-de-rotas)
- [Roteamento de subdomínio](#sub-domain-routing)
- [Prefixação de rota](#prefixacao-de-rota)
- [Ligação de Modelo de Rota](#ligação-de-modelo-de-rota)
- [Arremessando erros 404](#arremessando-erros-404)
- [Roteamento para controladores](#roteamento-para-controladores)

<a name="roteamento-básico"></a>

## Roteamento básico

A maioria das rotas para sua aplicação será definida no arquivo `app/routes.php`. As rotas mais simples do Laravel consistem em um URI e um callback de fechamento.

**Rota básica GET**

```
Route::get('/', function()
{
	return 'Hello World';
});
```

**Rota básica do POST**

```
Route::post('foo/bar', function()
{
	return 'Hello World';
});
```

**Registrando uma rota que responde a qualquer verbo HTTP**

```
Route::any('foo', function()
{
	return 'Hello World';
});
```

**Forçar a utilização de uma rota através do HTTPS**

```
Route::get('foo', array('https', function()
{
	return 'Must be over HTTPS';
}));
```

Muitas vezes, você precisará gerar URLs para suas rotas. Você pode fazer isso usando o método `URL::to`:

```
$url = URL::to('foo');
```

<a name="parâmetros-da-rota"></a>

## Parâmetros da rota

```
Route::get('user/{id}', function($id)
{
	return 'User '.$id;
});
```

**Parâmetros de rota opcionais**

```
Route::get('user/{name?}', function($name = null)
{
	return $name;
});
```

**Parâmetros de rota opcionais com valores padrão**

```
Route::get('user/{name?}', function($name = 'John')
{
	return $name;
});
```

**Restrições de rota de expressão regular**

```
Route::get('user/{name}', function($name)
{
	//
})
->where('name', '[A-Za-z]+');

Route::get('user/{id}', function($id)
{
	//
})
->where('id', '[0-9]+');
```

Claro, você pode passar um array de restrições quando necessário:

```
Route::get('user/{id}/{name}', function($id, $name)
{
	//
})
->where(array('id' => '[0-9]+', 'name' => '[a-z]+'))
```

Se você deseja que um parâmetro de rota seja sempre limitado por uma expressão regular específica, você pode usar o método `pattern`:

```
Route::pattern('id', '[0-9]+');

Route::get('user/{id}', function($id)
{
	// Only called if {id} is numeric.
});
```

<a name="filtros-de-rota"></a>

## Filtros de rota

Os filtros de rota oferecem uma maneira conveniente de limitar o acesso a uma rota específica, o que é útil para criar áreas do seu site que exigem autenticação. Existem vários filtros incluídos no framework Laravel, incluindo um filtro `auth`, um filtro `auth.basic`, um filtro `guest` e um filtro `csrf`. Esses filtros estão localizados no arquivo `app/filters.php`.

**Definindo um Filtro de Rota**

```
Route::filter('old', function()
{
	if (Input::get('age') < 200)
	{
		return Redirect::to('home');
	}
});
```

Se uma resposta for retornada de um filtro, essa resposta será considerada a resposta ao pedido e a rota não será executada, e quaisquer filtros `after` na rota também serão cancelados.

**Anexar um filtro a uma rota**

```
Route::get('user', array('before' => 'old', function()
{
	return 'You are over 200 years old!';
}));
```

**Anexar um filtro a uma ação do controlador**

```
Route::get('user', array('before' => 'old', 'uses' => 'UserController@showProfile'));
```

**Anexar vários filtros a uma rota**

```
Route::get('user', array('before' => 'auth|old', function()
{
	return 'You are authenticated and over 200 years old!';
}));
```

**Especificando Parâmetros do Filtro**

```
Route::filter('age', function($route, $request, $value)
{
	//
});

Route::get('user', array('before' => 'age:200', function()
{
	return 'Hello World';
}));
```

Após os filtros receberem uma `$response` como o terceiro argumento passado ao filtro:

```
Route::filter('log', function($route, $request, $response, $value)
{
	//
});
```

Filtros baseados em padrões

Você também pode especificar que um filtro se aplica a um conjunto inteiro de rotas com base em seu URI.

```
Route::filter('admin', function()
{
	//
});

Route::when('admin/*', 'admin');
```

No exemplo acima, o filtro `admin` seria aplicado a todas as rotas que começam com `admin/`. O asterisco é usado como um caractere curinga e corresponderá a qualquer combinação de caracteres.

Você também pode restringir os filtros de padrão por verbos HTTP:

```
Route::when('admin/*', 'admin', array('post'));
```

**Classes de filtro**

Para filtragem avançada, você pode querer usar uma classe em vez de uma Closures. Como as classes de filtro são resolvidas fora do container de controle de dependência da aplicação (/docs/ioc), você poderá utilizar a injeção de dependência nesses filtros para maior testável.

**Definindo uma Classe de Filtro**

```
class FooFilter {

	public function filter()
	{
		// Filter logic...
	}

}
```

**Registrar um filtro baseado em classe**

```
Route::filter('foo', 'FooFilter');
```

<a name="rotas-nomeadas"></a>

## Rotas nomeadas

As rotas nomeadas tornam a referência a rotas ao gerar redirecionamentos ou URLs mais conveniente. Você pode especificar um nome para uma rota da seguinte forma:

```
Route::get('user/profile', array('as' => 'profile', function()
{
	//
}));
```

Você também pode especificar nomes de rotas para ações de controle:

```
Route::get('user/profile', array('as' => 'profile', 'uses' => 'UserController@showProfile'));
```

Agora, você pode usar o nome da rota ao gerar URLs ou redirecionamentos:

```
$url = URL::route('profile');

$redirect = Redirect::route('profile');
```

Você pode acessar o nome de uma rota que está em execução através do método `currentRouteName`:

```
$name = Route::currentRouteName();
```

<a name="grupos-de-rota"></a>

## Grupos de rotas

Às vezes, você pode precisar aplicar filtros a um grupo de rotas. Em vez de especificar o filtro em cada rota, você pode usar um grupo de rotas:

```
Route::group(array('before' => 'auth'), function()
{
	Route::get('/', function()
	{
		// Has Auth Filter
	});

	Route::get('user/profile', function()
	{
		// Has Auth Filter
	});
});
```

<a name="sub-domínio-roteamento"></a>

## Roteamento de subdomínio

As rotas do Laravel também podem lidar com subdomínios com asteriscos e passar para você os parâmetros com asteriscos do domínio:

**Registrando Rotas de Subdomínio**

```
Route::group(array('domain' => '{account}.myapp.com'), function()
{

	Route::get('user/{id}', function($account, $id)
	{
		//
	});

});
```

<a name="prefixação-de-rota"></a>

## Prefixação de rota

Um grupo de rotas pode ser prefixado usando a opção `prefix` no array de atributos de um grupo:

**Prefixação de Rotas Agrupadas**

```
Route::group(array('prefix' => 'admin'), function()
{

	Route::get('user', function()
	{
		//
	});

});
```

<a name="route-model-binding"></a>

## Ligação de modelo de rota

A vinculação de modelos oferece uma maneira conveniente de injetar instâncias de modelos em suas rotas. Por exemplo, em vez de injetar o ID de um usuário, você pode injetar a própria instância do modelo User que corresponde ao ID fornecido. Primeiro, use o método `Route::model` para especificar o modelo que deve ser usado para um parâmetro dado:

**Vinculação de um parâmetro a um modelo**

```
Route::model('user', 'User');
```

Em seguida, defina uma rota que contenha um parâmetro `{user}`:

```
Route::get('profile/{user}', function(User $user)
{
	//
});
```

Como vinculamos o parâmetro `{user}` ao modelo `User`, uma instância `User` será injetada na rota. Por exemplo, uma solicitação para `profile/1` injeirá a instância `User` que tem um ID de 1.

> **Nota:**
>
>  Se uma instância de modelo correspondente não for encontrada no banco de dados, será lançada uma mensagem de erro 404.

Se você deseja especificar seu próprio comportamento de "não encontrado", você pode passar uma Closure como o terceiro argumento para o método `model`:

```
Route::model('user', 'User', function()
{
	throw new NotFoundException;
});
```

Às vezes, você pode querer usar seu próprio resolutor para os parâmetros de rota. Basta usar o método `Route::bind`:

```
Route::bind('user', function($value, $route)
{
	return User::where('name', $value)->first();
});
```

<a name="lançando-erros-404"></a>

## Lançar erros 404

Existem duas maneiras de acionar manualmente um erro 404 a partir de uma rota. Primeiro, você pode usar o método `App::abort`:

```
App::abort(404);
```

Em segundo lugar, você pode lançar uma instância de `Symfony\Component\HttpKernel\Exception\NotFoundHttpException`.

Mais informações sobre como lidar com exceções 404 e usar respostas personalizadas para esses erros podem ser encontradas na seção [erros](/docs/errors#handling-404-errors) da documentação.

<a name="roteamento para controladores"></a>

## Roteamento para controladores

O Laravel permite que você não apenas roteie para Closures, mas também para classes de controladores, e até permite a criação de [controladores de recursos](/docs/controllers#resource-controllers).

Consulte a documentação sobre [Controladores](/docs/controllers) para obter mais detalhes.
