# Migrações e semeadura

- [Introdução](#introdução)
- [Criando migrações](#criando-migrações)
- [Migrações em execução](#migrações-em-execucao)
- [Reverter migrações](#reverter-migrações)
- [Sementes de banco de dados](#database-seeding)

<a name="introdução"></a>

## Introdução

As migrações são um tipo de controle de versão para o seu banco de dados. Elas permitem que uma equipe modifique o esquema do banco de dados e fique atualizada sobre o estado atual do esquema. As migrações são tipicamente associadas ao [Construtor de Esquema](/docs/schema) para gerenciar facilmente o esquema da sua aplicação.

<a name="criando-migrações"></a>

## Criando migrações

Para criar uma migração, você pode usar o comando `migrate:make` na CLI do Artisan:

**Criando uma Migração**

```
php artisan migrate:make create_users_table
```

A migração será colocada na pasta `app/database/migrations` e conterá um timestamp que permite ao framework determinar a ordem das migrações.

Você também pode especificar a opção `--path` ao criar a migração. O caminho deve ser relativo ao diretório raiz da sua instalação:

```
php artisan migrate:make foo --path=app/migrations
```

As opções `--table` e `--create` também podem ser usadas para indicar o nome da tabela e se a migração criará uma nova tabela:

```
php artisan migrate:make create_users_table --table=users --create
```

<a name="running-migrations"></a>

## Migrações em execução

**Executando todas as migrações pendentes**

```
php artisan migrate
```

**Executando todas as migrações pendentes para um caminho**

```
php artisan migrate --path=app/foo/migrations
```

**Executando todas as migrações pendentes para um pacote**

```
php artisan migrate --package=vendor/package
```

> **Observação:**
>
>  Se você receber um erro de "classe não encontrada" ao executar as migrações, tente executar o comando 
>
> `composer dump-autoload`
>
> .

<a name="rolling-back-migrations"></a>

## Reverter as migrações

**Reverter a Última Operação de Migração**

```
php artisan migrate:rollback
```

**Reverter todas as migrações**

```
php artisan migrate:reset
```

**Reverter todas as migrações e executá-las novamente**

```
php artisan migrate:refresh

php artisan migrate:refresh --seed
```

<a name="database-seeding"></a>

## Sementes de banco de dados

O Laravel também inclui uma maneira simples de preencher seu banco de dados com dados de teste usando classes de semeadura. Todas as classes de semeadura estão armazenadas em `app/database/seeds`. As classes de semeadura podem ter qualquer nome que você desejar, mas provavelmente devem seguir alguma convenção sensata, como `UserTableSeeder`, etc. Por padrão, uma classe `DatabaseSeeder` é definida para você. A partir dessa classe, você pode usar o método `call` para executar outras classes de semeadura, permitindo que você controle a ordem de semeadura.

**Classe de Seed de Banco de Dados Exemplo**

```
class DatabaseSeeder extends Seeder {

	public function run()
	{
		$this->call('UserTableSeeder');

		$this->command->info('User table seeded!');
	}

}

class UserTableSeeder extends Seeder {

	public function run()
	{
		DB::table('users')->delete();

		User::create(array('email' => 'foo@bar.com'));
	}

}
```

Para semear seu banco de dados, você pode usar o comando `db:seed` na CLI do Artisan:

```
php artisan db:seed
```

Por padrão, o comando `db:seed` executa a classe `DatabaseSeeder`, que pode ser usada para chamar outras classes de semeadura. No entanto, você pode usar a opção `--class` para especificar uma classe de semeador específica para ser executada individualmente:

```
php artisan db:seed --class=UserTableSeeder
```

Você também pode semear seu banco de dados usando o comando `migrate:refresh`, que também fará o rollback e reexecutará todas as suas migrações:

```
php artisan migrate:refresh --seed
```
