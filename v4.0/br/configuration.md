# Configuração

- [Introdução](#introdução)
- [Configuração de ambiente](#configuracao-de-ambiente)
- Modo de manutenção [Modo de manutenção](#modo-de-manutenção)

<a name="introdução"></a>

## Introdução

Todos os arquivos de configuração do framework Laravel estão armazenados no diretório `app/config`. Cada opção em cada arquivo está documentada, então sinta-se à vontade para examinar os arquivos e se familiarizar com as opções disponíveis para você.

Às vezes, você pode precisar acessar os valores de configuração em tempo de execução. Você pode fazer isso usando a classe `Config`:

**Acesse um valor de configuração**

```
Config::get('app.timezone');
```

Você também pode especificar um valor padrão para retornar se a opção de configuração não existir:

```
$timezone = Config::get('app.timezone', 'UTC');
```

Observe que a sintaxe de estilo "ponto" pode ser usada para acessar valores nos vários arquivos. Você também pode definir valores de configuração no tempo de execução:

**Definindo um valor de configuração**

```
Config::set('database.default', 'sqlite');
```

Os valores de configuração definidos em tempo de execução são definidos apenas para o pedido atual e não serão transferidos para pedidos subsequentes.

<a name="configuração-ambiental"></a>

## Configuração de ambiente

Muitas vezes, é útil ter diferentes valores de configuração com base no ambiente em que a aplicação está sendo executada. Por exemplo, você pode querer usar um driver de cache diferente na máquina de desenvolvimento local do que no servidor de produção. É fácil realizar isso usando a configuração baseada no ambiente.

Basta criar uma pasta dentro do diretório `config` que corresponda ao nome do seu ambiente, como `local`. Em seguida, crie os arquivos de configuração que deseja sobrescrever e especifique as opções para esse ambiente. Por exemplo, para sobrescrever o driver de cache para o ambiente local, você criaria um arquivo `cache.php` em `app/config/local` com o seguinte conteúdo:

```
<?php

return array(

	'driver' => 'file',

);
```

> **Observação:**
>
>  Não use "testing" como nome de ambiente. Este é reservado para testes unitários.

Observe que você não precisa especificar *todas* as opções que estão no arquivo de configuração básica, mas apenas as opções que deseja substituir. Os arquivos de configuração do ambiente "cascatarão" sobre os arquivos básicos.

Em seguida, precisamos instruir o framework sobre como determinar em qual ambiente ele está rodando. O ambiente padrão é sempre `production`. No entanto, você pode configurar outros ambientes no arquivo `bootstrap/start.php` na raiz da sua instalação. Neste arquivo, você encontrará uma chamada `$app->detectEnvironment`. O array passado para este método é usado para determinar o ambiente atual. Você pode adicionar outros ambientes e nomes de máquinas ao array conforme necessário.

```
<?php

$env = $app->detectEnvironment(array(

    'local' => array('your-machine-name'),

));
```

Neste exemplo, 'local' é o nome do ambiente e 'seu-nome-da-máquina' é o nome do host do seu servidor. No Linux e no Mac, você pode determinar o nome do host usando o comando de terminal `hostname`.

Você também pode passar um `Closure` para o método `detectEnvironment`, permitindo que você implemente sua própria detecção de ambiente:

```
$env = $app->detectEnvironment(function()
{
	return $_SERVER['MY_LARAVEL_ENV'];
});
```

Você pode acessar o ambiente de aplicação atual através do método `environment`:

**Acesse o ambiente de aplicação atual**

```
$environment = App::environment();
```

<a name="modo-de-manutenção"></a>

## Modo de manutenção

Quando sua aplicação estiver no modo de manutenção, uma visualização personalizada será exibida para todas as rotas que chegam à sua aplicação. Isso facilita a "desativação" da sua aplicação enquanto ela está sendo atualizada. Uma chamada ao método `App::down` já está presente em seu arquivo `app/start/global.php`. A resposta desse método será enviada aos usuários quando sua aplicação estiver no modo de manutenção.

Para habilitar o modo de manutenção, execute o comando `down` do Artisan:

```
php artisan down
```

Para desativar o modo de manutenção, use o comando `up`:

```
php artisan up
```

Para exibir uma visualização personalizada quando sua aplicação estiver no modo de manutenção, você pode adicionar algo como o seguinte ao arquivo `app/start/global.php` da sua aplicação:

```
App::down(function()
{
	return Response::view('maintenance', array(), 503);
});
```
