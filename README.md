# Go System Design

## Key Features

- Domain Driven Design
- C4 Model
- Go Language
- Using Library https://github.com/goadesign/model

## Installations

```
go install goa.design/model/cmd/mdl@v1.7.6
go install goa.design/model/cmd/stz@v1.7.6
```

## Diagrams

To start a local HTTP server that serves a graphical editor:

```sh
mdl serve github.com/kecci/go-system-design/internal/doc -dir docs/diagrams/
```

To generate JSON artifact for uploading to [structurizr](https://structurizr.com/)

```sh
mdl gen github.com/kecci/go-system-design/internal/doc -out docs/model.json
```

## Sources

- https://github.com/MarioCarrion/todo-api-microservice-example
