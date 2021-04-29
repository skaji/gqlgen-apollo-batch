# gqlgen tranport for apollo-link-batch-http

This is a [gqlgen](https://github.com/99designs/gqlgen) transport for [apollo-link-batch-http](https://www.apollographql.com/docs/react/api/link/apollo-link-batch-http/).

This transport supports not only apollo-link-batch-http requests but also normal JSON post requests,
so you can replace `transport.POST{}` by this transport.

# Usage

Server:

```go
package main

import (
	"log"
	"net/http"
	"time"

	"github.com/99designs/gqlgen/example/todo"
	"github.com/99designs/gqlgen/graphql/handler"
	"github.com/99designs/gqlgen/graphql/handler/extension"
	"github.com/99designs/gqlgen/graphql/handler/lru"
	"github.com/99designs/gqlgen/graphql/handler/transport"
	"github.com/99designs/gqlgen/graphql/playground"
	batch "github.com/skaji/gqlgen-apollo-batch"
)

func main() {
	srv := handler.New(todo.NewExecutableSchema(todo.New()))
	srv.AddTransport(transport.Websocket{
		KeepAlivePingInterval: 10 * time.Second,
	})
	srv.AddTransport(transport.Options{})
	srv.AddTransport(transport.GET{})
	srv.AddTransport(batch.POST{}) // replace transport.POST{} by our batch.POST{}
	srv.AddTransport(transport.MultipartForm{})
	srv.SetQueryCache(lru.New(1000))
	srv.Use(extension.Introspection{})
	srv.Use(extension.AutomaticPersistedQuery{
		Cache: lru.New(100),
	})

	http.Handle("/", playground.Handler("Todo", "/query"))
	http.Handle("/query", srv)
	log.Println("Accepting connections at http://127.0.0.1:8081")
	log.Fatal(http.ListenAndServe(":8081", nil))
}
```

Client:

```
# request
❯ curl -s http://127.0.0.1:8081/query -H 'Content-Type: application/json' -d '
[
  {"query": "query todo($id: ID!) { todo(id: $id) { text } }", "variables": {"id":1}},
  {"query": "query todo($id: ID!) { todo(id: $id) { text } }", "variables": {"id":2}}
]
'

# response
[
  {
    "data": {
      "todo": {
        "text": "A todo not to forget"
      }
    }
  },
  {
    "data": {
      "todo": {
        "text": "This is the most important"
      }
    }
  }
]
```

# Author

Shoichi Kaji

# License

MIT
