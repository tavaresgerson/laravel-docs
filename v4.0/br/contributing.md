# Contribuindo para o Laravel

- [Introdução](#introdução)
- [Pull Requests](#pull-requests)
- [Diretrizes de codificação](#diretrizes-de-codificação)

<a name="introdução"></a>

## Introdução

O Laravel é um software gratuito e de código aberto, o que significa que qualquer pessoa pode contribuir para seu desenvolvimento e progresso. O código-fonte do Laravel está atualmente hospedado no [Github](https://github.com/laravel), que oferece um método fácil para bifurcar o projeto e incorporar suas contribuições.

<a name="pull-requests"></a>

## Pull Requests

O processo de pull request difere para novos recursos e bugs. Antes de enviar um pull request para um novo recurso, você deve primeiro criar um problema com `[Proposta]` no título. A proposta deve descrever o novo recurso, bem como as ideias de implementação. A proposta será então revisada e aprovada ou negada. Uma vez que uma proposta seja aprovada, um pull request pode ser criado implementando o novo recurso. Pull requests que não seguem essa diretriz serão fechados imediatamente.

Peças de código para correções de bugs podem ser enviadas sem a criação de uma questão de proposta. Se você acredita que conhece uma solução para um bug que foi registrado no Github, por favor, deixe um comentário detalhando a correção proposta.

Adições e correções à documentação também podem ser feitas através do repositório de documentação [docs](https://github.com/laravel/docs) no GitHub.

### Solicitações de funcionalidades

Se você tiver uma ideia para um novo recurso que gostaria de ver adicionado ao Laravel, você pode criar um problema no Github com `[Request]` no título. O pedido de recurso será então revisado por um colaborador principal.

<a name="linhas-de-código"></a>

## Diretrizes de codificação

O Laravel segue as normas de codificação [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) e [PSR-1](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md). Além dessas normas, abaixo está uma lista de outras normas de codificação que devem ser seguidas:

- As declarações de namespace devem estar na mesma linha que `<?php`.
- A abertura da classe `{` deve estar na mesma linha que o nome da classe.
- A função e a estrutura de controle da abertura `{` devem estar em uma linha separada.
- Os nomes das interfaces são sufixados com `Interface` (`FooInterface`)
