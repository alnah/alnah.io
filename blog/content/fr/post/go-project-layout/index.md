---
title: "Structure de projet Go : ce qui fonctionne vraiment"
description: "Patterns pratiques pour structurer vos projets Go, basés sur Helm, Hugo et Prometheus, pas sur le 'standard' non officiel."
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

La communauté Go n'a pas de structure de projet officielle. C'est intentionnel. Pourtant, les développeurs passent des heures à débattre de `pkg/` vs packages à la racine, alors que la réponse dépend de ce que vous construisez.

Voici une approche pratique pour structurer vos projets Go, basée sur les patterns de Helm, Hugo, Prometheus et de projets plus modestes.

## La seule règle qui compte

Go impose exactement une règle structurelle : les packages dans `internal/` ne peuvent pas être importés depuis l'extérieur du module. Tout le reste est convention.

La documentation officielle sur [go.dev/doc/modules/layout](https://go.dev/doc/modules/layout) dit : commencez simple. Un `main.go` et un `go.mod` suffisent pour les petits projets. Ajoutez de la structure quand vous en avez besoin, pas avant.

## Trois patterns qui fonctionnent

### Bibliothèque à la racine

Pour les projets destinés à être importés par d'autres.

```
mylib/
├── go.mod
├── mylib.go            # API principale
├── option.go           # Options fonctionnelles
├── types.go            # Types publics
└── internal/           # Helpers privés
    └── util/
```

Les utilisateurs importent directement :

```go
import "github.com/user/mylib"

client := mylib.New(mylib.WithTimeout(5 * time.Second))
```

Exemples : [Cobra](https://github.com/spf13/cobra), [Viper](https://github.com/spf13/viper), [goldmark](https://github.com/yuin/goldmark).

### Bibliothèque + CLI

Pour les projets offrant à la fois une bibliothèque et un outil en ligne de commande.

```
myproject/
├── go.mod
├── service.go          # API publique
├── types.go            # Types publics
├── internal/
│   ├── config/         # Parsing de config
│   └── util/           # Helpers privés
└── cmd/
    └── mytool/         # Binaire CLI
        └── main.go
```

Le CLI est une fine couche autour de la bibliothèque. Les utilisateurs peuvent :

```bash
# Importer la bibliothèque
go get github.com/user/myproject

# Installer le CLI
go install github.com/user/myproject/cmd/mytool@latest
```

Exemples : [Helm](https://github.com/helm/helm), [Hugo](https://github.com/gohugoio/hugo), [Prometheus](https://github.com/prometheus/prometheus).

### CLI uniquement

Pour les outils qui ne sont pas destinés à être importés comme bibliothèques.

```
mytool/
├── go.mod
├── main.go
├── internal/
│   ├── cmd/            # Implémentations des commandes
│   ├── config/         # Configuration
│   └── core/           # Logique métier
└── testdata/
```

Tout dans `internal/` signale : ceci est une application, pas une bibliothèque. [Terraform](https://github.com/hashicorp/terraform) utilise cette approche.

## Pourquoi pas pkg/ ?

Le répertoire `pkg/` ajoute un segment de chemin sans aucun bénéfice :

```go
// Avec pkg/
import "github.com/user/project/pkg/thing"

// Sans pkg/
import "github.com/user/project/thing"
```

Le second est plus court et plus clair. L'équipe Go ne recommande explicitement pas `pkg/`. [Russ Cox](https://github.com/golang-standards/project-layout/issues/117), le responsable technique de Go, a critiqué le repo [golang-standards/project-layout](https://github.com/golang-standards/project-layout) (54k+ stars) pour promouvoir des patterns que l'équipe Go n'a jamais approuvés.

Les projets modernes ([Hugo](https://github.com/gohugoio/hugo), [Prometheus](https://github.com/prometheus/prometheus), [Cobra](https://github.com/spf13/cobra)) n'utilisent pas `pkg/`.

## Quand utiliser internal/

Utilisez `internal/` pour du code qui :

- Supporte votre API publique mais ne devrait pas être exposé
- Peut changer sans préavis
- N'a pas de garanties de stabilité

Candidats courants :

- Parsing de configuration
- Fonctions utilitaires (manipulation de fichiers, chaînes)
- Implémentations de protocoles
- Helpers de test

N'utilisez pas `internal/` pour :

- Les petits projets où les fonctions non exportées suffisent
- Du code que vous pourriez vouloir exposer plus tard (sortir de `internal/` est un breaking change pour vos imports)

## Quand créer un nouveau package

Créez un package quand :

- Plusieurs fichiers partagent une responsabilité distincte
- Le code a ses propres types et fonctions formant une unité cohésive
- Vous voulez contrôler la visibilité aux frontières du package

Ne créez pas de package pour :

- Un seul fichier avec quelques fonctions helper
- De l'organisation sans bénéfice fonctionnel
- Copier la structure d'un projet externe

Un fichier de 200 lignes à la racine, c'est bien. Un package `utils/` avec trois fonctions, c'est de la sur-ingénierie.

## Le processus de décision

Quand vous ajoutez du code :

| Question                                     | Oui                                                          | Non                      |
| -------------------------------------------- | ------------------------------------------------------------ | ------------------------ |
| Les utilisateurs externes doivent importer ? | Package racine                                               | `internal/`              |
| C'est spécifique au CLI ?                    | `cmd/appname/`                                               | Code bibliothèque        |
| Cela nécessite son propre package ?          | Seulement si plusieurs fichiers avec responsabilité distincte | Garder dans le package existant |

## Organisation des fichiers à la racine

Pour les projets bibliothèque, découpez par responsabilité :

```
mylib/
├── client.go           # Type Client et méthodes
├── option.go           # Options fonctionnelles
├── types.go            # Types publics
├── errors.go           # Erreurs sentinelles
├── request.go          # Construction de requêtes
├── response.go         # Parsing de réponses
└── mylib_test.go       # Tests
```

Chaque fichier a une seule responsabilité. Vous cherchez les définitions d'erreurs ? Regardez `errors.go`. La configuration ? Regardez `option.go`.

Évitez :

- `util.go` (nommez-le selon ce qu'il fait)
- `helpers.go` (même problème)
- `misc.go` (là où le code va mourir)

## Considérations mono-repo

Pour les projets avec plusieurs binaires :

```
myproject/
├── go.mod              # Module unique
├── lib/                # Code bibliothèque partagé
├── cmd/
│   ├── server/
│   ├── worker/
│   └── cli/
└── internal/
    └── shared/         # Code interne partagé
```

Évitez les repos multi-modules (plusieurs fichiers `go.mod`) sauf raison impérieuse. Ils compliquent :

- Le développement local (besoin de directives `replace`)
- Le versioning (les tags doivent inclure le chemin du module)
- Les tests (`go test ./...` ne fonctionne pas entre modules)

L'équipe Go recommande les repos single-module pour la plupart des projets.

## Exemples réels

| Projet     | Structure                 | Bibliothèque importable ?                      |
| ---------- | ------------------------- | ---------------------------------------------- |
| Cobra      | Package racine            | Oui, `github.com/spf13/cobra`                  |
| Helm       | `pkg/` + `cmd/`           | Oui, `helm.sh/helm/v3/pkg/action`              |
| Hugo       | Racine + `hugolib/`       | Oui, `github.com/gohugoio/hugo/hugolib`        |
| Prometheus | Packages racine           | Oui, `github.com/prometheus/prometheus/promql` |
| Terraform  | Tout dans `internal/`     | Non, CLI uniquement                            |

## Erreurs courantes

- **Sur-structurer trop tôt** : commencer avec `pkg/`, `internal/`, `cmd/`, `api/`, `web/` pour un projet de 500 lignes. Commencez à plat, ajoutez de la structure quand la douleur apparaît.
- **Copier [golang-standards/project-layout](https://github.com/golang-standards/project-layout)** : ce repo n'est pas approuvé par l'équipe Go. Il promeut des patterns que la plupart des projets Go n'utilisent pas.
- **Packages vides pour usage futur** : si un package a un fichier avec deux fonctions, ce n'est pas un package. C'est un fichier.
- **internal/ pour tout** : si rien n'est importable, votre projet est une application, pas une bibliothèque. C'est bien, mais soyez intentionnel.

## Résumé

- **Commencez simple** : `main.go` + `go.mod` est valide
- **Code bibliothèque à la racine** : chemins d'import propres
- **CLI dans cmd/** : fine couche autour de la bibliothèque
- **Code privé dans internal/** : détails d'implémentation
- **Évitez pkg/** : ça n'apporte rien
- **Un seul module** : évitez la complexité multi-module

La meilleure structure est celle à laquelle vous ne pensez pas. Si vous passez du temps sur la structure plutôt que sur le code, vous sur-ingéniez.
