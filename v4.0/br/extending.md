# Ampliando o Quadro

- [Introdução](#introdução)
- [Gerentes e fábricas](#gerentes-e-fábricas)
- [Cache](#cache)
- [Sessão](#sessão)
- [Autenticação](#autenticação)
- Extensão baseada em IoC (#ioc-based-extension)
- [Solicitação de Extensão](#solicitação-de-extensão)

<a name="introdução"></a>

## Introdução

O Laravel oferece muitos pontos de extensão para você personalizar o comportamento dos componentes principais do framework ou até mesmo substituí-los completamente. Por exemplo, as facilidades de hashing são definidas por um contrato `HasherInterface`, que você pode implementar com base nas necessidades da sua aplicação. Você também pode estender o objeto `Request`, permitindo que você adicione seus próprios métodos "ajudantes" convenientes. Você pode até adicionar drivers de autenticação, cache e sessão completamente novos!

Os componentes do Laravel são geralmente estendidos de duas maneiras: vinculando novas implementações no container de IoC ou registrando uma extensão com uma classe `Manager`, que são implementações do padrão de design "Factory". Neste capítulo, exploraremos os vários métodos de extensão do framework e examinaremos o código necessário.

> **Nota:**
>
>  Lembre-se de que os componentes do Laravel são tipicamente estendidos de duas maneiras: vinculações IoC e as classes 
>
> `Manager`
>
> . As classes manager servem como uma implementação do padrão de design "fábrica" e são responsáveis por instanciar facilidades baseadas em drivers, como cache e sessão.

<a name="gerentes-e-fábricas"></a>

## Gerentes e fábricas

O Laravel possui várias classes `Manager` que gerenciam a criação de componentes baseados em drivers. Essas incluem os componentes cache, sessão, autenticação e fila. A classe manager é responsável por criar uma implementação específica do driver com base na configuração da aplicação. Por exemplo, a classe `CacheManager` pode criar APC, Memcached, Native e várias outras implementações de drivers de cache.

Cada um desses gestores inclui um método `extend` que pode ser usado para injetar facilmente nova funcionalidade de resolução de drivers no gerenciador. Vamos cobrir cada um desses gestores abaixo, com exemplos de como injetar suporte de driver personalizado em cada um deles.

> **Nota:**
>
>  Reserve um momento para explorar as várias classes 
>
> `Manager`
>
>  que vêm com o Laravel, como o 
>
> `CacheManager`
>
>  e o 
>
> `SessionManager`
>
> . Ler essas classes lhe dará uma compreensão mais completa de como o Laravel funciona por baixo do capô. Todas as classes de gerenciador estendem a classe base 
>
> `Illuminate\Support\Manager`
>
> , que fornece algumas funcionalidades úteis e comuns para cada gerenciador.

<a name="cache"></a>

## Cache

Para estender a funcionalidade de cache do Laravel, usaremos o método `extend` no `CacheManager`, que é usado para vincular um resolutor de driver personalizado ao gerenciador, e é comum em todas as classes de gerenciador. Por exemplo, para registrar um novo driver de cache chamado "mongo", faríamos o seguinte:

```
Cache::extend('mongo', function($app)
{
	// Return Illuminate\Cache\Repository instance...
});
```

O primeiro argumento passado para o método `extend` é o nome do driver. Isso corresponderá à sua opção `driver` no arquivo de configuração `app/config/cache.php`. O segundo argumento é uma Closure que deve retornar uma instância de `Illuminate\Cache\Repository`. A Closure será passada uma instância `$app`, que é uma instância de `Illuminate\Foundation\Application` e um contêiner de IoC.

Para criar nosso driver de cache personalizado, primeiro precisamos implementar o contrato `Illuminate\Cache\StoreInterface`. Portanto, nossa implementação de cache no MongoDB seria algo como isso:

```
class MongoStore implements Illuminate\Cache\StoreInterface {

	public function get($key) {}
	public function put($key, $value, $minutes) {}
	public function increment($key, $value = 1) {}
	public function decrement($key, $value = 1) {}
	public function forever($key, $value) {}
	public function forget($key) {}
	public function flush() {}

}
```

Só precisamos implementar cada um desses métodos usando uma conexão com o MongoDB. Uma vez que nossa implementação esteja concluída, podemos finalizar o registro do nosso driver personalizado:

```
use Illuminate\Cache\Repository;

Cache::extend('mongo', function($app)
{
	return new Repository(new MongoStore);
});
```

Como você pode ver no exemplo acima, você pode usar a base `Illuminate\Cache\Repository` ao criar drivers de cache personalizados. Normalmente, não é necessário criar sua própria classe de repositório.

Se você está se perguntando onde colocar seu código de driver de cache personalizado, considere disponibilizá-lo no Packagist! Ou, você poderia criar um namespace `Extensions` dentro da pasta principal do seu aplicativo. Por exemplo, se o aplicativo for chamado de `Snappy`, você poderia colocar a extensão de cache em `app/Snappy/Extensions/MongoStore.php`. No entanto, tenha em mente que o Laravel não tem uma estrutura de aplicativo rígida e você é livre para organizar seu aplicativo de acordo com suas preferências.

> **Nota:**
>
>  Se você já se perguntou onde colocar um pedaço de código, sempre considere um provedor de serviços. Como discutimos, usar um provedor de serviços para organizar extensões de framework é uma ótima maneira de organizar seu código.

<a name="sessão"></a>

## Sessão

Extender o Laravel com um driver de sessão personalizado é tão fácil quanto estender o sistema de cache. Novamente, usaremos o método `extend` para registrar nosso código personalizado:

```
Session::extend('mongo', function($app)
{
	// Return implementation of SessionHandlerInterface
});
```

Observe que nosso driver de cache personalizado deve implementar a `SessionHandlerInterface`. Essa interface está incluída no núcleo do PHP 5.4+. Se você estiver usando o PHP 5.3, a interface será definida para você pelo Laravel para garantir compatibilidade futura. Essa interface contém apenas alguns métodos simples que precisamos implementar. Uma implementação stubbed do MongoDB seria algo como isso:

```
class MongoHandler implements SessionHandlerInterface {

	public function open($savePath, $sessionName) {}
	public function close() {}
	public function read($sessionId) {}
	public function write($sessionId, $data) {}
	public function destroy($sessionId) {}
	public function gc($lifetime) {}

}	
```

Como esses métodos não são tão fáceis de entender quanto a interface de cache `StoreInterface`, vamos rapidamente explicar o que cada um deles faz:

- O método `open` seria tipicamente usado em sistemas de armazenamento de sessão baseados em arquivos. Como o Laravel vem com um driver de sessão `native` que usa o armazenamento de arquivos nativo do PHP para sessões, você quase nunca precisará colocar algo neste método. Você pode deixá-lo como um stub vazio. É simplesmente um fato do mau design da interface (que discutiremos mais tarde) que o PHP nos obriga a implementar este método.
- O método `close`, assim como o método `open`, geralmente também pode ser ignorado. Para a maioria dos drivers, ele não é necessário.
- O método `read` deve retornar a versão em string dos dados da sessão associados ao `$sessionId` fornecido. Não há necessidade de realizar qualquer serialização ou outra codificação ao recuperar ou armazenar dados da sessão no seu driver, pois o Laravel fará a serialização por você.
- O método `write` deve escrever a string `$data` fornecida associada ao `$sessionId` em algum sistema de armazenamento persistente, como MongoDB, Dynamo, etc.
- O método `destroy` deve remover os dados associados ao `$sessionId` do armazenamento persistente.
- O método `gc` deve destruir todos os dados da sessão que tiverem uma idade maior que o valor fornecido `$lifetime`, que é um timestamp UNIX. Para sistemas com expiração automática, como Memcached e Redis, este método pode ser deixado em branco.

Depois que a `SessionHandlerInterface` tiver sido implementada, estamos prontos para registrá-la com o gerenciador de sessão:

```
Session::extend('mongo', function($app)
{
	return new MongoHandler;
});
```

Depois que o driver de sessão tiver sido registrado, podemos usar o driver `mongo` em nosso arquivo de configuração `app/config/session.php`.

> **Nota:**
>
>  Lembre-se, se você escrever um manipulador de sessão personalizado, compartilhe-o no Packagist!

<a name="autenticação"></a>

## Autenticação

A autenticação pode ser estendida da mesma forma que as facilidades de cache e sessão. Novamente, usaremos o método `extend` com o qual nos familiarizamos:

```
Auth::extend('riak', function($app)
{
	// Return implementation of Illuminate\Auth\UserProviderInterface
});
```

As implementações da `UserProviderInterface` são responsáveis apenas por buscar uma implementação da `UserInterface` de um sistema de armazenamento persistente, como MySQL, Riak, etc. Essas duas interfaces permitem que os mecanismos de autenticação do Laravel continuem funcionando, independentemente de como os dados do usuário são armazenados ou qual tipo de classe é usada para representá-los.

Vamos dar uma olhada na `UserProviderInterface`:

```
interface UserProviderInterface {

	public function retrieveById($identifier);
	public function retrieveByCredentials(array $credentials);
	public function validateCredentials(UserInterface $user, array $credentials);

}
```

A função `retrieveById` normalmente recebe uma chave numérica que representa o usuário, como um ID de autoincremento de um banco de dados MySQL. A implementação da `UserInterface` que corresponda ao ID deve ser recuperada e devolvida pelo método.

O método `retrieveByCredentials` recebe a matriz de credenciais passadas para o método `Auth::attempt` ao tentar fazer login em uma aplicação. O método deve, então, "consultar" o armazenamento persistente subjacente para encontrar o usuário que corresponda a essas credenciais. Normalmente, esse método executará uma consulta com uma condição `where` em `$credentails['username']`. **Esse método não deve tentar realizar nenhuma validação ou autenticação de senha.**

O método `validateCredentials` deve comparar o `$user` fornecido com os `$credentials` para autenticar o usuário. Por exemplo, esse método pode comparar a string `$user->getAuthPassword()` com um `Hash::make` de `$credentials['password']`.

Agora que exploramos cada um dos métodos na `UserProviderInterface`, vamos dar uma olhada na `UserInterface`. Lembre-se, o provedor deve retornar implementações dessa interface dos métodos `retrieveById` e `retrieveByCredentials`:

```
interface UserInterface {

	public function getAuthIdentifier();
	public function getAuthPassword();

}
```

Essa interface é simples. O método `getAuthIdentifier` deve retornar a "chave primária" do usuário. Novamente, em um back-end MySQL, isso seria a chave primária que incrementa automaticamente. O `getAuthPassword` deve retornar a senha criptografada do usuário. Essa interface permite que o sistema de autenticação trabalhe com qualquer classe `User`, independentemente da ORM ou camada de abstração de armazenamento que você esteja usando. Por padrão, o Laravel inclui uma classe `User` no diretório `app/models` que implementa essa interface, então você pode consultar essa classe para um exemplo de implementação.

Por fim, depois de implementarmos a `UserProviderInterface`, estamos prontos para registrar nossa extensão com a fachada `Auth`:

```
Auth::extend('riak', function($app)
{
	return new RiakUserProvider($app['riak.connection']);
});
```

Depois de registrar o driver com o método `extend`, você passa para o novo driver no arquivo de configuração `app/config/auth.php`.

<a name="ioc-based-extension"></a>

## Extensão baseada em IoC

Quase todos os provedores de serviços incluídos no framework Laravel ligam objetos ao contêiner de IoC. Você pode encontrar uma lista dos provedores de serviços da sua aplicação no arquivo de configuração `app/config/app.php`. À medida que tiver tempo, você deve examinar o código-fonte de cada um desses provedores. Ao fazer isso, você terá uma compreensão muito melhor do que cada provedor adiciona ao framework, bem como quais chaves são usadas para ligar vários serviços ao contêiner de IoC.

Por exemplo, o `PaginationServiceProvider` vincula uma chave `paginator` no contêiner de IoC, que resolve em uma instância de `Illuminate\Pagination\Environment`. Você pode facilmente estender e sobrescrever essa classe dentro de sua própria aplicação sobrescrevendo essa vinculação de IoC. Por exemplo, você poderia criar uma classe que estenda a classe base `Environment`:

```
namespace Snappy\Extensions\Pagination;

class Environment extends \Illuminate\Pagination\Environment {

	//

}
```

Depois de criar a extensão da classe, você pode criar uma nova classe de provedor de serviço `SnappyPaginationProvider` que sobrescreve o paginador em seu método `boot`:

```
class SnappyPaginationProvider extends PaginationServiceProvider {

	public function boot()
	{
		App::bind('paginator', function()
		{
			return new Snappy\Extensions\Pagination\Environment;
		});

		parent::boot();
	}

}
```

Observe que essa classe estende o `PaginationServiceProvider`, e não a classe base padrão `ServiceProvider`. Depois de ter estendido o provedor de serviços, substitua o `PaginationServiceProvider` no arquivo de configuração `app/config/app.php` pelo nome do provedor que você estendeu.

Este é o método geral para estender qualquer classe principal vinculada no contêiner. Essencialmente, todas as classes principais são vinculadas no contêiner dessa maneira e podem ser sobrescritas. Novamente, ler os provedores de serviços de framework incluídos o familiarizará com onde várias classes são vinculadas ao contêiner e por quais chaves elas são vinculadas. Esta é uma ótima maneira de aprender mais sobre como o Laravel é montado.

<a name="solicitação-de-extensão"></a>

## Solicitar Extensão

Como é uma peça fundamental do framework e é instanciada muito cedo no ciclo de solicitação, estender a classe `Request` funciona um pouco diferente dos exemplos anteriores.

Primeiro, estenda a classe como de costume:

```
<?php namespace QuickBill\Extensions;

class Request extends \Illuminate\Http\Request {

	// Custom, helpful methods here...

}
```

Depois de estender a classe, abra o arquivo `bootstrap/start.php`. Este arquivo é um dos primeiros arquivos a serem incluídos em cada solicitação à sua aplicação. Observe que a primeira ação realizada é a criação da instância do Laravel `$app`:

```
$app = new \Illuminate\Foundation\Application;
```

Quando uma nova instância de aplicativo é criada, ela cria uma nova instância de `Illuminate\Http\Request` e a vincula ao contêiner IoC usando a chave `request`. Então, precisamos de uma maneira de especificar uma classe personalizada que deve ser usada como o tipo de solicitação "padrão", certo? E, felizmente, o método `requestClass` na instância do aplicativo faz exatamente isso! Então, podemos adicionar essa linha no topo do nosso arquivo `bootstrap/start.php`:

```
use Illuminate\Foundation\Application;

Application::requestClass('QuickBill\Extensions\Request');
```

Depois de especificar a classe de solicitação personalizada, o Laravel usará essa classe sempre que criar uma instância de `Request`, permitindo que você sempre tenha uma instância da sua classe de solicitação personalizada disponível, mesmo em testes unitários!
