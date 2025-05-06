---
date: '2025-05-05T23:15:33-03:00'
draft: false
title: 'Pycep a Criacao E Publicacao De Uma Lib Em Python'
author: "Erick Duarte"
description: ""
tags: []
categories: []
series: []
aliases: []
# cover:
#   image: images/msg.png
#   caption: "Generated using [OG Image Playground by Vercel](https://og-playground.vercel.app/)"
ShowToc: true
TocOpen: true
---

Quem nunca precisou fazer buscas de CEP, e o serviço de consulta estava indisponível? Daí surge a **pycep**, uma biblioteca escrita em python que objetiva fazer buscas em diversos serviços de maneira totalmente assíncrona.

Nesse post quero demonstrar algumas das etapas do processo de criação da biblioteca, como a concepção da ideia e também a publicação no **pypi**.

A ideia é simples, uma classe que carrega automaticamente os serviços disponíveis no diretório service como se fossem plugins, e os executa retornando os dados do serviço que responder primeiro.

Essa abordagem permite adicionar outros serviços sem maiores complicações, bastando implementar o protocolo **QueryService** e colocar o arquivo da classe no diretório services. E desta forma, ele será identificado pela biblioteca e será usado em novas consultas.

## A escolha do nome ideal

Pensei que o melhor nome da biblioteca fosse PyCEP, já que estamos falando de **Código de Endereçamento Postal**, mas o nome já estava em uso por um projeto no [PYPI](https://pypi.org/project/pycep/) .

## A criação de um repositório no Github

Não pretendo detalhar a criação do repositório de forma extensa, mas o repositório criado pode ser acessado em [https://github.com/erickod/pycep](https://github.com/erickod/pycep).

## A instalação do Poetry

O poetry é uma ferramenta para gerenciamento de pacotes no mundo python, que simplifica todo o processo de instalação, gerenciamento e publicação de pacotes.

Como o projeto depende fortemente do poetry, seja para configuração de algumas ferramentas ou mesmo para controle das dependências, é importante que ele esteja instalado e configurado.

Tudo que é necessário para a instalação pode ser encontrado em [https://python-poetry.org/docs/](https://python-poetry.org/docs/), mas vou tentar simplificar aqui.

Se você estiver utilizando Windows, abra o power shell e execute o comando abaixo:

```bash
(Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | py -
```

No entanto, se for um usuário do Linux, WSL ou do MacOS, execute este comando em um terminal:

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

Após a instalação, o comando **poetry** estará disponível para uso:

```bash
poetry --version
```

## Iniciando o projeto com o Poetry

Com o repositório criado, a primeira coisa que decidir fazer foi ir a um diretório qualquer e rodar alguns comandos no terminal.

No diretório escolhido, executei um simples **poetry new pycep**, e obtive um **Created package pycep in pycep**.

Ao abrir o diretório **pycep** criado pelo poetry, vamos notar alguns arquivos e subdiretórios:

-   **pycep** - Um diretório onde o código do projeto será criado
-   **pyproject.toml** - Um arquivo que armazena desde as versões do python, as dependências de desenvolvimento e de produção e a configuração de diversas ferramentas que projeto vai utilizar
-   **README.md** - Um arquivo para descrever o projeto, configurações para execução ou qualquer texto que quisermos colocar.
-   **tests** - Onde os testes automatizados serão criados e executados pelo test runner.

Com isso temos as seguintes linhas no **pyproject.toml**:

```toml
[tool.poetry]
name = "pycep"
version = "0.0.0"
description = ""
authors = ["Erick Duarte <erickod@gmail.com>"]
readme = "README.md"

[tool.poetry.dependencies]
python = "^3.11"

[tool.poetry.group.dev.dependencies]

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

O próximo passo seria 'comitar' o progresso feito até aqui, e para isso fiz o seguinte:

```bash
$ git init
$ git remote add origin https://github.com/erickod/pycep
$ git pull origin main
$ git branch --set-upstream-to=origin/main main
$ git add .
$ git commit -m "chore: add proetry project"
$ git push
```

## Instalando as dependências do projeto

Uma coisa é certa, o **static typing** pode fazer projetos terem mais qualidade, capturando erros sem que o projeto precise ser executado. Em ambientes python, existem diversas ferramentas que atingem esse objetivo, e neste caso vamos usar o **mypy**.

-   O **pytest** será utilizado como test runner;
-   O **black** será usado para formatação e estilo de código;

Para fazer a instalação das dependências, precisamos abrir algum terminal e executar o seguinte comando em nosso diretório de trabalho:

```bash
poetry add --group dev mypy pytest black
```

A execução do comando acima gera mudanças no **poetry.lock** e no pyproject.tom, e mais uma vez 'comitamos' as alterações;

```bash
git commit -m "chore: add mypy, black and pytest as dev dependencies"
```

E finalmente, para que a gente possa por nossas mãos no código, falta instalar uma lib que será usada em ambiente de produção, e para isso, vamos executar num terminal:

```bash
poetry add aiohttp
```

## A estrutura do Projeto

O conceito do projeto é bem simples, teremos uma classe CEP que fará algumas operações de validação e ou transformação do cep inputado, como converter CEP em inteiros para string ou outras validações relativas ao formato do CEP.

Teremos um **protocol / ABC** que servirá de contrato para cada serviço de consulta de CEP, e cada serviço implementado será armazenado no diretório **services**. Esse diretório será lido pelo **CepQueryServiceLoader**, que por sua vez será responsável por importar e carregar os serviços de consulta dinamicamente, e devolvê-los para o construtor da classe **pycep**.

Instanciado com a lista de serviços disponíveis, O **pycep** consultará cada serviço através do método **query\_cep**, de maneira assíncrona, e o retorno será dado pelo serviço que responder mais rápido.

Atualmente existem 3 serviços:

-   Correios
-   ViaCep
-   OpenCep

## Os protocolos e ABCs

Sendo úteis para ferramentas de checagem de tipos, usei Protocols para deixar explícito qual é a interface pública de certos objetos, dos quais gostaria de destacar os serviços de consulta

Os protocolos de certa forma se parecem com ABCs, e se formos olhar sua hierarquia de herança, veremos que em algum ponto ela herda de ABCMeta.

No exemplo abaixo, podemos ver exatamente como um serviço de consulta de cep deve ser parecer. O entrypoint para a consulta de um serviço qualquer ocorre através do método **query\_cep**, que por sua vez encapsula como o consulta ocorre.

Já o decorator **@runtime\_checkable** fará com que **isistance(MyInstance, QuerySerivce)** retorne um boolean, o que será útil em alguns momentos.

```
from typing import Protocol, runtime_checkable
from pycep.cep_data import CepData


@runtime_checkable
class QueryService(Protocol):
    async def query_cep(self, cep: str) -> CepData:
        pass
```

## Carregando os serviços de consulta

No diretório **pycep** existe um módulo chamado **service\_loader**. Nele, o objeto **CepServicesLoader** é responsável por carregar todos os módulos que estão em **services**. Ao carregar um módulo, chama a função construtora **make**, que por sua vez retornará o serviço de consulta corretamente instanciado e pronto para fazer consultas.

```python
import importlib
from pkgutil import walk_packages
from types import ModuleType
from typing import Any, Callable

from pycep import services
from pycep.protocols.query_service import QueryService


class CepQueryServiceLoader:
    """Import all sub modules dinamically"""

    def __init__(
        self, module: ModuleType = services, walk_packages: Callable = walk_packages
    ) -> None:
        self._services: list[Any] = []
        self._module = module
        self._walk_packages = walk_packages

    def load(self) -> Any:
        for loader, module_name, is_pkg in self._walk_packages(
            self._module.__path__, self._module.__name__ + "."
        ):
            submodule = importlib.import_module(module_name)
            service = submodule.make()
            self.__is_compatible(service) and self._services.append(service)
        return self._services

    def __is_compatible(self, module: QueryService, raises: bool = True) -> bool:
        is_compatbile = isinstance(module, QueryService)
        if raises and not is_compatbile:
            raise TypeError(f"{module} must implement QueryService")
        return is_compatbile
```

## A Publicação do pacote no pypi

A publicação se dá maneira bem simples, com alguns poucos passos, e envolve a execução de alguns comandos no poetry, a [criação de uma conta e a criação de um token no pypi](https://erickduarte.dev/blog/como-criar-uma-conta-no-pypi/), para que o poetry possa publicar pacotes.

1.  Criação da API token no PYPI
2.  Configuração do Poetry para usar o token: poetry config pypi-token.pypi
3.  Criamos o build do projeto: poetry build
4.  Publicação do Pacote: publicamos o pacote executando poetry publish
