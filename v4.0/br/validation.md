# Validação

- [Uso básico](#uso-básico)
- [Trabalhando com Mensagens de Erro](#trabalhando-com-mensagens-de-erro)
- [Mensagens de erro e visualizações](#mensagens-de-erro-e-visualizações)
- [Regras de validação disponíveis](#regras-de-validação-disponíveis)
- [Adicionando regras condicionalmente](#adicionando-regras-condicionalmente)
- [Mensagens de erro personalizadas](#mensagens-de-erro-personalizadas)
- [Regras de validação personalizadas](#custom-validation-rules)

<a name="uso-básico"></a>

## Uso básico

O Laravel vem com uma funcionalidade simples e conveniente para validar dados e recuperar mensagens de erro de validação via a classe `Validation`.

**Exemplo de validação básica**

```
$validator = Validator::make(
	array('name' => 'Dayle'),
	array('name' => 'required|min:5')
);
```

O primeiro argumento passado para o método `make` é os dados sob validação. O segundo argumento são as regras de validação que devem ser aplicadas aos dados.

Várias regras podem ser delimitadas usando um caractere "barra invertida" ou como elementos separados de um array.

**Usando Arrays para Especificar Regras**

```
$validator = Validator::make(
	array('name' => 'Dayle'),
	array('name' => array('required', 'min:5'))
);
```

**Validando vários campos**

```
$validator = Validator::make(
    array(
        'name' => 'Dayle',
        'password' => 'lamepassword',
        'email' => 'email@example.com'
    ),
    array(
        'name' => 'required',
        'password' => 'required|min:8',
        'email' => 'required|email|unique:users'
    )
);
```

Uma vez que uma instância do `Validator` tenha sido criada, o método `fails` (ou `passes`) pode ser usado para realizar a validação.

```
if ($validator->fails())
{
	// The given data did not pass validation
}
```

Se a validação falhou, você pode recuperar as mensagens de erro do validador.

```
$messages = $validator->messages();
```

Você também pode acessar uma matriz de regras de validação falhas, sem mensagens. Para fazer isso, use o método `failed`:

```
$failed = $validator->failed();
```

**Validando arquivos**

A classe `Validator` fornece várias regras para validar arquivos, como `size`, `mimes` e outras. Ao validar arquivos, você pode simplesmente passá-los para o validador junto com seus outros dados.

<a name="trabalhando-com-mensagens-de-erro"></a>

## Trabalhando com Mensagens de Erro

Após chamar o método `messages` em uma instância de `Validator`, você receberá uma instância de `MessageBag`, que possui uma variedade de métodos convenientes para trabalhar com mensagens de erro.

**Recuperando a Primeira Mensagem de Erro para um Campo**

```
echo $messages->first('email');
```

**Recuperando todos os mensagens de erro para um campo**

```
foreach ($messages->get('email') as $message)
{
	//
}
```

**Recuperando todos os mensagens de erro para todos os campos**

```
foreach ($messages->all() as $message)
{
	//
}
```

**Determinando se existem mensagens para um campo**

```
if ($messages->has('email'))
{
	//
}
```

**Recuperando uma mensagem de erro com um formato**

```
echo $messages->first('email', '<p>:message</p>');
```

> **Nota:**
>
>  Por padrão, as mensagens são formatadas usando a sintaxe compatível com o Bootstrap.

**Recuperando todos os Mensagens de Erro com um Formato**

```
foreach ($messages->all('<li>:message</li>') as $message)
{
	//
}
```

<a name="mensagens-de-erro-e-visões"></a>

## Mensagens de erro e visualizações

Depois de realizar a validação, você precisará de uma maneira fácil de obter as mensagens de erro de volta para suas visualizações. Isso é convenientemente gerenciado pelo Laravel. Considere as seguintes rotas como exemplo:

```
Route::get('register', function()
{
	return View::make('user.register');
});

Route::post('register', function()
{
	$rules = array(...);

	$validator = Validator::make(Input::all(), $rules);

	if ($validator->fails())
	{
		return Redirect::to('register')->withErrors($validator);
	}
});
```

Observe que, quando a validação falha, passamos a instância do `Validator` para o redirecionamento usando o método `withErrors`. Esse método exibe as mensagens de erro na sessão, para que estejam disponíveis na próxima solicitação.

No entanto, observe que não precisamos vincular explicitamente as mensagens de erro à vista em nossa rota GET. Isso ocorre porque o Laravel sempre verificará os erros nos dados da sessão e os vinculará automaticamente à vista se estiverem disponíveis. **Portanto, é importante notar que uma variável `$errors` estará sempre disponível em todas as suas vistas, em cada solicitação**, permitindo que você assuma convenientemente que a variável `$errors` está sempre definida e pode ser usada com segurança. A variável `$errors` será uma instância de `MessageBag`.

Então, após a redirecionamento, você pode utilizar a variável `$errors` vinculada automaticamente em sua visualização:

```
<?php echo $errors->first('email'); ?>
```

<a name="regras-de-validação-disponíveis"></a>

## Regras de validação disponíveis

Abaixo está uma lista de todas as regras de validação disponíveis e suas funções:

- [Aceito](#regra-aceito)
- [URL Ativo](#regra-ativo-url)
- [Após (Data)](#rule-after)
- [Alpha](#rule-alpha)
- [Alpha Dash](#rule-alpha-dash)
- [Alfanumérico](#regra-alfanum)
- [Antes de (Data)](#rule-before)
- [Entre](#rule-between)
- [Confirmado](#regra-confirmada)
- [Data](#rule-date)
- [Formato de data](#regra-formato-data)
- [Diferente](#regra-diferente)
- [E-Mail](#regra-e-mail)
- [Existe (Banco de dados)](#rule-exists)
- \[Imagem (Arquivo)] (#rule-image)
- [Em](#regras-de-inclusão)
- [Inteiro](#regra-inteiro)
- [Endereço IP](#regra-ip)
- [Max](#regra-max)
- [Tipos de MIME](#regra-mime)
- [Min](#regra-min)
- [Não em](#regra-não-em)
- [Numérico](#regra-numérica)
- [Expressão Regular](#regra-regex)
- [Obrigatório](#regra-obrigatório)
- [Obrigatório se](#rule-required-if)
- [Obrigatório com](#rule-obrigatorio-com)
- [Obrigatório sem](#regra-obrigatório-sem)
- [Mesmo](#regra-mesma)
- [Tamanho](#tamanho-da-regra)
- \[Único (Banco de dados)] (#rule-unique)
- [URL](#rule-url)

<a name="regra-aceita"></a>

#### aceito

O campo em validação deve ser *sim*, *ativado* ou *1*. Isso é útil para validar a aceitação dos Termos de Serviço.

<a name="rule-active-url"></a>

#### url\_ativa

O campo em validação deve ser uma URL válida de acordo com a função `checkdnsrr` do PHP.

<a name="rule-after"></a>

#### após:*data*

O campo em validação deve ser um valor após uma data específica. As datas serão passadas para a função `strtotime` do PHP.

<a name="regra-alfa"></a>

#### alfa

O campo em validação deve ser composto apenas por caracteres alfabéticos.

<a name="rule-alpha-dash"></a>

#### alpha\_dash

O campo em validação pode conter caracteres alfanuméricos, bem como hífens e sublinhados.

<a name="rule-alpha-num"></a>

#### alpha\_num

O campo em validação deve conter apenas caracteres alfanuméricos.

<a name="rule-before"></a>

#### antes de:*data*

O campo em validação deve ser um valor anterior à data fornecida. As datas serão passadas para a função `strtotime` do PHP.

<a name="rule-between"></a>

#### entre:*min*,*max*

O campo em validação deve ter um tamanho entre o *mínimo* e o *máximo* fornecidos. Strings, números e arquivos são avaliados da mesma forma que a regra `size`.

<a name="regra-confirmada"></a>

#### confirmado

O campo em validação deve ter um campo correspondente de `foo_confirmation`. Por exemplo, se o campo em validação for `password`, um campo correspondente `password_confirmation` deve estar presente na entrada.

<a name="rule-date"></a>

#### data

O campo em validação deve ser uma data válida de acordo com a função PHP `strtotime`.

<a name="rule-date-format"></a>

#### data\_format:*format*

O campo em validação deve corresponder ao *formato* definido de acordo com a função PHP `date_parse_from_format`.

<a name="regra-diferente"></a>

#### diferente:*campo*

O campo fornecido deve ser diferente do campo em validação.

<a name="rule-email"></a>

#### e-mail

O campo em validação deve ser formatado como um endereço de e-mail.

<a name="rule-exists"></a>

#### existe:*tabela*,*coluna*

O campo em validação deve existir em uma tabela de banco de dados específica.

**Uso básico da regra Exists**

```
'state' => 'exists:states'
```

**Especificando um Nome de Coluna Personalizado**

```
'state' => 'exists:states,abbreviation'
```

Você também pode especificar mais condições que serão adicionadas como cláusulas "onde" à consulta:

```
'email' => 'exists:staff,email,account_id,1'
```

<a name="rule-image"></a>

#### imagem

O arquivo em validação deve ser uma imagem (jpeg, png, bmp ou gif)

<a name="rule-in"></a>

#### em:*foo*,*bar*,...

O campo em validação deve estar incluído na lista de valores fornecida.

<a name="regra-inteira"></a>

#### inteiro

O campo em validação deve ter um valor inteiro.

<a name="rule-ip"></a>

#### ip

O campo em validação deve ser formatado como um endereço IP.

<a name="rule-max"></a>

#### max:*valor*

O campo em validação deve ser menor que um valor máximo. Strings, números e arquivos são avaliados da mesma forma que a regra `size`.

<a name="rule-mimes"></a>

#### mimes:*foo*,*bar*,...

O arquivo em validação deve ter um tipo MIME correspondente a uma das extensões listadas.

**Uso básico da regra MIME**

```
'photo' => 'mimes:jpeg,bmp,png'
```

<a name="rule-min"></a>

#### min:*valor*

O campo em validação deve ter um valor mínimo. Strings, números e arquivos são avaliados da mesma forma que a regra `size`.

<a name="rule-not-in"></a>

#### não\_em:*foo*,*bar*,...

O campo em validação não deve estar incluído na lista de valores fornecida.

<a name="rule-numeric"></a>

#### numérico

O campo em validação deve ter um valor numérico.

<a name="rule-regex"></a>

#### regex:*padrão*

O campo em validação deve corresponder à expressão regular fornecida.

**Observação:** Ao usar o padrão `regex`, pode ser necessário especificar regras em um array em vez de usar delimitadores de pipe, especialmente se a expressão regular contiver um caractere de pipe.

<a name="rule-required"></a>

#### obrigatório

O campo em validação deve estar presente nos dados de entrada.

<a name="rule-required-if"></a>

#### requerido\_se:*campo*,*valor*

O campo em validação deve estar presente se o campo *campo* for igual a *valor*.

<a name="rule-required-with"></a>

#### requerido\_com:*foo*,*bar*,...

O campo em validação deve estar presente *apenas se* os outros campos especificados estiverem presentes.

<a name="rule-required-without"></a>

#### requerido\_sem:*foo*,*bar*,...

O campo em validação deve estar presente *apenas quando* os outros campos especificados não estiverem presentes.

<a name="regra-mesma"></a>

#### mesmo:*campo*

O campo fornecido deve corresponder ao campo em validação.

<a name="rule-size"></a>

#### tamanho:*valor*

O campo em validação deve ter um tamanho correspondente ao valor fornecido. Para dados de texto, *valor* corresponde ao número de caracteres. Para dados numéricos, *valor* corresponde a um valor inteiro fornecido. Para arquivos, *tamanho* corresponde ao tamanho do arquivo em kilobytes.

<a name="regra-única"></a>

#### único:*tabela*,*coluna*,*exceto*,*idColumn*

O campo em validação deve ser único em uma tabela de banco de dados específica. Se a opção `column` não for especificada, o nome do campo será usado.

**Uso básico da regra exclusiva**

```
'email' => 'unique:users'
```

**Especificando um Nome de Coluna Personalizado**

```
'email' => 'unique:users,email_address'
```

**Forçar uma regra única para ignorar uma ID específica**

```
'email' => 'unique:users,email_address,10'
```

**Adicionando cláusulas onde adicionais**

Você também pode especificar mais condições que serão adicionadas como cláusulas "onde" à consulta:

```
'email' => 'unique:users,email_address,NULL,id,account_id,1'
```

Na regra acima, apenas as linhas com um `account_id` de `1` seriam incluídas na verificação única.

<a name="rule-url"></a>

#### url

O campo em validação deve ser formatado como um URL.

<a name="adicionando-regras-condicionalmente"></a>

## Adicionar regras condicionalmente

Às vezes, você pode querer exigir um campo específico apenas se outro campo tiver um valor maior que 100. Ou você pode precisar que dois campos tenham um valor específico apenas quando outro campo estiver presente. Adicionar essas regras de validação não precisa ser complicado. Primeiro, crie uma instância de `Validator` com suas regras *estáticas* que nunca mudam:

```
$v = Validator::make($data, array(
	'email' => 'required|email',
	'games' => 'required|numeric',
));
```

Vamos supor que nossa aplicação web é para colecionadores de jogos. Se um colecionador de jogos se registrar em nossa aplicação e possuir mais de 100 jogos, queremos que ele explique por que possui tantos jogos. Por exemplo, talvez ele gere uma loja de revenda de jogos ou talvez ele simplesmente goste de colecionar. Para adicionar condicionalmente essa exigência, podemos usar o método `sometimes` na instância do `Validator`.

```
$v->sometimes('reason', 'required|max:500', function($input)
{
	return $input->games >= 100;
});
```

O primeiro argumento passado para o método `sometimes` é o nome do campo que estamos validando condicionalmente. O segundo argumento são as regras que queremos adicionar. Se o `Closure` passado como o terceiro argumento retornar `true`, as regras serão adicionadas. Este método facilita a construção de validações condicionais complexas. Você pode até adicionar validações condicionais para vários campos de uma só vez:

```
$v->sometimes(array('reason', 'cost'), 'required', function($input)
{
	return $input->games >= 100;
});
```

> **Observação:**
>
>  O parâmetro 
>
> `$input`
>
>  passado para sua 
>
> `Closure`
>
>  será uma instância do 
>
> `Illuminate\Support\Fluent`
>
>  e pode ser usado como um objeto para acessar seu input e arquivos.

<a name="mensagens-de-erro-personalizadas"></a>

## Mensagens de erro personalizadas

Se necessário, você pode usar mensagens de erro personalizadas para validação em vez das mensagens padrão. Existem várias maneiras de especificar mensagens personalizadas.

**Passando Mensagens Personalizadas para o Validador**

```
$messages = array(
	'required' => 'The :attribute field is required.',
);

$validator = Validator::make($input, $rules, $messages);
```

*Observação:* O marcador `:attribute` será substituído pelo nome real do campo durante a validação. Você também pode utilizar outros marcadores nas mensagens de validação.

**Outros suportes de validação**

```
$messages = array(
	'same'    => 'The :attribute and :other must match.',
	'size'    => 'The :attribute must be exactly :size.',
	'between' => 'The :attribute must be between :min - :max.',
	'in'      => 'The :attribute must be one of the following types: :values',
);
```

Às vezes, você pode querer especificar mensagens de erro personalizadas apenas para um campo específico:

**Especificando uma mensagem personalizada para um atributo específico**

```
$messages = array(
	'email.required' => 'We need to know your e-mail address!',
);
```

Em alguns casos, você pode querer especificar suas mensagens personalizadas em um arquivo de idioma em vez de passá-las diretamente ao `Validator`. Para fazer isso, adicione suas mensagens ao array `custom` no arquivo de idioma `app/lang/xx/validation.php`.

<a name="localização"></a>
**Especificando Mensagens Personalizadas em Arquivos de Idioma**

```
'custom' => array(
	'email' => array(
		'required' => 'We need to know your e-mail address!',
	),
),
```

<a name="custom-validation-rules"></a>

## Regras de validação personalizadas

O Laravel oferece uma variedade de regras de validação úteis; no entanto, você pode querer especificar algumas de suas próprias. Um método de registro de regras de validação personalizadas é usar o método `Validator::extend`:

**Registrando uma regra de validação personalizada**

```
Validator::extend('foo', function($attribute, $value, $parameters)
{
	return $value == 'foo';
});
```

O Closure do validador personalizado recebe três argumentos: o nome do `$attribute` que está sendo validado, o `$value` do atributo e um array de `$parameters` passados para a regra.

Você também pode passar uma classe e um método para o método `extend` em vez de uma Closure:

```
Validator::extend('foo', 'FooValidator@validate');
```

Observe que você também precisará definir uma mensagem de erro para suas regras personalizadas. Você pode fazer isso usando um array de mensagem personalizada inline ou adicionando uma entrada no arquivo de linguagem de validação.

Em vez de usar callbacks de Closure para estender o Validator, você também pode estender a própria classe Validator. Para fazer isso, escreva uma classe Validator que estenda `Illuminate\Validation\Validator`. Você pode adicionar métodos de validação à classe, prefixando-os com `validate`:

**Extensão da Classe Validator**

```
<?php

class CustomValidator extends Illuminate\Validation\Validator {

	public function validateFoo($attribute, $value, $parameters)
	{
		return $value == 'foo';
	}

}
```

Em seguida, você precisa registrar sua extensão de Validador personalizado:

**Registrar um resolutor de validadores personalizados**

```
Validator::resolver(function($translator, $data, $rules, $messages)
{
	return new CustomValidator($translator, $data, $rules, $messages);
});
```

Ao criar uma regra de validação personalizada, às vezes você pode precisar definir substituições personalizadas para mensagens de erro. Isso pode ser feito criando um Validador personalizado conforme descrito acima e adicionando uma função `replaceXXX` ao validador.

```
protected function replaceFoo($message, $attribute, $rule, $parameters)
{
	return str_replace(':foo', $parameters[0], $message);
}
```
