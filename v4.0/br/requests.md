# Pedidos e entrada

- [Entrada Básica](#entrada-básica)
- [Cookies](#cookies)
- [Antigo Input](#antigo-input)
- [Arquivos](#arquivos)
- [Solicitar Informações](#solicitar-informações)

<a name="input-básico"></a>

## Entrada básica

Você pode acessar todas as entradas do usuário com alguns métodos simples. Você não precisa se preocupar com o verbo HTTP usado para a solicitação, pois as entradas são acessadas da mesma maneira para todos os verbos.

**Recuperando um valor de entrada**

```
$name = Input::get('name');
```

**Recuperando um valor padrão se o valor de entrada estiver ausente**

```
$name = Input::get('name', 'Sally');
```

**Determinando se um valor de entrada está presente**

```
if (Input::has('name'))
{
	//
}
```

**Obtendo todos os dados para a solicitação**

```
$input = Input::all();
```

**Obtendo apenas parte da entrada do pedido**

```
$input = Input::only('username', 'password');

$input = Input::except('credit_card');
```

Ao trabalhar com formulários com entradas de "matriz", você pode usar a notação de ponto para acessar as matrizes:

```
$input = Input::get('products.0.name');
```

> **Observação:**
>
>  Algumas bibliotecas JavaScript, como o Backbone, podem enviar dados de entrada para a aplicação como JSON. Você pode acessar esses dados via 
>
> `Input::get`
>
>  como de costume.

<a name="cookies"></a>

## Cookies

Todos os cookies criados pelo framework Laravel são criptografados e assinados com um código de autenticação, o que significa que serão considerados inválidos se forem alterados pelo cliente.

**Recuperando um valor de cookie**

```
$value = Cookie::get('name');
```

**Anexar um novo cookie a uma resposta**

```
$response = Response::make('Hello World');

$response->withCookie(Cookie::make('name', 'value', $minutes));
```

**Aguardar um cookie para a próxima resposta**

Se você quiser definir um cookie antes que uma resposta seja criada, use o método `Cookie::queue()`. O cookie será automaticamente anexado à resposta final da sua aplicação.

```
Cookie::queue($name, $value, $minutes);
```

**Criando um Biscoito que Dura para Sempre**

```
$cookie = Cookie::forever('name', 'value');
```

<a name="antigo-input"></a>

## Velho Input

Você pode precisar manter os dados de um pedido até o próximo pedido. Por exemplo, você pode precisar repopular um formulário após verificar se há erros de validação.

**Entrada intermitente para a sessão**

```
Input::flash();
```

**Apenas alguns inputs são exibidos na sessão**

```
Input::flashOnly('username', 'email');

Input::flashExcept('password');
```

Como você geralmente deseja exibir o campo de entrada em associação com um redirecionamento para a página anterior, você pode facilmente concatenar a exibição do campo de entrada com um redirecionamento.

```
return Redirect::to('form')->withInput();

return Redirect::to('form')->withInput(Input::except('password'));
```

> **Observação:**
>
>  Você pode transmitir outros dados entre solicitações usando a classe 
>
> [Session](/docs/session)
>
> .

**Recuperação de Dados Antigos**

```
Input::old('username');
```

<a name="arquivos"></a>

## Arquivos

**Recuperando um arquivo carregado**

```
$file = Input::file('photo');
```

**Determinando se um arquivo foi carregado**

```
if (Input::hasFile('photo'))
{
	//
}
```

O objeto retornado pelo método `file` é uma instância da classe `Symfony\Component\HttpFoundation\File\UploadedFile`, que estende a classe `SplFileInfo` do PHP e fornece uma variedade de métodos para interagir com o arquivo.

**Movendo um arquivo carregado**

```
Input::file('photo')->move($destinationPath);

Input::file('photo')->move($destinationPath, $fileName);
```

**Recuperando o caminho para um arquivo carregado**

```
$path = Input::file('photo')->getRealPath();
```

**Recuperando o Nome Original de um Arquivo Carregado**

```
$name = Input::file('photo')->getClientOriginalName();
```

**Recuperando a extensão de um arquivo carregado**

```
$extension = Input::file('photo')->getClientOriginalExtension();
```

**Recuperando o tamanho de um arquivo carregado**

```
$size = Input::file('photo')->getSize();
```

**Recuperando o Tipo MIME de um arquivo carregado**

```
$mime = Input::file('photo')->getMimeType();
```

<a name="solicitar-informações"></a>

## Solicitar Informações

A classe `Request` oferece muitos métodos para examinar a solicitação HTTP da sua aplicação e estende a classe `Symfony\Component\HttpFoundation\Request`. Aqui estão alguns dos destaques.

**Recuperando o URI da solicitação**

```
$uri = Request::path();
```

**Determinando se o caminho do pedido corresponde a um padrão**

```
if (Request::is('admin/*'))
{
	//
}
```

**Obtenha o URL do pedido**

```
$url = Request::url();
```

**Recuperar um segmento de URI de solicitação**

```
$segment = Request::segment(1);
```

**Recuperando um cabeçalho de solicitação**

```
$value = Request::header('Content-Type');
```

**Recuperando Valores de $\_SERVER**

```
$value = Request::server('PATH_INFO');
```

**Determinando se a solicitação está em HTTPS**

```
if (Request::secure())
{
	//
}
```

**Determine se o pedido está usando AJAX**

```
if (Request::ajax())
{
	//
}
```

**Detectar qualquer tipo de solicitação JSON**

```
if (Request::ajax() or Request::isJson() or Request::wantsJson())
{
	//
}
```
