# Funções Auxiliares

- [Arrays](#arrays)
- [Caminhos](#caminhos)
- [Strings](#strings)
- [URLs](#urls)
- [Outros](#outros)

<a name="arrays"></a>

## Arrays

### array\_add

A função `array_add` adiciona um par de chave/valor dado ao array se a chave dada não existir no array.

```
$array = array('foo' => 'bar');

$array = array_add($array, 'key', 'value');
```

### array\_divide

A função `array_divide` retorna dois arrays, um contendo as chaves e outro contendo os valores da matriz original.

```
$array = array('foo' => 'bar');

list($keys, $values) = array_divide($array);
```

### array\_dot

A função `array_dot` compacta uma matriz multidimensional em uma matriz de nível único que utiliza a notação "ponto" para indicar a profundidade.

```
$array = array('foo' => array('bar' => 'baz'));

$array = array_dot($array);

// array('foo.bar' => 'baz');
```

### array\_except

O método `array_except` remove os pares de chave/valor fornecidos da matriz.

```
$array = array_except($array, array('keys', 'to', 'remove'));
```

### array\_fetch

O método `array_fetch` retorna um array achatado contendo o elemento aninhado selecionado.

```
$array = array(
	array('developer' => array('name' => 'Taylor')),
	array('developer' => array('name' => 'Dayle')),
);

$array = array_fetch($array, 'developer.name');

// array('Taylor', 'Dayle');
```

### array\_first

O método `array_first` retorna o primeiro elemento de um array que passa por um teste de verdade dado.

```
$array = array(100, 200, 300);

$value = array_first($array, function($key, $value)
{
	return $value >= 150;
});
```

Um valor padrão também pode ser passado como o terceiro parâmetro:

```
$value = array_first($array, $callback, $default);
```

### array\_flatten

O método `array_flatten` irá transformar uma matriz multidimensional em uma única camada.

```
$array = array('name' => 'Joe', 'languages' => array('PHP', 'Ruby'));

$array = array_flatten($array);

// array('Joe', 'PHP', 'Ruby');
```

### array\_forget

O método `array_forget` removerá um par de chave/valor dado de um array profundamente aninhado usando a notação "ponto".

```
$array = array('names' => array('joe' => array('programmer')));

$array = array_forget($array, 'names.joe');
```

### array\_get

O método `array_get` recuperará um valor específico de uma matriz profundamente aninhada usando a notação "ponto".

```
$array = array('names' => array('joe' => array('programmer')));

$value = array_get($array, 'names.joe');
```

### array\_only

O método `array_only` retornará apenas os pares de chave/valor especificados da matriz.

```
$array = array('name' => 'Joe', 'age' => 27, 'votes' => 1);

$array = array_only($array, array('name', 'votes'));
```

### array\_pluck

O método `array_pluck` extrairá uma lista de pares chave/valor fornecidos da matriz.

```
$array = array(array('name' => 'Taylor'), array('name' => 'Dayle'));

$array = array_pluck($array, 'name');

// array('Taylor', 'Dayle');
```

### array\_pull

O método `array_pull` retornará um par chave/valor específico da matriz, além de removê-lo.

```
$array = array('name' => 'Taylor', 'age' => 27);

$name = array_pull($array, 'name');
```

### array\_set

O método `array_set` define um valor dentro de uma matriz profundamente aninhada usando a notação "ponto".

```
$array = array('names' => array('programmer' => 'Joe'));

array_set($array, 'names.editor', 'Taylor');
```

### array\_sort

O método `array_sort` ordena o array pelos resultados da função fechada fornecida.

```
$array = array(
	array('name' => 'Jill'),
	array('name' => 'Barry'),
);

$array = array_values(array_sort($array, function($value)
{
	return $value['name'];
}));
```

### cabeça

Retorne o primeiro elemento da matriz. Útil para a cadeia de métodos no PHP 5.3.x.

```
$first = head($this->returnsArray('foo'));
```

### último

Retorne o último elemento da matriz. Útil para a cadeia de métodos.

```
$last = last($this->returnsArray('foo'));
```

<a name="caminhos"></a>

## Caminhos

### app\_path

Obtenha o caminho totalmente qualificado para o diretório `app`.

### caminho\_base

Obtenha o caminho totalmente qualificado até a raiz da instalação do aplicativo.

### caminho\_público

Obtenha o caminho totalmente qualificado para o diretório `public`.

### caminho\_de\_armazenamento

Obtenha o caminho totalmente qualificado para o diretório `app/storage`.

<a name="strings"></a>

## Cordas

### camel\_case

Converta a string fornecida para `camelCase`.

```
$camel = camel_case('foo_bar');

// fooBar
```

### basename\_da\_classe

Obtenha o nome da classe da classe especificada, sem nomes de namespaces.

```
$class = class_basename('Foo\Bar\Baz');

// Baz
```

### e

Execute `htmlentities` sobre a string fornecida, com suporte ao UTF-8.

```
$entities = e('<html>foo</html>');
```

### termina com

Determinar se o pasto dado termina com uma agulha dada.

```
$value = ends_with('This is my name', 'name');
```

### snake\_case

Converta a string fornecida para `snake_case`.

```
$snake = snake_case('fooBar');

// foo_bar
```

### começa com

Determine se o pasto dado começa com a agulha dada.

```
$value = starts_with('This is my name', 'This');
```

### str\_contains

Determinar se o pasto dado contém a agulha dada.

```
$value = str_contains('This is my name', 'my');
```

### str\_finish

Adicione uma única instância da agulha fornecida ao porão. Remova quaisquer instâncias extras.

```
$string = str_finish('this/string', '/');

// this/string/
```

### str\_is

Determine se uma string dada corresponde a um padrão dado. Os asteriscos podem ser usados para indicar caracteres curinga.

```
$value = str_is('foo*', 'foobar');
```

### str\_plural

Converta uma string para sua forma plural (apenas em inglês).

```
$plural = str_plural('car');
```

### str\_random

Gerar uma string aleatória da determinada extensão.

```
$string = str_random(40);
```

### str\_singular

Converta uma string para sua forma singular (apenas em inglês).

```
$singular = str_singular('cars');
```

### studly\_case

Converta a string fornecida para `StudlyCase`.

```
$value = studly_case('foo_bar');

// FooBar
```

### trans

Traduza uma linha de idioma específica. Alias de `Lang::get`.

```
$value = trans('validation.required'):
```

### trans\_choice

Traduza uma linha de idioma específica com flexão. Alias de `Lang::choice`.

```
$value = trans_choice('foo.bar', $count);
```

<a name="urls"></a>

## URLs

### ação

Gerar um URL para uma ação de controle dada.

```
$url = action('HomeController@getIndex', $params);
```

### rota

Gerar um URL para uma rota nomeada específica.

```
$url = route('routeName', $params);
```

### ativo

Gerar um URL para um ativo.

```
$url = asset('img/photo.jpg');
```

### link\_to

Gerar um link HTML para o URL fornecido.

```
echo link_to('foo/bar', $title, $attributes = array(), $secure = null);
```

### link\_to\_asset

Gerar um link HTML para o ativo fornecido.

```
echo link_to_asset('foo/bar.zip', $title, $attributes = array(), $secure = null);
```

### link\_to\_route

Gerar um link HTML para a rota fornecida.

```
echo link_to_route('route.name', $title, $parameters = array(), $attributes = array());
```

### link\_to\_action

Gerar um link HTML para a ação do controlador fornecida.

```
echo link_to_action('HomeController@getIndex', $title, $parameters = array(), $attributes = array());
```

### ativo\_seguro

Gerar um link HTML para o ativo fornecido usando HTTPS.

```
echo secure_asset('foo/bar.zip', $title, $attributes = array());
```

### secure\_url

Gerar um URL totalmente qualificado para um caminho específico usando HTTPS.

```
echo secure_url('foo/bar', $parameters = array());
```

### url

Gerar um URL totalmente qualificado para o caminho fornecido.

```
echo url('foo/bar', $parameters = array(), $secure = null);
```

<a name="diversos"></a>

## Outros

### csrf\_token

Obtenha o valor do token CSRF atual.

```
$token = csrf_token();
```

### dd

Descarte a variável fornecida e termine a execução do script.

```
dd($value);
```

### valor

Se o valor fornecido for uma `Closure`, retorne o valor retornado pela `Closure`. Caso contrário, retorne o valor.

```
$value = value(function() { return 'bar'; });
```

### com

Retorne o objeto fornecido. Útil para a criação de cadeias de métodos no PHP 5.3.x.

```
$value = with(new Foo)->doWork();
```
