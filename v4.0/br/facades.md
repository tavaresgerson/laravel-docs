# Fachadas

- [Introdução](#introdução)
- [Explicação](#explicação)
- [Uso Prático](#uso-prático)
- [Criando Fachadas](#criando-fachadas)
- [Fachadas parodiadas](#fachadas-parodiadas)

<a name="introdução"></a>

## Introdução

As fachadas fornecem uma interface "estática" para classes que estão disponíveis no contêiner de [IoC do aplicativo](/docs/ioc). O Laravel vem com muitas fachadas, e você provavelmente as está usando sem sequer saber!

Às vezes, você pode querer criar suas próprias fachadas para suas aplicações e pacotes, então vamos explorar o conceito, o desenvolvimento e o uso dessas classes.

> **Nota:**
>
>  Antes de se aprofundar nas fachadas, é altamente recomendável que você se familiarize muito com o container 
>
> [IoC do Laravel](/docs/ioc)
>
> .

<a name="explicação"></a>

## Explicação

No contexto de uma aplicação Laravel, uma fachada é uma classe que fornece acesso a um objeto do contêiner. A máquina que faz isso funcionar está na classe `Facade`. As fachadas do Laravel, e quaisquer fachadas personalizadas que você criar, irão estender a classe base `Facade`.

Sua classe de fachada só precisa implementar um único método: `getFacadeAccessor`. É responsabilidade do método `getFacadeAccessor` definir o que deve ser resolvido a partir do container. A classe base `Facade` faz uso do método mágico `__callStatic()` para adiar as chamadas da sua fachada para o objeto resolvido.

<a name="uso-prático"></a>

## Uso prático

No exemplo abaixo, uma chamada é feita ao sistema de cache do Laravel. Ao olhar para esse código, alguém poderia supor que o método estático `get` está sendo chamado na classe `Cache`.

```
$value = Cache::get('key');
```

No entanto, se analisarmos a classe `Illuminate\Support\Facades\Cache`, você verá que não há um método estático `get`:

```
class Cache extends Facade {

	/**
	 * Get the registered name of the component.
	 *
	 * @return string
	 */
	protected static function getFacadeAccessor() { return 'cache'; }

}
```

A classe Cache estende a classe base `Facade` e define um método `getFacadeAccessor()`. Lembre-se de que o trabalho desse método é retornar o nome de uma vinculação IoC.

Quando um usuário faz referência a qualquer método estático na fachada `Cache`, o Laravel resolve o vinculo `cache` do container IoC e executa o método solicitado (neste caso, `get`) contra esse objeto.

Então, nossa chamada `Cache::get` poderia ser reescrita da seguinte forma:

```
$value = $app->make('cache')->get('key');
```

<a name="criando-fachadas"> </a>

## Criando Fachadas

Criar uma fachada para sua própria aplicação ou pacote é simples. Você só precisa de 3 coisas:

- Uma vinculação de IoC
- Uma classe de fachada.
- Uma configuração de alias de fachada.

Vamos ver um exemplo. Aqui, temos uma classe definida como `PaymentGateway\Payment`.

```
namespace PaymentGateway;

class Payment {

	public function process()
	{
		//
	}

}
```

Precisamos ser capazes de resolver essa classe a partir do container de IoC. Então, vamos adicionar um binding:

```
App::bind('payment', function()
{
	return new \PaymentGateway\Payment;
});
```

Um ótimo lugar para registrar essa associação seria criar um novo [provedor de serviços](/docs/ioc#service-providers) chamado `PaymentServiceProvider` e adicionar essa associação ao método `register`. Em seguida, você pode configurar o Laravel para carregar seu provedor de serviços a partir do arquivo de configuração `app/config/app.php`.

Em seguida, podemos criar nossa própria classe de fachada:

```
use Illuminate\Support\Facades\Facade;

class Payment extends Facade {

	protected static function getFacadeAccessor() { return 'payment'; }

}
```

Por fim, se desejarmos, podemos adicionar um alias para nossa fachada ao array `aliases` no arquivo de configuração `app/config/app.php`. Agora, podemos chamar o método `process` em uma instância da classe `Payment`.

```
Payment::process();
```

### Uma nota sobre aliases de autocarregamento

As classes no array `aliases` não estão disponíveis em alguns casos porque [o PHP não tentará carregar automaticamente classes com tipos indicados](https://bugs.php.net/bug.php?id=39003). Se `\ServiceWrapper\ApiTimeoutException` for aliassizado a `ApiTimeoutException`, um `catch(ApiTimeoutException $e)` fora do namespace `\ServiceWrapper` nunca capturará a exceção, mesmo que ela seja lançada. Um problema semelhante é encontrado em Modelos que têm dicas de tipo para classes aliassizadas. A única solução é renunciar à aliassificação e usar as classes que deseja tipificar no topo de cada arquivo que as requer.

<a name="mocking-facades"></a>

## Fachadas zombeteiras

O teste unitário é um aspecto importante da funcionalidade das fachadas. De fato, a testável é a principal razão pela qual as fachadas existem. Para mais informações, confira a seção [fachadas de teste](/docs/testing#mocking-facades) da documentação.
