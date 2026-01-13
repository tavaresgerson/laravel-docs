# Desenvolvimento de artesãos

- [Introdução](#introdução)
- [Edifício A Comando](#edifcio-a-comando)
- [Registrar comandos](#registrar-comandos)
- [Chamando outras comandos](#chamando-outros-comandos)

<a name="introdução"></a>

## Introdução

Além dos comandos fornecidos com o Artisan, você também pode criar seus próprios comandos personalizados para trabalhar com sua aplicação. Você pode armazenar seus comandos personalizados no diretório `app/commands`; no entanto, você é livre para escolher seu próprio local de armazenamento, desde que seus comandos possam ser carregados automaticamente com base nas configurações do seu `composer.json`.

<a name="construindo-um-comando"> </a>

## Construindo um Comando

### Gerando a Classe

Para criar um novo comando, você pode usar o comando `command:make` do Artisan, que gerará um esboço do comando para ajudá-lo a começar:

**Crie uma nova classe de comando**

```
php artisan command:make FooCommand
```

Por padrão, os comandos gerados serão armazenados no diretório `app/commands`; no entanto, você pode especificar um caminho ou namespace personalizado:

```
php artisan command:make FooCommand --path=app/classes --namespace=Classes
```

### Escrever o Comando

Depois que o comando for gerado, você deve preencher as propriedades `name` e `description` da classe, que serão usadas ao exibir o comando na tela `list`.

O método `fire` será chamado quando seu comando for executado. Você pode colocar qualquer lógica de comando neste método.

### Argumentos e Opções

Os métodos `getArguments` e `getOptions` são onde você pode definir quaisquer argumentos ou opções que seu comando recebe. Ambos esses métodos retornam um array de comandos, que são descritos por uma lista de opções de array.

Ao definir `arguments`, os valores da definição de array representam o seguinte:

```
array($name, $mode, $description, $defaultValue)
```

O argumento `mode` pode ser qualquer um dos seguintes: `InputArgument::REQUIRED` ou `InputArgument::OPTIONAL`.

Ao definir `options`, os valores da definição do array representam o seguinte:

```
array($name, $shortcut, $mode, $description, $defaultValue)
```

Para opções, o argumento `mode` pode ser: `InputOption::VALUE_REQUIRED`, `InputOption::VALUE_OPTIONAL`, `InputOption::VALUE_IS_ARRAY`, `InputOption::VALUE_NONE`.

O modo `VALUE_IS_ARRAY` indica que o interruptor pode ser usado várias vezes ao chamar o comando:

```
php artisan foo --option=bar --option=baz
```

A opção `VALUE_NONE` indica que a opção é simplesmente usada como um "interruptor":

```
php artisan foo --option
```

### Recuperação de entrada

Enquanto seu comando estiver sendo executado, você obviamente precisará acessar os valores dos argumentos e opções aceitos por sua aplicação. Para fazer isso, você pode usar os métodos `argument` e `option`:

**Recuperando o Valor de um Argumento de Comando**

```
$value = $this->argument('name');
```

**Recuperando todos os argumentos**

```
$arguments = $this->argument();
```

**Recuperando o valor de uma opção de comando**

```
$value = $this->option('name');
```

**Recuperando todas as opções**

```
$options = $this->option();
```

### Saída de escrita

Para enviar a saída para o console, você pode usar os métodos `info`, `comment`, `question` e `error`. Cada um desses métodos usará as cores ANSI apropriadas para o seu propósito.

**Enviando informações para o console**

```
$this->info('Display this on the screen');
```

**Enviando uma mensagem de erro para o console**

```
$this->error('Something went wrong!');
```

### Fazendo perguntas

Você também pode usar os métodos `ask` e `confirm` para solicitar ao usuário uma entrada:

**Pedindo ao usuário para fornecer informações**

```
$name = $this->ask('What is your name?');
```

**Pedindo ao usuário para fornecer informações secretas**

```
$password = $this->secret('What is the password?');
```

**Pedindo confirmação ao usuário**

```
if ($this->confirm('Do you wish to continue? [yes|no]'))
{
	//
}
```

Você também pode especificar um valor padrão para o método `confirm`, que deve ser `true` ou `false`:

```
$this->confirm($question, true);
```

<a name="registrar-comandos"></a>

## Registrar comandos

Depois que seu comando estiver pronto, você precisa registrá-lo no Artisan para que ele esteja disponível para uso. Isso geralmente é feito no arquivo `app/start/artisan.php`. Dentro deste arquivo, você pode usar o método `Artisan::add` para registrar o comando:

**Registrar um comando de artesão**

```
Artisan::add(new CustomCommand);
```

Se o seu comando estiver registrado no container [IoC](/docs/ioc), você pode usar o método `Artisan::resolve` para torná-lo disponível para o Artisan:

**Registrando um comando que está no container IoC**

```
Artisan::resolve('binding.name');
```

<a name="chamando-outros-comandos"></a>

## Chamando outras comandos

Às vezes, você pode querer chamar outros comandos a partir do seu comando. Você pode fazer isso usando o método `call`:

**Chamando outro comando**

```
$this->call('command.name', array('argument' => 'foo', '--option' => 'bar'));
```
