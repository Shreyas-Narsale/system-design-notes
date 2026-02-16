Design Patterns in Go
=====================

Repository Pattern
------------------

Interview Answer: Explain Repository Pattern in Go with example

The Repository Pattern separates business logic from data access by defining repository interfaces in the domain layer and implementing them in the infrastructure layer.
This allows your services to work with data without knowing how it’s stored, making your code testable, database-independent, and easier to maintain.

Key Ideas
---------

- Repository pattern abstracts data access logic from business logic.
- It acts as a middle layer between:
  - Service Layer → Repository Interface → Database
- Your service should NOT know:
  - SQL queries
  - ORM details
  - Table structure
  - DB driver
- It only knows:
  - `repo.Save(user)`
  - `repo.FindByID(id)`

Why It’s Important in Backend Systems
-------------------------------------

In production systems:

- DB may change (MySQL → PostgreSQL → MongoDB)
- You need unit testing without DB
- You may introduce caching later
- You may add transactions
- You may split into microservices

Core Idea
---------

Repository works with:

- Domain models
- Business concepts

NOT:

- Tables
- Rows
- SQL structs

Example
-------

Domain Model:

```go
package domain

type User struct {
    ID    int64
    Name  string
    Email string
}
```

Repository Interface (Domain Layer):

```go
package domain

import "context"

type UserRepository interface {
    Save(ctx context.Context, user *User) error
    FindByID(ctx context.Context, id int64) (*User, error)
    FindByEmail(ctx context.Context, email string) (*User, error)
    Delete(ctx context.Context, id int64) error
}
```

Notice:

- Uses `context.Context`
- Works with domain model
- No SQL visible

Infrastructure Implementation: (Database logic)

```go
package infrastructure

import "context"

type userRepository struct {
    db *sql.DB
}

func NewUserRepository(db *sql.DB) UserRepository {
    return &userRepository{db: db}
}

func (r *userRepository) FindByID(ctx context.Context, id int64) (*domain.User, error) {
    query := "SELECT id, name, email FROM users WHERE id = ?"

    row := r.db.QueryRowContext(ctx, query, id)

    var user domain.User
    err := row.Scan(&user.ID, &user.Name, &user.Email)
    if err != nil {
        return nil, err
    }

    return &user, nil
}
```

Real Production Folder Structure:

```
internal/
domain (Repository Interface)/
    user.go
    user_repository.go
service (Bussiness logic)/
    user_service.go
infrastructure(database logic)/
    mysql/
        user_repository.go
```

Flow of Request in Production:

HTTP Request
↓
Handler
↓
Service
↓
Repository (Interface)
↓
Infrastructure (MySQL / Mongo / Redis)
↓
Database

Advantages
----------

Easier to manage test cases and mock database with the repository interface.

```go
type MockUserRepo struct{}

func (m *MockUserRepo) FindByID(ctx context.Context, id int64) (*User, error) {
    return &User{ID: id, Name: "MockUser"}, nil
}

mockRepo := &MockUserRepo{}
service := UserService{repo: mockRepo}
```

Transactions Support:

```go
type userRepository struct {
    db *sql.DB
}

func (r *userRepository) WithTx(tx *sql.Tx) *userRepository {
    return &userRepository{db: tx}
}

func (s *UserService) CreateUser(ctx context.Context, user *User) error {
    tx, _ := s.db.BeginTx(ctx, nil)

    repo := s.repo.WithTx(tx)

    if err := repo.Save(ctx, user); err != nil {
        tx.Rollback()
        return err
    }

    return tx.Commit()
}
```

Adding Caching Without Changing Service Layer:

```go
type CachedUserRepo struct {
    next  UserRepository
    cache map[int64]*User
}

func (c *CachedUserRepo) FindByID(ctx context.Context, id int64) (*User, error) {
    if user, ok := c.cache[id]; ok {
        return user, nil
    }

    user, err := c.next.FindByID(ctx, id)
    if err == nil {
        c.cache[id] = user
    }
    return user, err
}

// main logic:

dbRepo := NewUserRepository(db)
cachedRepo := &CachedUserRepo{
    next:  dbRepo,
    cache: make(map[int64]*User),
}

service := UserService{repo: cachedRepo}
```

Switching Database Is Easy:

```go
// mysql
type MySQLUserRepo struct {
    db *sql.DB
}

// mongo
type MongoUserRepo struct {
    client *mongo.Client
}

// repository interface
type UserRepository interface {
    FindByID(ctx context.Context, id int64) (*User, error)
}

// main logic
func main() {
    db := NewMySQLDB()
    repo := NewUserRepository(db)
    service := UserService{repo: repo}
}
```

