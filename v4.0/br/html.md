# Formulários e HTML

- [Abrir um formulário](#abrir-um-formulário)
- Proteção contra CSRF]\(#csrf-protection)
- [Vinculação de Modelos de Formulário](#form-model-binding)
- [Labels](#labels)
- [Texto, Área de Texto, Senha e Campos Ocultas](#texto)
- [Caixa de seleção e botões de rádio](#caixas-de-selecao-e-botons-de-radio)
- [Entrada de arquivo](#entrada-de-arquivo)
- Listas suspensas (#drop-down-lists)
- Botões]\(#botões)
- [Macros Personalizadas](#macros-personalizadas)
- [Gerando URLs](#gerando-urls)

<a name="abertura-de-um-formulário"></a>

## Abrir um formulário

**Abrir um formulário**

```
{{ Form::open(array('url' => 'foo/bar')) }}
	//
{{ Form::close() }}
```

Por padrão, será assumido um método `POST`; no entanto, você pode especificar outro método:

```
echo Form::open(array('url' => 'foo/bar', 'method' => 'put'))
```

> **Observação:**
>
>  Como os formulários HTML só suportam os métodos 
>
> `POST`
>
>  e 
>
> `GET`
>
> , os métodos 
>
> `PUT`
>
>  e 
>
> `DELETE`
>
>  serão falsificados ao adicionar automaticamente um campo oculto 
>
> `_method`
>
>  ao seu formulário.

Você também pode abrir formulários que apontam para rotas nomeadas ou ações de controle:

```
echo Form::open(array('route' => 'route.name'))

echo Form::open(array('action' => 'Controller@method'))
```

Você pode passar parâmetros de rota também:

```
echo Form::open(array('route' => array('route.name', $user->id)))

echo Form::open(array('action' => array('Controller@method', $user->id)))
```

Se o seu formulário vai aceitar uploads de arquivos, adicione uma opção `files` ao seu array:

```
echo Form::open(array('url' => 'foo/bar', 'files' => true))
```

<a name="proteção-contra-CSRF"></a>

## Proteção contra CSRF

O Laravel oferece um método fácil para proteger sua aplicação contra falsificações de solicitações entre sites. Primeiro, um token aleatório é colocado na sessão do usuário. Não se preocupe, isso é feito automaticamente. O token CSRF será adicionado às suas formulários como um campo oculto automaticamente. No entanto, se você deseja gerar o HTML para o campo oculto, pode usar o método `token`:

**Adicionando o Token CSRF a um formulário**

```
echo Form::token();
```

**Anexar o Filtro CSRF a uma Rota**

```
Route::post('profile', array('before' => 'csrf', function()
{
	//
}));
```

<a name="form-model-binding"></a>

## Modelo de Formulário de Ligação

Muitas vezes, você vai querer preencher um formulário com base no conteúdo de um modelo. Para fazer isso, use o método `Form::model`:

**Abrindo um formulário modelo**

```
echo Form::model($user, array('route' => array('user.update', $user->id)))
```

Agora, quando você gera um elemento de formulário, como um campo de entrada de texto, o valor do modelo que corresponde ao nome do campo será automaticamente definido como o valor do campo. Por exemplo, para um campo de entrada de texto chamado `email`, o atributo `email` do modelo do usuário seria definido como o valor. No entanto, há mais! Se houver um item nos dados de flash da sessão que corresponda ao nome do campo, ele terá precedência sobre o valor do modelo. Portanto, a prioridade é a seguinte:

1. Dados de Flash de Sessão (Entrada Antiga)
2. Valor explicitamente passado
3. Dados de atributos do modelo

Isso permite que você crie formulários rapidamente que não apenas se conectam aos valores do modelo, mas também sejam facilmente preenchidos novamente se houver um erro de validação no servidor!

> **Observação:**
>
>  Ao usar 
>
> `Form::model`
>
> , não se esqueça de fechar seu formulário com 
>
> `Form::close`
>
> !

<a name="etiquetas"></a>

## Etiqueta

**Gerando um Elemento de Rótulo**

```
echo Form::label('email', 'E-Mail Address');
```

**Especificando atributos HTML extras**

```
echo Form::label('email', 'E-Mail Address', array('class' => 'awesome'));
```

> **Observação:**
>
>  Após criar uma etiqueta, qualquer elemento do formulário que você criar com um nome que corresponda ao nome da etiqueta receberá automaticamente uma ID que corresponda ao nome da etiqueta também.

<a name="texto"></a>

## Texto, Área de Texto, Senha e Campos Ocultas

**Gerando uma Entrada de Texto**

```
echo Form::text('username');
```

**Especificando um valor padrão**

```
echo Form::text('email', 'example@gmail.com');
```

> **Observação:**
>
>  Os métodos 
>
> *hidden*
>
>  e 
>
> *textarea*
>
>  têm a mesma assinatura que o método 
>
> *text*
>
> .

**Gerando uma senha**

```
echo Form::password('password');
```

**Gerando Outros Inputs**

```
echo Form::email($name, $value = null, $attributes = array());
echo Form::file($name, $attributes = array());
```

<a name="checkboxes-e-radio-buttons"></a>

## Caixa de seleção e Botões de rádio

**Gerando um Checkbox ou Entrada de Rádio**

```
echo Form::checkbox('name', 'value');

echo Form::radio('name', 'value');
```

**Gerando um Checkbox ou Campo de Entrada de Rádio Marcado**

```
echo Form::checkbox('name', 'value', true);

echo Form::radio('name', 'value', true);
```

<a name="file-input"></a>

## Entrada de arquivo

**Gerando uma entrada de arquivo**

```
echo Form::file('image');
```

<a name="listas-deslizantes"></a>

## Listas suspensivas

**Gerando uma Lista de Seleção Dinâmica**

```
echo Form::select('size', array('L' => 'Large', 'S' => 'Small'));
```

**Gerando uma lista suspensa com o valor padrão selecionado**

```
echo Form::select('size', array('L' => 'Large', 'S' => 'Small'), 'S');
```

**Gerando uma Lista Gruppada**

```
echo Form::select('animal', array(
	'Cats' => array('leopard' => 'Leopard'),
	'Dogs' => array('spaniel' => 'Spaniel'),
));
```

**Gerando uma lista suspensa com uma faixa**

```
echo Form::selectRange('number', 10, 20);
```

**Gerando uma lista com os nomes dos meses**

```
echo Form::selectMonth('month');
```

<a name="botões"></a>

## Botões

**Gerando um Botão de Submissão**

```
echo Form::submit('Click Me!');
```

> **Observação:**
>
>  Precisa criar um elemento de botão? Experimente o método 
>
> *button*
>
> . Ele tem a mesma assinatura que 
>
> *submit*
>
> .

<a name="macros personalizados"></a>

## Macros Personalizadas

É fácil definir seus próprios ajudantes de classe de formulário personalizados, chamados de "macros". Veja como funciona. Primeiro, basta registrar o macro com um nome dado e uma Closure:

**Registrando um macro de formulário**

```
Form::macro('myField', function()
{
	return '<input type="awesome">';
});
```

Agora você pode chamar sua macro usando seu nome:

**Chamando um macro de formulário personalizado**

```
echo Form::myField();
```

<a name="gerando-urls"></a>

Para obter mais informações sobre a geração de URLs, consulte a documentação sobre [ajudantes](/docs/ajudantes#urls).
