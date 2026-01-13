# Instalação

- [Instale o Composer](#install-composer)
- [Instalar Laravel](#install-laravel)
- [Requisitos do servidor](#requisitos-do-servidor)
- [Configuração](#configuração)
- [URLs bonitas](#urls-bonitas)

<a name="instalar-composer"></a>

## Instale o Composer

O Laravel utiliza o [Composer](http://getcomposer.org) para gerenciar suas dependências. Primeiro, baixe uma cópia do `composer.phar`. Uma vez que você tenha o arquivo PHAR, você pode mantê-lo no diretório do seu projeto local ou movê-lo para `usr/local/bin` para usá-lo globalmente no seu sistema. No Windows, você pode usar o [instalador do Composer](https://getcomposer.org/Composer-Setup.exe) para Windows.

<a name="instalar-laravel"></a>

## Instale o Laravel

### Via Composer Create-Project

Você pode instalar o Laravel executando o comando `create-project` do Composer em seu terminal:

```
composer create-project laravel/laravel --prefer-dist
```

### Via Download

Depois que o Composer for instalado, baixe a [versão mais recente](https://github.com/laravel/laravel/archive/master.zip) do framework Laravel e extraia seu conteúdo para um diretório no seu servidor. Em seguida, na raiz da sua aplicação Laravel, execute o comando `php composer.phar install` (ou `composer install`) para instalar todas as dependências do framework. Esse processo requer que o Git esteja instalado no servidor para que a instalação seja concluída com sucesso.

Se você quiser atualizar o framework Laravel, pode emitir o comando `php composer.phar update`.

<a name="requisitos-do-servidor"></a>

## Requisitos do servidor

O framework Laravel tem alguns requisitos de sistema:

- PHP >= 5.3.7
- Extensão MCrypt para PHP

A partir do PHP 5.5, algumas distribuições do sistema operacional podem exigir que você instale manualmente a extensão PHP JSON. Ao usar o Ubuntu, isso pode ser feito via `apt-get install php5-json`.

<a name="configuração"></a>

## Configuração

O Laravel não precisa de quase nenhuma configuração para começar. Você pode começar a desenvolver imediatamente! No entanto, você pode querer revisar o arquivo `app/config/app.php` e sua documentação. Ele contém várias opções, como `timezone` e `locale`, que você pode querer alterar de acordo com sua aplicação.

<a name="permissões"></a>

### Permissões

O Laravel exige um conjunto de permissões configuradas: pastas dentro do app/storage precisam ter acesso de escrita pelo servidor web.

<a name="caminhos"></a>

### Caminhos

Vários dos caminhos dos diretórios do framework são configuráveis. Para alterar a localização desses diretórios, confira o arquivo `bootstrap/paths.php`.

<a name="urls-bonitinha"></a>

## URLs bonitas

O framework vem com um arquivo `public/.htaccess` que é usado para permitir URLs sem `index.php`. Se você estiver usando o Apache para servir sua aplicação Laravel, certifique-se de habilitar o módulo `mod_rewrite`.

Se o arquivo `.htaccess` que vem com o Laravel não funcionar com sua instalação do Apache, tente este:

```
Options +FollowSymLinks
RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```
