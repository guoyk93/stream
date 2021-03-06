# stream
Generics based Golang Stream API

## Requirements

* Golang 1.18+

## Features

* Preserving types while chaining, no `interface{}`
* `stream`, `map`, `collect` API
* Support `context.Context`
* Support `error` output

## Example

Find usernames starting with `a` of active users in `section-a` and `section-b`

```go
package main

import (
	"context"
	"github.com/guoyk93/stream"
	"log"
	"strings"
)

type User struct {
	Name   string
	Active bool
}

func main() {
	// mock a database
	var database = map[string][]User{
		"section-a": {
			{Name: "alice", Active: false}, {Name: "bob", Active: true},
		},
		"section-b": {
			{Name: "alex", Active: true},
		},
        "section-c": {
            {Name: "bob", Active: true},
        },
	}

	// build Stream[string] of sections
	sections := stream.Literal("section-a", "section-b")
	// build Stream[User] by query 'database'
	users := stream.Map(
		sections,
		func(ctx context.Context, section string) ([]User, error) {
			// this is just a mock, you can do actual database query here
			return database[section], nil
		},
	)
	// build Stream[string] of names by filter and map User.Name
	names := stream.Map(
		users,
		// SimpleMapper is just a wrapper to ignore ctx and error
		stream.SimpleMapper(func(u User) []string {
			if u.Active && strings.HasPrefix(u.Name, "a") {
				return []string{u.Name}
			} else {
				return nil
			}
		}),
	)
	// collect Stream[string] as []string
	// be advised, this is where actual operations are executed
	result, _ := stream.Collect(
		context.Background(),
		names,
		nil,
		stream.ToSlice[string](),
	)
	log.Println(result)
	// ["alex"]
}
```

## Credits

Guo Y.K., MIT License