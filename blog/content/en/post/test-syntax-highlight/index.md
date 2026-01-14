---
date: '2025-01-14'
draft: true
title: 'Test Syntax Highlighting'
image: cover.jpg
---

Testing Go syntax highlighting with the Stack theme.

## Basic Function

```go
package main

import (
	"errors"
	"fmt"
)

// ErrUserNotFound is returned when a user cannot be found.
var ErrUserNotFound = errors.New("user not found")

// User represents a user in the system.
type User struct {
	ID    int
	Name  string
	Email string
}

// FindUserByID retrieves a user by their ID.
func FindUserByID(id int) (*User, error) {
	if id <= 0 {
		return nil, fmt.Errorf("invalid user id: %d", id)
	}

	// Simulate a database lookup
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

## Short Snippet

```go
for i := range 10 {
	fmt.Println(i)
}
```
