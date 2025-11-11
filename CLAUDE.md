# Development Guidelines - Go + CQRS + Hexagonal Architecture

## Core Principles

### Hexagonal Architecture
- **Domain-first**: Business logic lives in the domain layer, isolated from infrastructure
- **Ports & Adapters**: Domain defines interfaces (ports), infrastructure implements them (adapters)
- **Dependency Inversion**: High-level modules don't depend on low-level modules
- **Testability**: Each layer can be tested in isolation

### CQRS/CQS Pattern
- **Commands**: Write operations that change state (CreateOrderCommand, CancelOrderCommand)
- **Queries**: Read operations that return data (OrderQueryService)
- **Separation**: Clear distinction between reads and writes
- **Command Handlers**: Each command has a dedicated handler with single responsibility

### Go Idiomatic Patterns
- **Package by Aggregate**: Organize by domain/business concept, not technical layers
- **Avoid Import Cycles**: Use dependency inversion and interfaces
- **Single Responsibility**: One package = one reason to change
- **Use internal/**: Prevent unwanted exports between packages
- **Use `any` over `interface{}`**: Always prefer `any` instead of `interface{}` for better readability

## Folder Structure Pattern

```
internal/
└── {aggregate}/                    # e.g., order, customer, product
    ├── {entity}.go                 # Domain entity (order.go)
    ├── {value_objects}.go          # Value objects (id.go, status.go)
    ├── repository.go               # Repository interface (port)
    ├── view.go                     # Query model interface (port)
    
    # Command Handlers (Write Operations)
    ├── creator/
    │   └── create_{entity}_command.go
    ├── canceller/
    │   └── cancel_{entity}_command.go
    ├── {action}er/                 # Pattern: action + "er" suffix
    │   └── {action}_{entity}_command.go
    
    # Query Services (Read Operations)  
    ├── reader/
    │   └── {entity}_query_service.go
    
    # Infrastructure Adapters
    ├── grpc/                       # gRPC transport
    │   ├── {entity}_server.go
    │   └── {action}_{entity}_handler.go
    ├── http/                       # HTTP transport  
    │   ├── server.go
    │   └── {action}_{entity}_handler.go
    ├── mongo/                      # MongoDB persistence
    │   └── {entity}_repository.go
    ├── kafka/                      # Event messaging
    │   └── {event}_handler.go
    
    # Bootstrap & Configuration
    └── bootstrap/
        ├── config.go               # Configuration struct
        ├── providers.go            # Dependency injection
        └── service.go              # Service orchestration
```

## Implementation Patterns

### 1. Domain Entity Example
```go
package order

type Order struct {
    aggregate.BaseAggregate  // Event sourcing support

    id     ID
    status Status
}

func New(id ID) Order {
    o := Order{
        id:     id,
        status: Created,
    }
    o.Record(event.OrderCreated{ID: string(id)})
    return o
}

func (o *Order) Cancel() error {
    err := o.status.changeTo(Cancelled)
    if err != nil {
        return err
    }

    o.status = Cancelled
    o.Record(event.OrderCancelled{ID: string(o.id)})
    return nil
}
```

### 2. Domain Entity Validation Pattern
Domain entities must follow this validation pattern:

```go
package order

import "errors"

var (
    ErrOrderIDBlank     = errors.New("order id cannot be blank")
    ErrOrderAmountZero  = errors.New("order amount must be greater than zero")
)

type NewOrderRequest struct {
    ID        string
    Amount    float64
    CreatedAt time.Time
    UpdatedAt time.Time
    // Other fields...
}

func NewOrder(req NewOrderRequest) (*Order, error) {
    o := &Order{
        id:        req.ID,
        amount:    req.Amount,
        createdAt: req.CreatedAt,
        updatedAt: req.UpdatedAt,
        // Initialize other fields...
    }
    return o, o.validate()
}

func (o *Order) validate() error {
    if o.id == "" {
        return ErrOrderIDBlank
    }
    if o.amount <= 0 {
        return ErrOrderAmountZero
    }
    return nil
}
```

**Key principles:**
- Define validation errors as package-level `var` declarations using `errors.New()`
- Use descriptive error names with `Err` prefix (e.g., `ErrOrderIDBlank`)
- Create the entity first, then validate using a private `validate()` method
- Return pattern: `return entity, entity.validate()` - entity is returned even on error
- Constructor receives a request struct with all required data including `CreatedAt` and `UpdatedAt`
- Never use `time.Now()` inside entity constructors - timestamps must come from the caller
- Keep validation errors simple and specific to the domain
- Tests should use `errors.Is()` to check for specific error types

### 3. Command Handler Pattern
```go
package creator

type CreateOrderCommand struct {
    ID string
    // Primitives only - converted to domain objects in handler
}

type CreateOrderCommandHandler struct {
    repository order.Repository
    dispatcher aggregate.EventDispatcher
}

func (h CreateOrderCommandHandler) Handle(ctx context.Context, cmd CreateOrderCommand) error {
    // 1. Convert primitives to domain objects
    id, err := order.NewID(cmd.ID)
    if err != nil {
        return fmt.Errorf("cannot handle create order command. id error: %w", err)
    }
    
    // 2. Execute domain logic
    o := order.New(id)
    
    // 3. Persist changes
    err = h.repository.Save(ctx, o)
    if err != nil {
        return fmt.Errorf("cannot handle create order command. repository error: %w", err)
    }
    
    // 4. Dispatch domain events
    h.dispatcher.Dispatch(ctx, o.GetEvents())
    
    return nil
}
```

### 4. Repository Interface (Port)
```go
package order

type Repository interface {
    ByID(ctx context.Context, id string) (Order, error)
    Save(ctx context.Context, order Order) error
}
```

### 5. Value Objects with Business Rules
```go
package order

type Status int

const (
    Created Status = iota
    Processing
    Completed
    Cancelled
    Returned
)

var validTransitions = map[Status][]Status{
    Created:    {Processing, Cancelled, Completed},
    Processing: {Cancelled, Completed},
    Completed:  {Returned},
    Cancelled:  {Returned},
    Returned:   {}, // Immutable final state
}

func (s *Status) changeTo(newStatus Status) error {
    statuses, ok := validTransitions[*s]
    if !ok {
        return StatusNotValid
    }
    
    for _, status := range statuses {
        if status == newStatus {
            return nil
        }
    }
    
    return TransitionNotValid
}
```

### 6. Bootstrap/Dependency Injection
```go
package bootstrap

func NewService(ctx context.Context, cfg Config) (*Service, error) {
    // 1. Infrastructure setup
    db, err := mongo.NewClient(cfg.MongoDB.URI)
    if err != nil {
        return nil, err
    }
    
    // 2. Repository adapters
    orderRepo := mongo.NewOrderRepository(db)
    
    // 3. Command handlers
    createHandler := creator.NewCreateOrderCommandHandler(orderRepo, eventDispatcher)
    
    // 4. Transport layer
    grpcServer := grpc.NewOrderServer(createHandler, cancelHandler)
    
    return &Service{grpcServer: grpcServer}, nil
}
```

## Naming Conventions

### Packages
- **Aggregates**: lowercase, singular (order, customer, product)
- **Command handlers**: action + "er" suffix (creator, canceller, returner, updater)
- **Query services**: "reader" for simple cases, or specific names (finder, searcher)

### Files
- **Domain entities**: `{entity}.go` (order.go)
- **Value objects**: descriptive names (id.go, status.go, email.go)
- **Commands**: `{action}_{entity}_command.go`
- **Handlers**: `{entity}_server.go`, `{action}_{entity}_handler.go`

### Types
- **Commands**: `{Action}{Entity}Command` (CreateOrderCommand)
- **Handlers**: `{Action}{Entity}CommandHandler` (CreateOrderCommandHandler)
- **Events**: `{Entity}{Action}` (OrderCreated, OrderCancelled)

## Testing Patterns
- Use mocks for interfaces (Repository, EventDispatcher)
- Test command handlers with different scenarios
- Test domain logic independently of infrastructure
- Use testify/mock or generate mocks with mockery

## Key Dependencies
- **Event Sourcing**: Custom aggregate.BaseAggregate
- **Configuration**: Koanf library for config management
- **Database**: MongoDB with custom repository implementations
- **Transport**: gRPC with protocol buffers
- **Testing**: Standard testing + mocks

## Error Handling
- Wrap errors with context: `fmt.Errorf("cannot handle command. reason: %w", err)`
- Domain errors are specific: `TransitionNotValid`, `StatusNotValid`
- Return errors from command handlers, don't panic

## Domain Events
- Record events in domain entities: `o.Record(event.OrderCreated{})`
- Dispatch events after successful persistence
- Events contain primitive data, not domain objects

## Git Commit Rules
- Use conventional commits format: `type: description`
- Use only `-m` flag for commit messages (no additional descriptions)
- Never include co-authored lines or Claude attribution
- Keep commit messages concise and clear
- **NEVER commit or push changes unless explicitly requested by the user**
- **NEVER create or update PRs unless explicitly requested by the user**

Example:
```bash
git commit -m "feat: add instance completion validation"
git commit -m "fix: handle null pointer in message handler"
git commit -m "test: add unit tests for IsComplete method"
```

## Pull Request Rules
- PR title should use conventional commit format
- Include only context explanation in PR description (no Summary section)
- Never include co-authored lines or Claude attribution
- Never merge a PR unless it is asked
- **NEVER create, update, or push to PRs unless explicitly requested by the user**

Example PR creation:
```bash
gh pr create --title "fix: handle completed instance errors gracefully" --body "$(cat <<'EOF'
Explanation of the problem and solution, including what was implemented and why it was needed.

EOF
)"
```

## Code Style Rules
- NEVER add comments to code unless explicitly requested by the user
- Code should be self-documenting through clear naming and structure
- Comments are considered noise and reduce code readability
- All files must end with a newline (blank line at the end)
