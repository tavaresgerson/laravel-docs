# paginação

- [Configuração](#configuração)
- [Uso](#uso)
- [Anexando Links de Paginação](#anexando-links-de-paginacao)

<a name="configuração"></a>

## Configuração

Em outros frameworks, a paginação pode ser muito trabalhosa. O Laravel facilita muito. Há uma única opção de configuração no arquivo `app/config/view.php`. A opção `pagination` especifica qual visual deve ser usado para criar links de paginação. Por padrão, o Laravel inclui duas visualizações.

A visualização `pagination::slider` exibirá um "intervalo" inteligente de links com base na página atual, enquanto a visualização `pagination::simple` exibirá simplesmente os botões "anterior" e "próximo". **Ambas as visualizações são compatíveis com o Twitter Bootstrap sem necessidade de configuração adicional.**

<a name="uso"></a>

## Uso

Existem várias maneiras de paginar itens. A mais simples é usar o método `paginate` no construtor de consulta ou em um modelo Eloquent.

**Paginando Resultados de Banco de Dados**

```
$users = DB::table('users')->paginate(15);
```

Você também pode fazer paginação de modelos [Eloquent](/docs/eloquent).

**Paginando um Modelo Eloquente**

```
$allUsers = User::paginate(15);

$someUsers = User::where('votes', '>', 100)->paginate(15);
```

O argumento passado para o método `paginate` é o número de itens que você deseja exibir por página. Depois de recuperar os resultados, você pode exibí-los em sua visualização e criar os links de paginação usando o método `links`:

```
<div class="container">
	<?php foreach ($users as $user): ?>
		<?php echo $user->name; ?>
	<?php endforeach; ?>
</div>

<?php echo $users->links(); ?>
```

Isso é tudo o que é necessário para criar um sistema de paginação! Observe que não tivemos que informar ao framework a página atual. O Laravel determinará isso automaticamente para você.

Você também pode acessar informações adicionais de paginação pelos seguintes métodos:

- `getCurrentPage`
- `getLastPage`
- `getPerPage`
- `getTotal`
- `getFrom`
- `getTo`
- `contagem`

Às vezes, você pode querer criar uma instância de paginação manualmente, passando-lhe um array de itens. Você pode fazer isso usando o método `Paginator::make`:

**Criando um Paginator manualmente**

```
$paginator = Paginator::make($items, $totalItems, $perPage);
```

**Personalizando o URI do Paginador**

Você também pode personalizar o URI usado pelo paginador via o método `setBaseUrl`:

```
$users = User::paginate();

$users->setBaseUrl('custom/url');
```

O exemplo acima criará URLs como as seguintes: <http://example.com/custom/url?page=2>

<a name="adicionando-links-de-paginação"></a>

## Adicionando links de paginação

Você pode adicionar ao string de consulta dos links de paginação usando o método `appends` no Paginator:

```
<?php echo $users->appends(array('sort' => 'votes'))->links(); ?>
```

Isso gerará URLs que parecem algo assim:

```
http://example.com/something?page=2&sort=votes
```
