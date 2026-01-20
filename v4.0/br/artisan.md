# Artisan CLI

## Introdução

Artisan é o nome da interface de linha de comando incluída no Laravel. Ela oferece uma série de comandos úteis para o desenvolvimento da sua aplicação. É impulsionada pelo poderoso componente Symfony Console.

## Uso

Para ver uma lista de todos os comandos disponíveis do Artisan, você pode usar o comando `list`:

**Listar todas as comandos disponíveis**

```sh
php artisan list
```

Cada comando também inclui uma tela de "ajuda" que exibe e descreve os argumentos e opções disponíveis do comando. Para visualizar uma tela de ajuda, basta antecipar o nome do comando com `help`:

**Visualizar a tela de Ajuda para um comando**

```sh
php artisan help migrate
```

Você pode especificar o ambiente de configuração que deve ser usado ao executar um comando usando a opção `--env`:

**Especificando o Ambiente de Configuração**

```sh
php artisan migrate --env=local
```

Você também pode visualizar a versão atual da sua instalação do Laravel usando a opção `--version`:

**Exibindo sua versão atual do Laravel**

```sh
php artisan --version
```
