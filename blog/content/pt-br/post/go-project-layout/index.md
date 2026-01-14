---
title: "Estrutura de Projeto Go: O Que Realmente Funciona"
description: "Padrões práticos para estruturar projetos Go baseados em Helm, Hugo e Prometheus, não no layout 'padrão' não oficial."
date: 2026-01-14
lastmod: 2026-01-14
draft: true
image: cover.jpg
categories:
  - Best Practices
tags:
  - project-structure
  - modules
  - tooling
---

A comunidade Go não tem uma estrutura de projeto oficial. Isso é intencional. Mesmo assim, desenvolvedores passam horas debatendo `pkg/` vs pacotes na raiz, quando a resposta depende do que você está construindo.

Aqui está uma visão prática sobre como estruturar projetos Go, baseada em padrões do Helm, Hugo, Prometheus e projetos menores.

## A única regra que importa

Go impõe exatamente uma regra estrutural: pacotes em `internal/` não podem ser importados de fora do módulo. Todo o resto é convenção.

A documentação oficial em [go.dev/doc/modules/layout](https://go.dev/doc/modules/layout) diz: comece simples. Um `main.go` e `go.mod` é suficiente para projetos pequenos. Adicione estrutura quando precisar, não antes.

## Três padrões que funcionam

### Biblioteca na raiz

Para projetos destinados a serem importados por outros.

```
mylib/
├── go.mod
├── mylib.go            # API principal
├── option.go           # Opções funcionais
├── types.go            # Tipos públicos
└── internal/           # Helpers privados
    └── util/
```

Usuários importam diretamente:

```go
import "github.com/user/mylib"

client := mylib.New(mylib.WithTimeout(5 * time.Second))
```

Exemplos: [Cobra](https://github.com/spf13/cobra), [Viper](https://github.com/spf13/viper), [goldmark](https://github.com/yuin/goldmark).

### Biblioteca + CLI

Para projetos que oferecem tanto uma biblioteca quanto uma ferramenta de linha de comando.

```
myproject/
├── go.mod
├── service.go          # API pública
├── types.go            # Tipos públicos
├── internal/
│   ├── config/         # Parsing de config
│   └── util/           # Helpers privados
└── cmd/
    └── mytool/         # Binário CLI
        └── main.go
```

O CLI é uma camada fina sobre a biblioteca. Usuários podem:

```bash
# Importar a biblioteca
go get github.com/user/myproject

# Instalar o CLI
go install github.com/user/myproject/cmd/mytool@latest
```

Exemplos: [Helm](https://github.com/helm/helm), [Hugo](https://github.com/gohugoio/hugo), [Prometheus](https://github.com/prometheus/prometheus).

### Apenas CLI

Para ferramentas que não são destinadas a serem importadas como bibliotecas.

```
mytool/
├── go.mod
├── main.go
├── internal/
│   ├── cmd/            # Implementações de comandos
│   ├── config/         # Configuração
│   └── core/           # Lógica de negócio
└── testdata/
```

Tudo em `internal/` sinaliza: isto é uma aplicação, não uma biblioteca. [Terraform](https://github.com/hashicorp/terraform) usa essa abordagem.

## Por que não pkg/?

O diretório `pkg/` adiciona um segmento de caminho sem benefício:

```go
// Com pkg/
import "github.com/user/project/pkg/thing"

// Sem pkg/
import "github.com/user/project/thing"
```

O segundo é mais curto e claro. O time Go explicitamente não recomenda `pkg/`. [Russ Cox](https://github.com/golang-standards/project-layout/issues/117), líder técnico do Go, criticou o repo [golang-standards/project-layout](https://github.com/golang-standards/project-layout) (54k+ stars) por promover padrões que o time Go nunca endossou.

Projetos modernos ([Hugo](https://github.com/gohugoio/hugo), [Prometheus](https://github.com/prometheus/prometheus), [Cobra](https://github.com/spf13/cobra)) não usam `pkg/`.

## Quando usar internal/

Use `internal/` para código que:

- Suporta sua API pública mas não deve ser exposto
- Pode mudar sem aviso
- Não tem garantias de estabilidade

Candidatos comuns:

- Parsing de configuração
- Funções utilitárias (manipulação de arquivos, strings)
- Implementações de protocolos
- Helpers de teste

Não use `internal/` para:

- Projetos pequenos onde funções não exportadas são suficientes
- Código que você pode querer expor depois (mover para fora de `internal/` é uma breaking change para seus imports)

## Quando criar um novo pacote

Crie um pacote quando:

- Múltiplos arquivos compartilham uma responsabilidade distinta
- O código tem seus próprios tipos e funções que formam uma unidade coesa
- Você quer controlar visibilidade nas fronteiras do pacote

Não crie um pacote para:

- Um único arquivo com algumas funções helper
- "Organização" sem benefício funcional
- Copiar a estrutura de algum projeto externo

Um arquivo de 200 linhas na raiz é ok. Um pacote `utils/` com três funções é over-engineering.

## O processo de decisão

Ao adicionar código:

| Pergunta                                    | Sim                                                          | Não                      |
| ------------------------------------------- | ------------------------------------------------------------ | ------------------------ |
| Usuários externos devem importar isso?      | Pacote raiz                                                  | `internal/`              |
| Isso é específico do CLI?                   | `cmd/appname/`                                               | Código de biblioteca     |
| Isso precisa de seu próprio pacote?         | Apenas se múltiplos arquivos com responsabilidade distinta   | Manter no pacote existente |

## Organização de arquivos na raiz

Para projetos de biblioteca, divida por responsabilidade:

```
mylib/
├── client.go           # Tipo Client e métodos
├── option.go           # Opções funcionais
├── types.go            # Tipos públicos
├── errors.go           # Erros sentinela
├── request.go          # Construção de requests
├── response.go         # Parsing de responses
└── mylib_test.go       # Testes
```

Cada arquivo tem uma responsabilidade. Procurando definições de erro? Veja `errors.go`. Procurando configuração? Veja `option.go`.

Evite:

- `util.go` (nomeie pelo que ele faz)
- `helpers.go` (mesmo problema)
- `misc.go` (onde código vai para morrer)

## Considerações para mono-repo

Para projetos com múltiplos binários:

```
myproject/
├── go.mod              # Módulo único
├── lib/                # Código de biblioteca compartilhado
├── cmd/
│   ├── server/
│   ├── worker/
│   └── cli/
└── internal/
    └── shared/         # Código interno compartilhado
```

Evite repos multi-módulo (múltiplos arquivos `go.mod`) a menos que tenha uma razão forte. Eles complicam:

- Desenvolvimento local (precisa de diretivas `replace`)
- Versionamento (tags devem incluir o caminho do módulo)
- Testes (`go test ./...` não funciona entre módulos)

O time Go recomenda repos single-module para a maioria dos projetos.

## Exemplos reais

| Projeto    | Estrutura                 | Biblioteca importável?                         |
| ---------- | ------------------------- | ---------------------------------------------- |
| Cobra      | Pacote raiz               | Sim, `github.com/spf13/cobra`                  |
| Helm       | `pkg/` + `cmd/`           | Sim, `helm.sh/helm/v3/pkg/action`              |
| Hugo       | Raiz + `hugolib/`         | Sim, `github.com/gohugoio/hugo/hugolib`        |
| Prometheus | Pacotes raiz              | Sim, `github.com/prometheus/prometheus/promql` |
| Terraform  | Tudo em `internal/`       | Não, apenas CLI                                |

## Erros comuns

- **Estruturar demais cedo**: começar com `pkg/`, `internal/`, `cmd/`, `api/`, `web/` para um projeto de 500 linhas. Comece simples, adicione estrutura quando a dor aparecer.
- **Copiar [golang-standards/project-layout](https://github.com/golang-standards/project-layout)**: esse repo não é endossado pelo time Go. Ele promove padrões que a maioria dos projetos Go não usa.
- **Pacotes vazios para uso futuro**: se um pacote tem um arquivo com duas funções, não é um pacote. É um arquivo.
- **internal/ para tudo**: se nada é importável, seu projeto é uma aplicação, não uma biblioteca. Tudo bem, mas seja intencional sobre isso.

## Resumo

- **Comece simples**: `main.go` + `go.mod` é válido
- **Código de biblioteca na raiz**: caminhos de import limpos
- **CLI em cmd/**: camada fina sobre a biblioteca
- **Código privado em internal/**: detalhes de implementação
- **Evite pkg/**: não adiciona nada
- **Um módulo**: evite complexidade multi-módulo

A melhor estrutura é aquela em que você não pensa. Se você está gastando tempo com estrutura em vez de código, está fazendo over-engineering.
