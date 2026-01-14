---
date: '2025-01-14'
draft: true
title: 'Test Coloration Syntaxique'
image: cover.jpg
---

Test de la coloration syntaxique Go avec le theme Stack.

## Fonction de Base

```go
package main

import (
	"errors"
	"fmt"
)

// ErrUserNotFound est retourne quand un utilisateur est introuvable.
var ErrUserNotFound = errors.New("user not found")

// User represente un utilisateur dans le systeme.
type User struct {
	ID    int
	Name  string
	Email string
}

// FindUserByID recupere un utilisateur par son ID.
func FindUserByID(id int) (*User, error) {
	if id <= 0 {
		return nil, fmt.Errorf("invalid user id: %d", id)
	}

	// Simule une recherche en base de donnees
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

## Extrait Court

```go
for i := range 10 {
	fmt.Println(i)
}
```
