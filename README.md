# Go System Design

Example Project for System Design C4 Model with Golang Libary goadesign.

## Key Features

- Domain Driven Design
- C4 Model (https://c4model.com/)
- Go Language
- Using Library https://github.com/goadesign/model

## Installations

```
go install goa.design/model/cmd/mdl@v1.7.6
go install goa.design/model/cmd/stz@v1.7.6
```

## DSL Syntax / Functions

`internal/doc/doc.go`:
```go
package doc

import (
	"fmt"

	. "goa.design/model/dsl"
	"goa.design/model/expr"
)

var _ = Design("ToDo API Microservice Example", "Go microservice tutorial project using Domain Driven Design and Hexagonal Architecture!", func() {
	var (
		System               *expr.SoftwareSystem
		ElasticSearch        *expr.Container
		Kafka                *expr.Container
		PostgreSQL           *expr.Container
		RESTfulAPI           *expr.Container
		CLI                  *expr.Container
		ElasticSearchIndexer *expr.Container
	)

	const (
		containerRESTfulAPI           = "RESTful API"
		containerCLITool              = "CLI Tool"
		containerElasticsearchIndexer = "ElasticSearch Indexer"
		softwareSystem                = "ToDo System"
		restPackage                   = "internal.rest"
	)

	const (
		styleSoftwareSystem = "Software System"
		styleComponent      = "Component"
		styleContainer      = "Container"
		styleDatabase       = "Database"
		stylePerson         = "Person"
		styleCLI            = "Command Line Interface"
	)

	const (
		componentElasticsearch = "internal.elasticsearch"
		componentKafka         = "internal.kafka"
		componentPostgreSQL    = "internal.postgresql"
	)

	Person("User", "Interacts with Service", func() {
		External()

		Tag(stylePerson)

		Uses(softwareSystem, "Reads and writes tasks using", "HTTPS/JSON", Synchronous, func() {
			Tag("Relationship", "Synchronous")
		})

		Uses(fmt.Sprintf("%s/%s", softwareSystem, containerCLITool), "Reads and writes tasks using", "Go 1.16", Synchronous, func() {
			Tag("Relationship", "Synchronous")
		})

		Uses(fmt.Sprintf("%s/%s", softwareSystem, containerRESTfulAPI), "Reads and writes tasks using", "HTTPS/JSON", Synchronous, func() {
			Tag("Relationship", "Synchronous")
		})
	})

	System = SoftwareSystem(softwareSystem, "Allows users to interact with their ToDo Tasks", func() {
		URL("https://github.com/MarioCarrion/todo-api-microservice-example")

		PostgreSQL = Container("PostgreSQL", "Stores Tasks records", "PosgreSQL 12.5", func() {
			Tag(styleDatabase)
			Tag(styleContainer)
		})

		ElasticSearch = Container("ElasticSearch", "Stores searchable Task records", "ElasticSearch 7.x", func() {
			Tag(styleDatabase)
			Tag(styleContainer)
		})

		Kafka = Container("Kafka", "Streams Task events", "Kafka 2.13", func() {
			Tag(styleDatabase)
			Tag(styleContainer)
		})

		RESTfulAPI = Container(containerRESTfulAPI, "RESTful API", "Go 1.16", func() {
			Uses(PostgreSQL, "Reads from and Writes to", "SQL", Synchronous, func() {})
			Uses(ElasticSearch, "Reads from", "HTTPS", Synchronous, func() {})
			Uses(Kafka, "Produces", "Kafka", Asynchronous, func() {})

			Component(componentElasticsearch, "interacts with ElasticSearch", "Go Package", func() {
				Uses(ElasticSearch, "Uses", Synchronous, func() {
					Tag("Relationship", "Synchronous")
				})

				Tag(styleComponent)
			})

			Component(componentPostgreSQL, "interacts with PostgreSQL", "Go Package", func() {
				Uses(PostgreSQL, "Uses", Synchronous, func() {
					Tag("Relationship", "Synchronous")
				})

				Tag(styleComponent)
			})

			Component(componentKafka, "interacts with Kafka", "Go Package", func() {
				Uses(Kafka, "Uses", Asynchronous, func() {
					Tag("Relationship", "Asynchronous")
				})

				Tag(styleComponent)
			})

			Component("internal.service", "interacts with all datastores", "Go Package", func() {
				Uses(componentElasticsearch, "Reads records from", Synchronous, func() {
					Tag("Relationship", "Synchronous")
				})

				Uses(componentKafka, "Produce events to", Synchronous, func() {
					Tag("Relationship", "Synchronous")
				})

				Uses(componentPostgreSQL, "Uses", Synchronous, func() {
					Tag("Relationship", "Synchronous")
				})

				Tag(styleComponent)
			})

			Component("internal.rest", "defines HTTP handlers", "Go Package", func() {
				Uses("internal.service", "Uses", Synchronous, func() {
					Tag("Relationship", "Synchronous")
				})

				Tag(styleComponent)
			})

			Tag(styleContainer)
		})

		CLI = Container(containerCLITool, "CLI Tool", "Go Binary", func() {
			Uses(containerRESTfulAPI, "Uses", "HTTPS/JSON", Synchronous, func() {})

			Uses(fmt.Sprintf("%s/%s", containerRESTfulAPI, "internal.rest"), "Reads and writes tasks using", "HTTPS/JSON", Synchronous, func() {
				Tag("Relationship", "Synchronous")
			})

			Component("internal.service", "interacts with all datastores", "Go Package", func() {
				Tag(styleComponent)
			})

			Component("internal.cli", "defines HTTP handlers", "Go Package", func() {
				Uses("internal.service", "Uses", Synchronous, func() {
					Tag("Relationship", "Synchronous")
				})

				Tag(styleComponent)
			})

			Tag(styleCLI)
			Tag(styleContainer)
		})

		ElasticSearchIndexer = Container(containerElasticsearchIndexer, "Updates searchable tasks", "Go Binary", func() {
			Uses(ElasticSearch, "Writes to", "HTTPS", Synchronous, func() {})
			Uses(Kafka, "Consumes", "Kafka", Synchronous, func() {})

			Component(componentKafka, "interacts with Kafka", "Go 1.16", func() {
				Uses(Kafka, "Consumes events from", Asynchronous, func() {
					Tag("Relationship", "Asynchronous")
				})

				Tag(styleComponent)
			})

			Component(componentElasticsearch, "interacts with ElasticSearch", "Go Binary", func() {
				Uses(ElasticSearch, "Writes records to", Synchronous, func() {
					Tag("Relationship", "Synchronous")
				})

				Tag(styleComponent)
			})

			Tag(styleContainer)
		})

		Tag(styleSoftwareSystem)
	})

	Views(func() {
		SystemContextView(System, "ToDo System", func() {
			AddDefault()

			EnterpriseBoundaryVisible()
		})

		ContainerView(softwareSystem, "Containers", "Container diagram for the ToDo System", func() {
			AddDefault()

			SystemBoundariesVisible()
		})

		ComponentView(RESTfulAPI, "RESTful API", "Component diagram for the REST Server", func() {
			AddDefault()

			ContainerBoundariesVisible()
		})

		ComponentView(CLI, "CLI", "Component diagram for the CLI Server", func() {
			AddDefault()

			ContainerBoundariesVisible()
		})

		ComponentView(ElasticSearchIndexer, "ElasticSearch Indexer", "Component diagram for the Elasticsearch Indexer", func() {
			AddDefault()

			ContainerBoundariesVisible()
		})

		Styles(func() {
			ElementStyle(styleSoftwareSystem, func() {
				Background("#1168bd")
				Color("#ffffff")
			})

			ElementStyle(stylePerson, func() {
				Background("#08427b")
				Color("#ffffff")
				Shape(ShapePerson)
			})

			ElementStyle(styleComponent, func() {
				Background("#85bbf0")
				Color("#000000")
			})

			ElementStyle(styleContainer, func() {
				Background("#438dd5")
				Color("#ffffff")
			})

			ElementStyle(styleDatabase, func() {
				Shape(ShapeCylinder)
			})

			ElementStyle(styleCLI, func() {
				Shape(ShapeRoundedBox)
			})
		})
	})
})
Footer

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

## Output

`/docs/diagrams/<ViewName>.svg`

![image](docs/diagrams/ToDo%20System.svg)

![image](docs/diagrams/Containers.svg)

![image](docs/diagrams/RESTful%20API.svg)

![image](docs/diagrams/CLI.svg)

![image](docs/diagrams/ElasticSearch%20Indexer.svg)

## Sources

- https://github.com/MarioCarrion/todo-api-microservice-example
