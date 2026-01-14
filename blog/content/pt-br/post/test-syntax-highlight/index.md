---
date: '2025-01-14'
draft: true
title: 'Teste de Syntax Highlighting'
image: cover.jpg
---

Testando syntax highlighting de Go com o tema Stack.

## Funcao Basica

```go
package main

import (
	"errors"
	"fmt"
)

// ErrUserNotFound e retornado quando um usuario nao pode ser encontrado.
var ErrUserNotFound = errors.New("user not found")

// User representa um usuario no sistema.
type User struct {
	ID    int
	Name  string
	Email string
}

// FindUserByID recupera um usuario pelo seu ID.
func FindUserByID(id int) (*User, error) {
	if id <= 0 {
		return nil, fmt.Errorf("invalid user id: %d", id)
	}

	// Simula uma busca no banco de dados
	users := map[int]*User{
		1: {ID: 1, Name: "Alice", Email: "alice@example.com"},
		2: {ID: 2, Name: "Bob", Email: "bob@example.com"},
	}

	user, ok := users[id]
	if !ok {
		return nil, ErrUserNotFound
	}

	return user, nil
}

func main() {
	user, err := FindUserByID(1)
	if err != nil {
		fmt.Printf("error: %v\n", err)
		return
	}
	fmt.Printf("Found user: %s <%s>\n", user.Name, user.Email)
}
```

## Trecho Curto

```go
for i := range 10 {
	fmt.Println(i)
}
```
