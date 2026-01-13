# Desenvolvimento de pacotes

- [Introdução](#introdução)
- [Criando um pacote](#criando-um-pacote)
- [Estrutura do pacote](#estrutura-do-pacote)
- [Prestadores de Serviços](#prestadores-de-servicos)
- [Convenções de pacote](#convenções-de-pacote)
- [Fluxo de trabalho de desenvolvimento](#fluxo-de-trabalho-de-desenvolvimento)
- [Roteamento de pacotes](#roteamento-de-pacotes)
- [Configuração do pacote](#configuracao-do-pacote)
- [Migrações de pacote](#migrações-de-pacote)
- [Ativos do pacote](#a-propriedade-do-pacote)
- Pacotes de Publicação [Pacotes de Publicação](#pacotes-de-publicação)

<a name="introdução"></a>

## Introdução

Os pacotes são a principal maneira de adicionar funcionalidades ao Laravel. Os pacotes podem ser qualquer coisa, desde uma ótima maneira de trabalhar com datas, como o [Carbon](https://github.com/briannesbitt/Carbon), ou um framework completo de teste BDD, como o [Behat](https://github.com/Behat/Behat).

Claro, existem diferentes tipos de pacotes. Alguns pacotes são independentes, o que significa que funcionam com qualquer framework, não apenas o Laravel. Tanto o Carbon quanto o Behat são exemplos de pacotes independentes. Qualquer um desses pacotes pode ser usado com o Laravel simplesmente solicitando-os no seu arquivo `composer.json`.

Por outro lado, outros pacotes são especificamente destinados para uso com o Laravel. Em versões anteriores do Laravel, esses tipos de pacotes eram chamados de "bundles". Esses pacotes podem ter rotas, controladores, visualizações, configuração e migrações especificamente destinados a aprimorar uma aplicação Laravel. Como não é necessário um processo especial para desenvolver pacotes autônomos, este guia cobre principalmente o desenvolvimento dos que são específicos para o Laravel.

Todos os pacotes do Laravel são distribuídos através do [Packagist](http://packagist.org) e do [Composer](http://getcomposer.org), portanto, aprender sobre essas maravilhosas ferramentas de distribuição de pacotes PHP é essencial.

<a name="criando-um-pacote">Criando um pacote</a>

## Criando um pacote

A maneira mais fácil de criar um novo pacote para uso com o Laravel é o comando `workbench` do Artisan. Primeiro, você precisará definir algumas opções no arquivo `app/config/workbench.php`. Nesse arquivo, você encontrará as opções `name` e `email`. Esses valores serão usados para gerar um arquivo `composer.json` para seu novo pacote. Uma vez que você tenha fornecido esses valores, você está pronto para criar um pacote de workbench!

**Emissão do comando do Artisan Workbench**

```
php artisan workbench vendor/package --resources
```

O nome do fornecedor é uma maneira de distinguir seu pacote de outros pacotes com o mesmo nome de autores diferentes. Por exemplo, se eu (Taylor Otwell) criar um novo pacote chamado "Zapper", o nome do fornecedor poderia ser `Taylor`, enquanto o nome do pacote seria `Zapper`. Por padrão, o ambiente de trabalho criará pacotes independentes do framework; no entanto, o comando `resources` instrui o ambiente de trabalho a gerar o pacote com diretórios específicos do Laravel, como `migrations`, `views`, `config`, etc.

Depois que o comando `workbench` for executado, seu pacote estará disponível no diretório `workbench` da sua instalação do Laravel. Em seguida, você deve registrar o `ServiceProvider` que foi criado para seu pacote. Você pode registrar o provedor adicionando-o ao array `providers` no arquivo `app/config/app.php`. Isso instruirá o Laravel a carregar seu pacote quando a aplicação começar. Os provedores usam a convenção de nomenclatura `[Package]ServiceProvider`. Então, usando o exemplo acima, você adicionaria `Taylor\Zapper\ZapperServiceProvider` ao array `providers`.

Depois que o provedor tiver sido registrado, você estará pronto para começar a desenvolver seu pacote! No entanto, antes de mergulhar, você pode querer revisar as seções abaixo para se familiarizar com a estrutura do pacote e o fluxo de trabalho de desenvolvimento.

> **Nota:**
>
>  Se o seu provedor de serviços não for encontrado, execute o comando 
>
> `php artisan dump-autoload`
>
>  a partir do diretório raiz da sua aplicação.

<a name="package-structure"></a>

## Estrutura do pacote

Ao usar o comando `workbench`, seu pacote será configurado com convenções que permitem que o pacote se integre bem com outras partes do framework Laravel:

**Estrutura do Diretório do Pacote Básico**

```
/src
	/Vendor
		/Package
			PackageServiceProvider.php
	/config
	/lang
	/migrations
	/views
/tests
/public
```

Vamos explorar essa estrutura mais a fundo. O diretório `src/Vendor/Package` é o lar de todas as classes do seu pacote, incluindo o `ServiceProvider`. Os diretórios `config`, `lang`, `migrations` e `views`, como você pode imaginar, contêm os recursos correspondentes ao seu pacote. Os pacotes podem ter qualquer um desses recursos, assim como as aplicações "regulares".

<a name="prestadores-de-serviços"></a>

## Prestadores de Serviços

Os provedores de serviço são simplesmente classes de bootstrap para pacotes. Por padrão, eles contêm dois métodos: `boot` e `register`. Dentro desses métodos, você pode fazer o que quiser: incluir um arquivo de rotas, registrar vinculações no contêiner de IoC, se conectar a eventos ou qualquer outra coisa que você queira fazer.

O método `register` é chamado imediatamente quando o provedor de serviço é registrado, enquanto o comando `boot` é chamado apenas antes de um pedido ser roteado. Portanto, se as ações do seu provedor de serviço dependem de outro provedor de serviço já estar registrado, ou se você está sobrescrevendo serviços vinculados por outro provedor, você deve usar o método `boot`.

Ao criar um pacote usando o `workbench`, o comando `boot` já conterá uma ação:

```
$this->package('vendor/package');
```

Esse método permite que o Laravel saiba como carregar corretamente as visualizações, configurações e outros recursos da sua aplicação. Geralmente, não é necessário alterar essa linha de código, pois ela configurará o pacote usando as convenções do workbench.

Por padrão, após registrar um pacote, seus recursos estarão disponíveis usando a metade "package" do `vendor/package`. No entanto, você pode passar um segundo argumento para o método `package` para sobrescrever esse comportamento. Por exemplo:

```
// Passing custom namespace to package method
$this->package('vendor/package', 'custom-namespace');

// Package resources now accessed via custom-namespace
$view = View::make('custom-namespace::foo');
```

Não há uma localização "padrão" para as classes de provedores de serviços. Você pode colocá-las em qualquer lugar que desejar, talvez organizando-as em um namespace `Providers` dentro do seu diretório `app`. O arquivo pode ser colocado em qualquer lugar, desde que as facilidades de carregamento automático do Composer (<http://getcomposer.org/doc/01-basic-usage.md#autoloading>) saibam carregar a classe.

<a name="convenções-do-pacote"></a>

## Convenções de pacote

Ao utilizar recursos de um pacote, como itens de configuração ou visualizações, geralmente será usada a sintaxe de dois pontos e dois colchetes:

**Carregando uma visão de um pacote**

```
return View::make('package::view.name');
```

**Recuperando um Item de Configuração de Pacote**

```
return Config::get('package::group.option');
```

> **Observação:**
>
>  Se o seu pacote contém migrações, considere prefixar o nome da migração com o nome do seu pacote para evitar potenciais conflitos de nomes de classes com outros pacotes.

<a name="workflow-de-desenvolvimento"></a>

## Fluxo de trabalho de desenvolvimento

Ao desenvolver um pacote, é útil poder trabalhar dentro do contexto de uma aplicação, permitindo que você visualize e experimente facilmente seus modelos, etc. Então, para começar, instale uma cópia nova do framework Laravel e, em seguida, use o comando `workbench` para criar a estrutura do seu pacote.

Depois que o comando `workbench` criou seu pacote, você pode fazer `git init` a partir do diretório `workbench/[vendor]/[package]` e `git push` seu pacote diretamente do workbench! Isso permitirá que você desenvolva o pacote convenientemente em um contexto de aplicativo sem ficar preso em constantes comandos `composer update`.

Como seus pacotes estão no diretório `workbench`, você pode estar se perguntando como o Composer sabe carregar automaticamente os arquivos do seu pacote. Quando o diretório `workbench` existir, o Laravel irá digitalizar inteligentemente ele em busca de pacotes, carregando seus arquivos de autocarregamento do Composer quando a aplicação for iniciada!

Se você precisar regenerar os arquivos de autoload do seu pacote, você pode usar o comando `php artisan dump-autoload`. Esse comando regenerará os arquivos de autoload do seu projeto raiz, bem como de quaisquer bancadas que você tenha criado.

**Executando o comando de autocarga do Artisan**

```
php artisan dump-autoload
```

<a name="package-routing"></a>

## Roteamento de pacotes

Em versões anteriores do Laravel, uma cláusula `handles` era usada para especificar quais URIs um pacote poderia responder. No entanto, no Laravel 4, um pacote pode responder a qualquer URI. Para carregar um arquivo de rotas para seu pacote, basta incluí-lo dentro do método `boot` do provedor de serviços.

**Inclusão de um arquivo de rotas de um provedor de serviços**

```
public function boot()
{
	$this->package('vendor/package');

	include __DIR__.'/../../routes.php';
}
```

> **Observação:**
>
>  Se o seu pacote estiver usando controladores, você precisará garantir que eles estejam configurados corretamente na seção de autocarga do arquivo 
>
> `composer.json`
>
> .

<a name="configuração-do-pacote"></a>

## Configuração do pacote

Alguns pacotes podem exigir arquivos de configuração. Esses arquivos devem ser definidos da mesma maneira que os arquivos de configuração típicos de aplicativos. E, ao usar o método padrão `$this->package` para registrar recursos no seu provedor de serviços, eles podem ser acessados usando a sintaxe usual de "dois colchetes":

**Acesse os arquivos de configuração do pacote**

```
Config::get('package::file.option');
```

No entanto, se o seu pacote contiver um único arquivo de configuração, você pode simplesmente nomeá-lo `config.php`. Quando isso for feito, você poderá acessar as opções diretamente, sem especificar o nome do arquivo:

**Acesse a configuração do pacote de arquivo único**

```
Config::get('package::option');
```

Às vezes, você pode querer registrar recursos do pacote, como vistas, fora do método típico `$this->package`. Normalmente, isso só seria feito se os recursos não estivessem em um local convencional. Para registrar os recursos manualmente, você pode usar o método `addNamespace` das classes `View`, `Lang` e `Config`:

**Registrar um namespace de recurso manualmente**

```
View::addNamespace('package', __DIR__.'/path/to/views');
```

Depois que o namespace for registrado, você pode usar o nome do namespace e a sintaxe de "dois colchetes" para acessar os recursos:

```
return View::make('package::view.name');
```

A assinatura do método para `addNamespace` é idêntica nas classes `View`, `Lang` e `Config`.

### Arquivos de configuração em cascata

Quando outros desenvolvedores instalarem seu pacote, eles podem querer sobrescrever algumas das opções de configuração. No entanto, se eles alterarem os valores no código-fonte do seu pacote, esses valores serão sobrescritos na próxima vez que o Composer atualizar o pacote. Em vez disso, o comando `config:publish` do Artisan deve ser usado:

**Executando o comando de publicação Config**

```
php artisan config:publish vendor/package
```

Quando este comando for executado, os arquivos de configuração do seu aplicativo serão copiados para `app/config/packages/vendor/package`, onde podem ser modificados com segurança pelo desenvolvedor!

> **Nota:**
>
>  O desenvolvedor também pode criar arquivos de configuração específicos para o ambiente do seu pacote, colocando-os em 
>
> `app/config/packages/vendor/package/environment`
>
> .

<a name="package-migrations"></a>

## Migrações de pacotes

Você pode criar e executar migrações para qualquer um dos seus pacotes com facilidade. Para criar uma migração para um pacote no workbench, use a opção `--bench`:

**Criando migrações para pacotes do Workbench**

```
php artisan migrate:make create_users_table --bench="vendor/package"
```

**Execução de migrações para pacotes do Workbench**

```
php artisan migrate --bench="vendor/package"
```

Para executar migrações para um pacote concluído que foi instalado via Composer no diretório `vendor`, você pode usar a diretiva `--package`:

**Executando migrações para um pacote instalado**

```
php artisan migrate --package="vendor/package"
```

<a name="package-assets"></a>

## Ativos do pacote

Alguns pacotes podem ter ativos como JavaScript, CSS e imagens. No entanto, não podemos vincular ativos nos diretórios `vendor` ou `workbench`, então precisamos de uma maneira de mover esses ativos para o diretório `public` da nossa aplicação. O comando `asset:publish` cuidará disso por você:

**Movendo ativos de pacote para o público**

```
php artisan asset:publish

php artisan asset:publish vendor/package
```

Se o pacote ainda estiver no `workbench`, use a diretiva `--bench`:

```
php artisan asset:publish --bench="vendor/package"
```

Esse comando moverá os ativos para o diretório `public/packages` de acordo com o fornecedor e o nome do pacote. Portanto, um pacote chamado `userscape/kudos` terá seus ativos movidos para `public/packages/userscape/kudos`. O uso dessa convenção de publicação de ativos permite que você codifique os caminhos dos ativos de forma segura nas visualizações do seu pacote.

<a name="publicacao-pacotes"></a>

## Pacotes de Publicação

Quando seu pacote estiver pronto para ser publicado, você deve enviá-lo para o repositório do [Packagist](http://packagist.org). Se o pacote for específico para o Laravel, considere adicionar uma tag `laravel` ao arquivo `composer.json` do seu pacote.

Além disso, é cortês e útil marcar suas versões para que os desenvolvedores possam confiar em versões estáveis ao solicitar seu pacote em seus arquivos `composer.json`. Se uma versão estável ainda não estiver pronta, considere usar a diretiva `branch-alias` do Composer.

Depois que o pacote for publicado, sinta-se à vontade para continuar desenvolvendo-o dentro do contexto da aplicação criado pelo `workbench`. Esta é uma ótima maneira de continuar desenvolvendo o pacote de forma conveniente mesmo depois de ele ter sido publicado.

Algumas organizações optam por hospedar seu próprio repositório privado de pacotes para seus próprios desenvolvedores. Se você estiver interessado em fazer isso, revise a documentação do projeto [Satis](http://github.com/composer/satis) fornecida pela equipe do Composer.
