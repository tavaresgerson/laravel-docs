# Eventos

- [Uso básico](#uso-básico)
- [Ouvidores wildcard](#ouvidores-wildcard)
- [Usando Classes como Ouvidores](#using-classes-as-listeners)
- Eventos em fila (#eventos-em-fila)
- [Assinantes de eventos](#event-subscribers)

<a name="uso-básico"></a>

## Uso básico

A classe `Event` do Laravel oferece uma implementação simples de observador, permitindo que você se inscreva e ouça eventos em sua aplicação.

**Assinando para um Evento**

```
Event::listen('user.login', function($user)
{
	$user->last_login = new DateTime;

	$user->save();
});
```

**Desligar um Evento**

```
$event = Event::fire('user.login', array($user));
```

Você também pode especificar uma prioridade ao se inscrever em eventos. Os ouvintes com prioridade mais alta serão executados primeiro, enquanto os ouvintes que têm a mesma prioridade serão executados na ordem de inscrição.

**Assinando para Eventos com Prioridade**

```
Event::listen('user.login', 'LoginHandler', 10);

Event::listen('user.login', 'OtherHandler', 5);
```

Às vezes, você pode querer interromper a propagação de um evento para outros ouvintes. Você pode fazer isso retornando `false` de seu ouvinte:

**Parando a propagação de um evento**

```
Event::listen('user.login', function($event)
{
	// Handle the event...

	return false;
});
```

<a name="wildcards-de-ouvidos">wildcards-listeners</a>

## Ouvintes com wildcard

Ao registrar um ouvinte de evento, você pode usar asteriscos para especificar ouvinte de wildcard:

**Registrando ouvintes de eventos com wildcard**

```
Event::listen('foo.*', function($param, $event)
{
	// Handle the event...
});
```

Este ouvinte tratará de todos os eventos que começam com `foo`. Observe que o nome completo do evento é passado como o último argumento ao manipulador.

<a name="usando-classes-como-ouvidores"></a>

## Usando Classes como Ouvidores

Em alguns casos, você pode querer usar uma classe para lidar com um evento em vez de uma Closure. Os ouvintes de eventos da classe serão resolvidos pelo [container de IoC do Laravel](/docs/ioc), fornecendo a você todo o poder de injeção de dependências em seus ouvintes.

**Registrar um Ouvinte de Classe**

```
Event::listen('user.login', 'LoginHandler');
```

Por padrão, o método `handle` na classe `LoginHandler` será chamado:

**Definindo uma Classe de Ouvinte de Eventos**

```
class LoginHandler {

	public function handle($data)
	{
		//
	}

}
```

Se você não quiser usar o método padrão `handle`, você pode especificar o método que deve ser assinado:

**Especificando qual método de assinatura**

```
Event::listen('user.login', 'LoginHandler@onLogin');
```

<a name="eventos_em_fila"></a>

## Eventos em fila

Usando os métodos `queue` e `flush`, você pode "enfileirar" um evento para ser disparado, mas não dispará-lo imediatamente:

**Registrar um evento em espera**

```
Event::queue('foo', array($user));
```

**Registrar um limpador de eventos**

```
Event::flusher('foo', function($user)
{
	//
});
```

Por fim, você pode executar o "flusher" e limpar todos os eventos em fila usando o método `flush`:

```
Event::flush('foo');
```

<a name="event-subscribers"></a>

## Assinantes de eventos

Os assinantes de eventos são classes que podem se inscrever em vários eventos dentro da própria classe. Os assinantes devem definir um método `subscribe`, que será passado uma instância do dispatcher de eventos:

**Definindo um Assinante de Eventos**

```
class UserEventHandler {

	/**
	 * Handle user login events.
	 */
	public function onUserLogin($event)
	{
		//
	}

	/**
	 * Handle user logout events.
	 */
	public function onUserLogout($event)
	{
		//
	}

	/**
	 * Register the listeners for the subscriber.
	 *
	 * @param  Illuminate\Events\Dispatcher  $events
	 * @return array
	 */
	public function subscribe($events)
	{
		$events->listen('user.login', 'UserEventHandler@onUserLogin');

		$events->listen('user.logout', 'UserEventHandler@onUserLogout');
	}

}
```

Depois que o assinante for definido, ele pode ser registrado na classe `Event`.

**Registrar um assinante de evento**

```
$subscriber = new UserEventHandler;

Event::subscribe($subscriber);
```
