# Testes Unitários

- [Introdução](#introdução)
- [Definindo e executando testes](#definindo-e-executando-testes)
- [Ambiente de teste](#ambiente-de-teste)
- [Chamadas de rotas a partir de testes](#chamadas-de-rotas-a-partir-de-testes)
- [Fachadas parodiadas](#fachadas-parodiadas)
- [Afirmações do quadro](#afirmacoes-do-quadro)
- Métodos auxiliares (#helper-methods)

<a name="introdução"></a>

## Introdução

O Laravel é construído com o teste unitário em mente. De fato, o suporte para testes com PHPUnit está incluído por padrão, e um arquivo `phpunit.xml` já está configurado para sua aplicação. Além do PHPUnit, o Laravel também utiliza os componentes Symfony HttpKernel, DomCrawler e BrowserKit para permitir que você inspecione e manipule suas visualizações durante os testes, permitindo simular um navegador da web.

Um arquivo de teste de exemplo está disponível no diretório `app/tests`. Após instalar uma nova aplicação Laravel, basta executar `phpunit` na linha de comando para executar seus testes.

<a name="definindo-e-executando-testes"></a>

## Definindo e executando testes

Para criar um caso de teste, basta criar um novo arquivo de teste no diretório `app/tests`. A classe de teste deve estender `TestCase`. Em seguida, você pode definir métodos de teste como faria normalmente ao usar o PHPUnit.

**Uma Classe de Teste de Exemplo**

```
class FooTest extends TestCase {

	public function testSomethingIsTrue()
	{
		$this->assertTrue(true);
	}

}
```

Você pode executar todos os testes da sua aplicação executando o comando `phpunit` no seu terminal.

> **Observação:**
>
>  Se você definir seu próprio método 
>
> `setUp`
>
> , não se esqueça de chamar 
>
> `parent::setUp`
>
> .

<a name="ambiente-de-teste"></a>

## Ambiente de Teste

Ao executar testes unitários, o Laravel configurará automaticamente o ambiente de configuração para `testing`. Além disso, o Laravel inclui arquivos de configuração para `session` e `cache` no ambiente de teste. Ambos esses drivers estão configurados para `array` no ambiente de teste, o que significa que nenhum dado de sessão ou cache será persistido durante o teste. Você é livre para criar outras configurações de ambiente de teste conforme necessário.

<a name="rotas-de-chamada-de-testes"></a>

## Chamadas de rotas a partir de testes

Você pode facilmente chamar uma de suas rotas para um teste usando o método `call`:

**Chamando uma rota a partir de um teste**

```
$response = $this->call('GET', 'user/profile');

$response = $this->call($method, $uri, $parameters, $files, $server, $content);
```

Você pode, então, inspecionar o objeto `Illuminate\Http\Response`:

```
$this->assertEquals('Hello World', $response->getContent());
```

Você também pode chamar um controlador a partir de um teste:

**Chamando um Controlador de um Teste**

```
$response = $this->action('GET', 'HomeController@index');

$response = $this->action('GET', 'UserController@profile', array('user' => 1));
```

O método `getContent` retornará o conteúdo da string avaliada da resposta. Se sua rota retornar uma `View`, você pode acessá-la usando a propriedade `original`:

```
$view = $response->original;

$this->assertEquals('John', $view['name']);
```

Para chamar uma rota HTTPS, você pode usar o método `callSecure`:

```
$response = $this->callSecure('GET', 'foo/bar');
```

> **Nota:**
>
>  Os filtros de rota estão desativados quando você está no ambiente de teste. Para ativá-los, adicione 
>
> `Route::enableFilters()`
>
>  ao seu teste.

### DOM Crawler

Você também pode chamar uma rota e receber uma instância do DOM Crawler que você pode usar para inspecionar o conteúdo:

```
$crawler = $this->client->request('GET', '/');

$this->assertTrue($this->client->getResponse()->isOk());

$this->assertCount(1, $crawler->filter('h1:contains("Hello World!")'));
```

Para mais informações sobre como usar o rastreador, consulte sua [documentação oficial](http://symfony.com/doc/master/components/dom_crawler.html).

<a name="mocking-facades"></a>

## Fachadas zombeteiras

Durante as testes, você pode querer simular uma chamada a uma fachada estática do Laravel. Por exemplo, considere a seguinte ação do controlador:

```
public function getIndex()
{
	Event::fire('foo', array('name' => 'Dayle'));

	return 'All done!';
}
```

Podemos fazer piadas com a chamada para a classe `Event` usando o método `shouldReceive` na fachada, que retornará uma instância de um mock do [Mockery](https://github.com/padraic/mockery).

**Imitando uma fachada**

```
public function testGetIndex()
{
	Event::shouldReceive('fire')->once()->with('foo', array('name' => 'Dayle'));

	$this->call('GET', '/');
}
```

> **Observação:**
>
>  Você não deve zombar da fachada 
>
> `Request`
>
> . Em vez disso, passe a entrada desejada para o método 
>
> `call`
>
>  ao executar seu teste.

<a name="framework-assertions"></a>

## Afirmações de quadro

O Laravel vem com vários métodos `assert` para tornar o teste um pouco mais fácil:

**As respostas assertivas estão bem-vindas**

```
public function testMethod()
{
	$this->call('GET', '/');

	$this->assertResponseOk();
}
```

**Afastando Status de Resposta**

```
$this->assertResponseStatus(403);
```

**As respostas afirmativas são redirecionamentos**

```
$this->assertRedirectedTo('foo');

$this->assertRedirectedToRoute('route.name');

$this->assertRedirectedToAction('Controller@method');
```

**Afirmar que uma visão tem alguns dados**

```
public function testMethod()
{
	$this->call('GET', '/');

	$this->assertViewHas('name');
	$this->assertViewHas('age', $value);
}
```

**Afirmando que a sessão tem alguns dados**

```
public function testMethod()
{
	$this->call('GET', '/');

	$this->assertSessionHas('name');
	$this->assertSessionHas('age', $value);
}
```

<a name="helper-methods"></a>

## Métodos auxiliares

A classe `TestCase` contém vários métodos auxiliares para facilitar o teste da sua aplicação.

Você pode definir o usuário autenticado atualmente usando o método `be`:

**Definindo o Usuário Atualmente Autenticado**

```
$user = new User(array('name' => 'John'));

$this->be($user);
```

Você pode re-sematizar seu banco de dados a partir de um teste usando o método `seed`:

**Re-semente a base de dados a partir de testes**

```
$this->seed();

$this->seed($connection);
```

Mais informações sobre como criar sementes podem ser encontradas na seção [migrações e semeadura](/docs/migrations#database-seeding) da documentação.
