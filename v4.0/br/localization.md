# Localização

- [Introdução](#introdução)
- Arquivos de idioma [Arquivos de idioma](#arquivos-de-idioma)
- [Uso básico](#uso-básico)
- [Pluralização](#pluralização)
- [Validação Localização](#validação)

<a name="introdução"></a>

## Introdução

A classe `Lang` do Laravel oferece uma maneira conveniente de recuperar strings em vários idiomas, permitindo que você suporte facilmente múltiplos idiomas em sua aplicação.

<a name="arquivos-de-linguagem"></a>

## Arquivos de Idioma

As cadeias de caracteres de idioma são armazenadas em arquivos dentro do diretório `app/lang`. Dentro deste diretório, deve haver um subdiretório para cada idioma suportado pelo aplicativo.

```
/app
	/lang
		/en
			messages.php
		/es
			messages.php
```

Os arquivos de linguagem simplesmente retornam um array de strings com chaves. Por exemplo:

**Exemplo de arquivo de linguagem**

```
<?php

return array(
	'welcome' => 'Welcome to our application'
);
```

O idioma padrão da sua aplicação é armazenado no arquivo de configuração `app/config/app.php`. Você pode alterar o idioma ativo a qualquer momento usando o método `App::setLocale`:

**Mudando o idioma padrão em tempo de execução**

```
App::setLocale('es');
```

<a name="uso-básico"></a>

## Uso básico

**Recuperando Linhas de um Arquivo de Idioma**

```
echo Lang::get('messages.welcome');
```

O primeiro segmento da cadeia passada para o método `get` é o nome do arquivo de linguagem, e o segundo é o nome da linha que deve ser recuperada.

> **Nota**
>
> : Se uma linha de idioma não existir, a chave será devolvida pelo método 
>
> `get`
>
> .

Você também pode usar a função `trans` do helper, que é um alias para o método `Lang::get`.

```
echo trans('messages.welcome');
```

**Fazendo Substituições nas Linhas**

Você também pode definir marcadores em suas linhas de idioma:

```
'welcome' => 'Welcome, :name',
```

Em seguida, passe um segundo argumento de substituições para o método `Lang::get`:

```
echo Lang::get('messages.welcome', array('name' => 'Dayle'));
```

**Determinar se um arquivo de idioma contém uma linha**

```
if (Lang::has('messages.welcome'))
{
	//
}
```

<a name="pluralização"></a>

## Pluralização

A pluralização é um problema complexo, pois diferentes idiomas têm uma variedade de regras complexas para a pluralização. Você pode gerenciar isso facilmente em seus arquivos de idioma. Usando um caractere "barra" (|) você pode separar as formas singular e plural de uma string:

```
'apples' => 'There is one apple|There are many apples',
```

Você pode, então, usar o método `Lang::choice` para recuperar a linha:

```
echo Lang::choice('messages.apples', 10);
```

Como o tradutor do Laravel é alimentado pelo componente de tradução do Symfony, você também pode criar regras de pluralização mais explícitas com facilidade:

```
'apples' => '{0} There are none|[1,19] There are some|[20,Inf] There are many',
```

<a name="validação"></a>

## Validação

Para a localização de erros e mensagens de validação, confira a <a href="/docs/validation#localization">documentação sobre Validação</a>.
