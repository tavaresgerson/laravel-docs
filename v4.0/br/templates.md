# Modelos

- [Layouts de controladores](#layouts-de-controladores)
- [Modelagem de lâminas](#modelagem-de-las)
- [Outras estruturas de controle de lâminas](#outras-estruturas-de-controle-de-laminas)

<a name="controller-layouts"></a>

## Layouts de controlador

Um método de usar modelos no Laravel é através dos layouts de controladores. Ao especificar a propriedade `layout` no controlador, a visualização especificada será criada automaticamente e será a resposta assumida que deve ser retornada a partir das ações.

**Definindo um layout em um controlador**

```
class UserController extends BaseController {

	/**
	 * The layout that should be used for responses.
	 */
	protected $layout = 'layouts.master';

	/**
	 * Show the user profile.
	 */
	public function showProfile()
	{
		$this->layout->content = View::make('user.profile');
	}

}
```

<a name="blade-templating"></a>

## Modelagem de lâminas

Blade é um motor de modelagem simples, mas poderoso, fornecido com o Laravel. Ao contrário dos layouts de controladores, o Blade é impulsionado pela *herança de modelos* e *seções*. Todos os modelos Blade devem usar a extensão `.blade.php`.

**Definindo um Layout de Espada**

```
<!-- Stored in app/views/layouts/master.blade.php -->

<html>
	<body>
		@section('sidebar')
			This is the master sidebar.
		@show

		<div class="container">
			@yield('content')
		</div>
	</body>
</html>
```

**Usando um layout de lâmina**

```
@extends('layouts.master')

@section('sidebar')
	@parent

	<p>This is appended to the master sidebar.</p>
@stop

@section('content')
	<p>This is my body content.</p>
@stop
```

Observe que as vistas que `extend`m um layout Blade simplesmente substituem seções do layout. O conteúdo do layout pode ser incluído em uma vista filho usando a diretiva `@parent` em uma seção, permitindo que você adicione ao conteúdo de uma seção do layout, como uma barra lateral ou rodapé.

Às vezes, como quando você não tem certeza se uma seção foi definida, você pode querer passar um valor padrão para a diretiva `@yield`. Você pode passar o valor padrão como o segundo argumento:

```
@yield('section', 'Default Content');
```

<a name="outras-estruturas-de-controle-de-lâminas"></a>

## Outras estruturas de controle de lâminas

**Echoando Dados**

```
Hello, {{ $name }}.

The current UNIX timestamp is {{ time() }}.
```

Se você precisar exibir uma string que está envolvida em chaves angulares, você pode escapar do comportamento do Blade, prefixando seu texto com o símbolo `@`:

**Exibindo texto bruto com chaves curvas**

```
@{{ This will not be processed by Blade }}
```

Claro, todos os dados fornecidos pelo usuário devem ser escapados ou purificados. Para escapar a saída, você pode usar a sintaxe de chaves duplas:

```
Hello, {{{ $name }}}.
```

> **Observação:**
>
>  Tenha muito cuidado ao replicar conteúdo fornecido por usuários de sua aplicação. Sempre use a sintaxe de chaves duplas para escapar de quaisquer entidades HTML no conteúdo.

**Instruções `if`**

```
@if (count($records) === 1)
	I have one record!
@elseif (count($records) > 1)
	I have multiple records!
@else
	I don't have any records!
@endif

@unless (Auth::check())
	You are not signed in.
@endunless
```

**Loops**

```
@for ($i = 0; $i < 10; $i++)
	The current value is {{ $i }}
@endfor

@foreach ($users as $user)
	<p>This is user {{ $user->id }}</p>
@endforeach

@while (true)
	<p>I'm looping forever.</p>
@endwhile
```

**Incluindo subvisões**

```
@include('view.name')
```

Você também pode passar um array de dados para a visualização incluída:

```
@include('view.name', array('some'=>'data'))
```

**Sobreposição de seções**

Por padrão, as seções são anexadas a qualquer conteúdo anterior que exista na seção. Para sobrescrever uma seção inteiramente, você pode usar a instrução `overwrite`:

```
@extends('list.item.container')

@section('list.item.content')
	<p>This is an item of type {{ $item->type }}</p>
@overwrite
```

**Exibindo Linhas de Idioma**

```
@lang('language.line')

@choice('language.line', 1);
```

**Comentários**

```
{{-- This comment will not be in the rendered HTML --}}
```
