# Segurança

- [Configuração](#configuração)
- [Armazenamento de Senhas](#armazenamento-de-senhas)
- [Autenticação de usuários](#autenticação-de-usuários)
- [Fazer login manualmente dos usuários](#manualmente)
- [Proteger rotas](#proteger-rotas)
- [Autenticação básica HTTP](#http-basic-authentication)
- [Lembretes e redefinição de senha](#lembretes-e-redefinicao-de-senha)
- [Criptografia](#criptografia)

<a name="configuração"></a>

## Configuração

O Laravel visa tornar a implementação de autenticação muito simples. Na verdade, quase tudo é configurado para você, pronto para uso. O arquivo de configuração de autenticação está localizado em `app/config/auth.php`, que contém várias opções bem documentadas para ajustar o comportamento das facilidades de autenticação.

Por padrão, o Laravel inclui um modelo `User` no diretório `app/models` que pode ser usado com o driver de autenticação Eloquent padrão. Lembre-se de incluir, ao criar o esquema desse modelo, que o campo de senha tenha um comprimento mínimo de 60 caracteres.

Se o seu aplicativo não estiver usando Eloquent, você pode usar o driver de autenticação `database`, que utiliza o construtor de consultas do Laravel.

<a name="storing-passwords"></a>

## Armazenamento de senhas

A classe `Hash` do Laravel oferece hashing seguro com Bcrypt:

**Hashando uma senha usando Bcrypt**

```
$password = Hash::make('secret');
```

**Verificação de uma senha contra um hash**

```
if (Hash::check('secret', $hashedPassword))
{
	// The passwords match...
}
```

**Verificando se uma senha precisa ser rehashada**

```
if (Hash::needsRehash($hashed))
{
	$hashed = Hash::make('secret');
}
```

<a name="authenticando-usuarios"></a>

## Autenticação de Usuários

Para fazer o login de um usuário em sua aplicação, você pode usar o método `Auth::attempt`.

```
if (Auth::attempt(array('email' => $email, 'password' => $password)))
{
	return Redirect::intended('dashboard');
}
```

Tenha em mente que `email` não é uma opção obrigatória, ela é apenas usada como exemplo. Você deve usar qualquer nome de coluna que corresponda a um "username" no seu banco de dados. A função `Redirect::intended` redirecionará o usuário para o URL que ele estava tentando acessar antes de ser capturado pelo filtro de autenticação. Um URI de fallback pode ser fornecido a este método caso o destino pretendido não esteja disponível.

Quando o método `attempt` for chamado, o evento `auth.attempt` [eventos](/docs/events) será disparado. Se a tentativa de autenticação for bem-sucedida e o usuário estiver logado, o evento `auth.login` também será disparado.

Para determinar se o usuário já está logado na sua aplicação, você pode usar o método `check`:

**Determinando se um usuário está autenticado**

```
if (Auth::check())
{
	// The user is logged in...
}
```

Se você deseja fornecer a funcionalidade "lembrar-me" em sua aplicação, você pode passar `true` como o segundo argumento para o método `attempt`, o que manterá o usuário autenticado indefinidamente (ou até que ele faça logout manualmente):

**Autenticação de um Usuário e "Lembrança" deles**

```
if (Auth::attempt(array('email' => $email, 'password' => $password), true))
{
	// The user is being remembered...
}
```

**Nota:** Se o método `attempt` retornar `true`, o usuário é considerado autenticado na aplicação.

Você também pode adicionar condições adicionais à consulta de autenticação:

**Autenticação de um Usuário com Condições**

```
if (Auth::attempt(array('email' => $email, 'password' => $password, 'active' => 1)))
{
    // The user is active, not suspended, and exists.
}
```

Depois que um usuário for autenticado, você poderá acessar o modelo/registro do usuário:

**Acessando o Usuário Ativado**

```
$email = Auth::user()->email;
```

Para simplesmente fazer o login de um usuário na aplicação por meio de seu ID, use o método `loginUsingId`:

```
Auth::loginUsingId(1);
```

O método `validate` permite que você valide as credenciais de um usuário sem realmente logá-lo na aplicação:

**Validação de Credenciais de Usuário sem Login**

```
if (Auth::validate($credentials))
{
	//
}
```

Você também pode usar o método `once` para fazer o login de um usuário na aplicação para uma única solicitação. Nenhuma sessão ou cookie será utilizado.

**Fazer login de um usuário para uma única solicitação**

```
if (Auth::once($credentials))
{
	//
}
```

**Sair um usuário do aplicativo**

```
Auth::logout();
```

<a name="manualmente"></a>

## Acessar manualmente os usuários

Se você precisar registrar uma instância de usuário existente em sua aplicação, você pode simplesmente chamar o método `login` com a instância:

```
$user = User::find(1);

Auth::login($user);
```

Isso é equivalente a fazer login de um usuário por meio de credenciais usando o método `attempt`.

<a name="protegendo-as-vias">protegendo-as-vias</a>

## Proteger rotas

Os filtros de rota podem ser usados para permitir que apenas usuários autenticados acessem uma rota específica. O Laravel fornece o filtro `auth` por padrão, e ele está definido em `app/filters.php`.

**Protegendo uma rota**

```
Route::get('profile', array('before' => 'auth', function()
{
	// Only authenticated users may enter...
}));
```

### Proteção contra CSRF

O Laravel oferece um método fácil de proteger sua aplicação contra falsificações de solicitações entre sites.

**Inserindo o Token CSRF na Formulário**

```
<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">
```

**Valide o Token CSRF enviado**

```
Route::post('register', array('before' => 'csrf', function()
{
    return 'You gave a valid CSRF token!';
}));
```

<a name="http-basic-authentication"></a>

## Autenticação básica HTTP

A Autenticação Básica HTTP oferece uma maneira rápida de autenticar os usuários do seu aplicativo sem precisar configurar uma página de "login" dedicada. Para começar, adicione o filtro `auth.basic` à sua rota:

**Protegendo uma rota com HTTP Basic**

```
Route::get('profile', array('before' => 'auth.basic', function()
{
	// Only authenticated users may enter...
}));
```

Por padrão, o filtro `basic` usará a coluna `email` no registro do usuário durante a autenticação. Se você deseja usar outra coluna, pode passar o nome da coluna como o primeiro parâmetro para o método `basic`:

```
return Auth::basic('username');
```

Você também pode usar a Autenticação Básica HTTP sem definir um cookie de identificador de usuário na sessão, o que é particularmente útil para a autenticação de API. Para fazer isso, defina um filtro que retorne o método `onceBasic`:

**Configurando um Filtro HTTP Basic sem Estado**

```
Route::filter('basic.once', function()
{
	return Auth::onceBasic();
});
```

Se você estiver usando PHP FastCGI, a autenticação básica HTTP não funcionará corretamente por padrão. As seguintes linhas devem ser adicionadas ao seu arquivo `.htaccess`:

```
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

<a name="lembranças-de-senha-e-redefinir">Lembranças de senha e redefinir</a>

## Lembretes e redefinição de senhas

### Enviar lembretes de senha

A maioria das aplicações web oferece uma maneira para os usuários redefinir suas senhas esquecidas. Em vez de forçá-lo a reimplementar isso em cada aplicativo, o Laravel fornece métodos convenientes para enviar lembretes de senha e realizar redefinições de senha. Para começar, verifique se seu modelo `User` implementa o contrato `Illuminate\Auth\Reminders\RemindableInterface`. Claro, o modelo `User` incluído no framework já implementa essa interface.

**Implementando o RemindableInterface**

```
class User extends Eloquent implements RemindableInterface {

	public function getReminderEmail()
	{
		return $this->email;
	}

}
```

Em seguida, uma tabela deve ser criada para armazenar os tokens de redefinição de senha. Para gerar uma migração para essa tabela, execute simplesmente o comando `auth:reminders` do Artisan:

**Gerando a Migração da Tabela de Lembranças**

```
php artisan auth:reminders

php artisan migrate
```

Para enviar um lembrete de senha, podemos usar o método `Password::remind`:

**Enviar um lembrete de senha**

```
Route::post('password/remind', function()
{
	$credentials = array('email' => Input::get('email'));

	return Password::remind($credentials);
});
```

Observe que os argumentos passados para o método `remind` são semelhantes ao método `Auth::attempt`. Esse método recuperará o `User` e enviará um link de redefinição de senha por e-mail. A visualização do e-mail receberá uma variável `token`, que pode ser usada para construir o link para o formulário de redefinição de senha. O objeto `user` também será passado para a visualização.

> **Observação:**
>
>  Você pode especificar qual visualização será usada como mensagem de e-mail alterando a opção de configuração 
>
> `auth.reminder.email`
>
> . Claro, uma visualização padrão é fornecida por padrão.

Você pode modificar a instância da mensagem que é enviada ao usuário passando uma Closure como o segundo argumento para o método `remind`:

```
return Password::remind($credentials, function($message, $user)
{
	$message->subject('Your Password Reminder');
});
```

Você também pode ter notado que estamos retornando os resultados do método `remind` diretamente de uma rota. Por padrão, o método `remind` retornará um `Redirect` para o URI atual. Se ocorrer um erro ao tentar redefinir a senha, uma variável `error` será exibida na sessão, bem como uma `reason`, que pode ser usada para extrair uma linha de idioma do arquivo de idioma `reminders`. Se o redefinimento da senha foi bem-sucedido, uma variável `success` será exibida na sessão. Portanto, a visualização do formulário de redefinição de senha pode parecer algo como isso:

```
@if (Session::has('error'))
	{{ trans(Session::get('reason')) }}
@elseif (Session::has('success'))
	An e-mail with the password reset has been sent.
@endif

<input type="text" name="email">
<input type="submit" value="Send Reminder">
```

### Redefinir senhas

Depois que um usuário clicar no link de redefinição do e-mail de lembretes, ele deve ser direcionado para um formulário que inclui um campo `token` oculto, além de campos `senha` e `confirmação_senha`. Abaixo está um exemplo de rota para o formulário de redefinição de senha:

```
Route::get('password/reset/{token}', function($token)
{
	return View::make('auth.reset')->with('token', $token);
});
```

E, um formulário de redefinição de senha pode parecer assim:

```
@if (Session::has('error'))
	{{ trans(Session::get('reason')) }}
@endif

<input type="hidden" name="token" value="{{ $token }}">
<input type="text" name="email">
<input type="password" name="password">
<input type="password" name="password_confirmation">
```

Novamente, observe que estamos usando a `Session` para exibir quaisquer erros que possam ser detectados pelo framework durante o redefinição de senhas. Em seguida, podemos definir uma rota `POST` para lidar com o redefinição:

```
Route::post('password/reset/{token}', function()
{
	$credentials = array(
	    'email' => Input::get('email'),
	    'password' => Input::get('password'),
	    'password_confirmation' => Input::get('password_confirmation')
	);

	return Password::reset($credentials, function($user, $password)
	{
		$user->password = Hash::make($password);

		$user->save();

		return Redirect::to('home');
	});
});
```

Se a redefinição da senha for bem-sucedida, a instância `User` e a senha serão passadas para a sua Closure, permitindo que você realize a operação de salvar. Em seguida, você pode retornar uma `Redirect` ou qualquer outro tipo de resposta da Closure, que será retornada pelo método `reset`. Observe que o método `reset` verifica automaticamente a existência de um `token` válido no pedido, credenciais válidas e senhas correspondentes.

Por padrão, os tokens de redefinição de senha expiram após uma hora. Você pode alterar isso através da opção `reminder.expire` do seu arquivo `app/config/auth.php`.

Além disso, de forma semelhante ao método `remind`, se ocorrer um erro durante o redefinimento da senha, o método `reset` retornará um `Redirect` para o URI atual com um `error` e `reason`.

<a name="criptografia"></a>

## Criptografia

O Laravel oferece recursos para criptografia AES-256 forte através da extensão PHP mcrypt:

**Cifrar um valor**

```
$encrypted = Crypt::encrypt('secret');
```

> **Observação:**
>
>  Certifique-se de definir uma string aleatória de 32 caracteres na opção 
>
> `key`
>
>  do arquivo 
>
> `app/config/app.php`
>
> . Caso contrário, os valores criptografados não serão seguros.

**Descifrando um Valor**

```
$decrypted = Crypt::decrypt($encryptedValue);
```

Você também pode definir o cifrador e o modo usados pelo encriptador:

**Definindo o Cifrado e o Modo**

```
Crypt::setMode('ctr');

Crypt::setCipher($cipher);
```
