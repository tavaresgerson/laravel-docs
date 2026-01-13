# Contêiner IoC

- [Introdução](#introdução)
- [Uso básico](#uso-básico)
- [Resolução Automática](#resolução-automática)
- [Uso Prático](#uso-prático)
- [Prestadores de Serviços](#prestadores-de-servicos)
- Eventos de contêineres (#container-events)

<a name="introdução"></a>

## Introdução

O container de inervação de controle da Laravel é uma ferramenta poderosa para gerenciar as dependências das classes. A injeção de dependências é um método para remover dependências de classes codificadas manualmente. Em vez disso, as dependências são injetadas no tempo de execução, permitindo maior flexibilidade, pois as implementações das dependências podem ser trocadas facilmente.

Entender o contêiner IoC do Laravel é essencial para a construção de uma aplicação poderosa e de grande porte, bem como para contribuir para o próprio núcleo do Laravel.

<a name="uso-básico"></a>

## Uso básico

Existem duas maneiras pelas quais o contêiner IoC pode resolver dependências: por meio de callbacks de Closure ou resolução automática. Primeiro, vamos explorar os callbacks de Closure. Primeiro, um "tipo" pode ser vinculado ao contêiner:

**Encaixar um tipo no contêiner**

```
App::bind('foo', function($app)
{
	return new FooBar;
});
```

**Resolvendo um Tipo do Contêiner**

```
$value = App::make('foo');
```

Quando o método `App::make` é chamado, o callback do Closure é executado e o resultado é retornado.

Às vezes, você pode querer vincular algo ao contêiner que só deve ser resolvido uma vez, e a mesma instância deve ser devolvida em chamadas subsequentes ao contêiner:

**Vinculação de um tipo "Compartilhado" no Contêiner**

```
App::singleton('foo', function()
{
	return new FooBar;
});
```

Você também pode vincular uma instância de objeto existente ao contêiner usando o método `instance`:

**Vinculação de uma Instância Existente ao Contêiner**

```
$foo = new Foo;

App::instance('foo', $foo);
```

<a name="resolução-automática"></a>

## Resolução automática

O contêiner IoC é poderoso o suficiente para resolver classes sem qualquer configuração em muitos cenários. Por exemplo:

**Resolvendo uma Classe**

```
class FooBar {

	public function __construct(Baz $baz)
	{
		$this->baz = $baz;
	}

}

$fooBar = App::make('FooBar');
```

Observe que, embora não tenhamos registrado a classe FooBar no contêiner, ele ainda poderá resolver a classe, mesmo injetando a dependência `Baz` automaticamente!

Quando um tipo não está vinculado no contêiner, ele usará as facilidades de Reflexão do PHP para inspecionar a classe e ler as dicas de tipo do construtor. Usando essas informações, o contêiner pode construir automaticamente uma instância da classe.

No entanto, em alguns casos, uma classe pode depender de uma implementação de interface, e não de um "tipo concreto". Quando isso acontece, o método `App::bind` deve ser usado para informar o container qual implementação de interface deve ser injetada:

**Vinculando uma interface a uma implementação**

```
App::bind('UserRepositoryInterface', 'DbUserRepository');
```

Agora, considere o seguinte controlador:

```
class UserController extends BaseController {

	public function __construct(UserRepositoryInterface $users)
	{
		$this->users = $users;
	}

}
```

Como vinculamos a `UserRepositoryInterface` a um tipo concreto, o `DbUserRepository` será injetado automaticamente neste controlador quando ele for criado.

<a name="uso-prático"></a>

## Uso prático

O Laravel oferece várias oportunidades para usar o container de IoC para aumentar a flexibilidade e a testável da sua aplicação. Um exemplo primário é quando se resolvem os controladores. Todos os controladores são resolvidos através do container de IoC, o que significa que você pode definir dependências de tipo no construtor de um controlador e elas serão injetadas automaticamente.

**Dependências de controladores com indicação de tipo H**

```
class OrderController extends BaseController {

	public function __construct(OrderRepository $orders)
	{
		$this->orders = $orders;
	}

	public function getIndex()
	{
		$all = $this->orders->all();

		return View::make('orders', compact('all'));
	}

}
```

Neste exemplo, a classe `OrderRepository` será injetada automaticamente no controlador. Isso significa que, ao realizar testes unitários (/docs/testing), um `OrderRepository` "fantoche" pode ser vinculado ao contêiner e injetado no controlador, permitindo a substituição suave da interação com a camada de banco de dados.

Os filtros (/docs/routing#route-filters), os compositores (/docs/responses#view-composers) e os manipuladores de eventos (/docs/events#using-classes-as-listeners) também podem ser resolvidos fora do contêiner de IoC. Ao registrá-los, basta fornecer o nome da classe que deve ser usada:

**Outros exemplos de uso do IoC**

```
Route::filter('foo', 'FooFilter');

View::composer('foo', 'FooComposer');

Event::listen('foo', 'FooHandler');
```

<a name="prestadores-de-serviços"></a>

## Prestadores de Serviços

Os provedores de serviço são uma ótima maneira de agrupar registros de IoC relacionados em um único local. Pense neles como uma maneira de inicializar componentes em sua aplicação. Dentro de um provedor de serviço, você pode registrar um driver de autenticação personalizado, registrar as classes de repositório da sua aplicação com o contêiner de IoC ou até mesmo configurar um comando personalizado do Artisan.

Na verdade, a maioria dos componentes principais do Laravel inclui provedores de serviço. Todos os provedores de serviço registrados para sua aplicação estão listados no array `providers` do arquivo de configuração `app/config/app.php`.

Para criar um provedor de serviços, basta estender a classe `Illuminate\Support\ServiceProvider` e definir um método `register`:

**Definindo um Prestador de Serviços**

```
use Illuminate\Support\ServiceProvider;

class FooServiceProvider extends ServiceProvider {

	public function register()
	{
		$this->app->bind('foo', function()
		{
			return new Foo;
		});
	}

}
```

Observe que, no método `register`, o container de IoC da aplicação está disponível para você através da propriedade `$this->app`. Depois de criar um provedor e estar pronto para registrá-lo na sua aplicação, basta adicioná-lo ao array `providers` no seu arquivo de configuração `app`.

Você também pode registrar um provedor de serviço no tempo de execução usando o método `App::register`:

**Registrar um Prestador de Serviços em Tempo Real**

```
App::register('FooServiceProvider');
```

<a name="container-eventos"></a>

## Eventos de Contêineres

O contêiner dispara um evento toda vez que resolve um objeto. Você pode ouvir esse evento usando o método `resolving`:

**Registrando um Ouvinte de Resolução**

```
App::resolvingAny(function($object)
{
	//
});

App::resolving('foo', function($foo)
{
	//
});
```

Observe que o objeto que foi resolvido será passado para o callback.
