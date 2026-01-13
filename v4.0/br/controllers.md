# Controladores

- Controladores básicos [Basic Controllers](#basic-controllers)
- [Filtros do controlador](#filtros-do-controlador)
- [Controladores RESTful](#restful-controllers)
- Controladores de recursos (#resource-controllers)
- [Tratamento de métodos ausentes](#tratamento-de-métodos-ausentes)

<a name="controladores-básicos"></a>

## Controladores básicos

Em vez de definir toda a lógica do nível de rota em um único arquivo `routes.php`, você pode querer organizar esse comportamento usando classes de controladores. Os controladores podem agrupar a lógica de rotas relacionadas em uma classe, além de aproveitar recursos mais avançados do framework, como a injeção automática de dependências (<https://docs.laravel.com/5.7/docs/ioc>).

Os controladores são normalmente armazenados no diretório `app/controllers`, e esse diretório é registrado na opção `classmap` do seu arquivo `composer.json` por padrão.

Aqui está um exemplo de uma classe de controle básica:

```
class UserController extends BaseController {

	/**
	 * Show the profile for the given user.
	 */
	public function showProfile($id)
	{
		$user = User::find($id);

		return View::make('user.profile', array('user' => $user));
	}

}
```

Todos os controladores devem estender a classe `BaseController`. O `BaseController` também está armazenado no diretório `app/controllers` e pode ser usado como um local para colocar a lógica compartilhada do controlador. O `BaseController` estende a classe `Controller` do framework. Agora, podemos redirecionar para essa ação do controlador da seguinte forma:

```
Route::get('user/{id}', 'UserController@showProfile');
```

Se você optar por aninhar ou organizar seu controlador usando namespaces PHP, basta usar o nome da classe totalmente qualificado ao definir a rota:

```
Route::get('foo', 'Namespace\FooController@method');
```

Você também pode especificar nomes nas rotas do controlador:

```
Route::get('foo', array('uses' => 'FooController@method',
										'as' => 'name'));
```

Para gerar um URL para uma ação de controle, você pode usar o método `URL::action`:

```
$url = URL::action('FooController@method');
```

Você pode acessar o nome da ação do controlador que está sendo executada usando o método `currentRouteAction`:

```
$action = Route::currentRouteAction();
```

<a name="controller-filters"></a>

## Filtros de controle

Os filtros podem ser especificados em rotas de controladores de forma semelhante às rotas "regulares": [Filtros](/docs/routing#route-filters)

```
Route::get('profile', array('before' => 'auth',
			'uses' => 'UserController@showProfile'));
```

No entanto, você também pode especificar filtros dentro do seu controlador:

```
class UserController extends BaseController {

	/**
	 * Instantiate a new UserController instance.
	 */
	public function __construct()
	{
		$this->beforeFilter('auth', array('except' => 'getLogin'));

		$this->beforeFilter('csrf', array('on' => 'post'));

		$this->afterFilter('log', array('only' =>
							array('fooAction', 'barAction')));
	}

}
```

Você também pode especificar filtros de controle inline usando uma Closure:

```
class UserController extends BaseController {

	/**
	 * Instantiate a new UserController instance.
	 */
	public function __construct()
	{
		$this->beforeFilter(function()
		{
			//
		});
	}

}
```

<a name="controladores-restful"></a>

## Controladores RESTful

O Laravel permite que você defina facilmente uma única rota para lidar com todas as ações em um controlador usando convenções de nomenclatura REST simples. Primeiro, defina a rota usando o método `Route::controller`:

**Definindo um Controlador RESTful**

```
Route::controller('users', 'UserController');
```

O método `controller` aceita dois argumentos. O primeiro é o URI base que o controlador gerencia, enquanto o segundo é o nome da classe do controlador. Em seguida, basta adicionar métodos ao seu controlador, precedidos pelo verbo HTTP ao qual eles respondem:

```
class UserController extends BaseController {

	public function getIndex()
	{
		//
	}

	public function postProfile()
	{
		//
	}

}
```

Os métodos `index` responderão ao URI raiz tratado pelo controlador, que, neste caso, é `users`.

Se sua ação do controlador contiver várias palavras, você pode acessar a ação usando a sintaxe "dash" no URI. Por exemplo, a seguinte ação do controlador em nosso `UserController` responderia ao URI `users/admin-profile`:

```
public function getAdminProfile() {}
```

<a name="resource-controllers"></a>

## Controladores de Recursos

Os controladores de recursos facilitam a criação de controladores RESTful em torno dos recursos. Por exemplo, você pode querer criar um controlador que gerencie as "fotos" armazenadas pelo seu aplicativo. Usando o comando `controller:make` via o Artisan CLI e o método `Route::resource`, podemos criar rapidamente um controlador desse tipo.

Para criar o controlador via linha de comando, execute o seguinte comando:

```
php artisan controller:make PhotoController
```

Agora podemos registrar uma rota inteligente para o controlador:

```
Route::resource('photo', 'PhotoController');
```

Essa declaração de rota única cria múltiplas rotas para lidar com uma variedade de ações RESTful no recurso de foto. Da mesma forma, o controlador gerado já terá métodos stubbed para cada uma dessas ações, com notas informando quais URIs e verbos eles lidam.

**Ações Geridas pelo Controlador de Recursos**

| Verbo     | Caminho                     | Ação      | Nome da rota     |
| --------- | --------------------------- | --------- | ---------------- |
| GET       | /resource                   | índice    | recurso.index    |
| GET       | /resource/create            | criar     | recurso.criar    |
| POST      | /resource                   | loja      | resource.store   |
| GET       | /resource/{resource}        | mostrar   | recurso.mostrar  |
| GET       | /resource/{resource}/editar | editar    | recurso.editar   |
| PUT/PATCH | /resource/{resource}        | atualizar | recurso.update   |
| DELETA    | /resource/{resource}        | destruir  | resource.destroy |

Às vezes, você pode precisar apenas lidar com um subconjunto das ações do recurso:

```
php artisan controller:make PhotoController --only=index,show

php artisan controller:make PhotoController --except=index
```

E você também pode especificar um subconjunto de ações a serem tratadas na rota:

```
Route::resource('photo', 'PhotoController',
				array('only' => array('index', 'show')));

Route::resource('photo', 'PhotoController',
				array('except' => array('create', 'store', 'update', 'delete')));
```

<a name="manejo-de-métodos-faltantes"></a>

## Lidando com métodos ausentes

Pode ser definido um método genérico que será chamado quando nenhum outro método correspondente for encontrado em um controlador específico. O método deve ser chamado de `missingMethod` e receber o array de parâmetros do pedido como seu único argumento:

**Definindo um Método de Caça-Núcleo**

```
public function missingMethod($parameters)
{
	//
}
```
