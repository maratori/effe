# Getting Started

## Install

```bash
go get github.com/GettEngineering/effe/cmd/effe
```

## Create new project

```bash
mkdir example && cd example
```

```bash
go mod init github.com/example/example
```

## Create repositories/structs

### repositories.go

```go
package mypackage

import (
	"context"
)

type UserRepository interface {
    Create(context.Context, User) error
}
```

### structs.go

```go
package mypackage

type User struct {
	Email    string
	Password string
}

type UserAttributes struct {
	Email    string
	Password string
}
```

## Create steps

### steps.go

```go
package mypackage

import (
	"context"
)

func buildUser() func(UserAttributes) User {
	return func(uAttrs UserAttributes) User {
		return User{
			Email:    uAttrs.Email,
			Password: uAttrs.Password,
		}
	}
}

func createUser(userRepo UserRepository) func(context.Context, User) error {
	return func(ctx context.Context, user User) error {
		return userRepo.Create(ctx, user)
	}
}
```

## Define Workflow

### effe.go

```go
// +build effeinject

package mypackage

import (
    "github.com/GettEngineering/effe"
)

func BuildCreateUserFlow(uAttrs UserAttributes) error {
	effe.BuildFlow(
		effe.Step(buildUser),
		effe.Step(createUser),
	)
	return nil
}
```

## Generate flow

run `effe` command

```bash
$ effe
effe: wrote /app/effe_gen.go
```

### effe_gen.go

```go
// Code generated by Effe. DO NOT EDIT.

//+build !effeinject

package mypackage

import (
	"context"
	"fmt"
	"github.com/example/example"
)

func BuildCreateUserFlow(service BuildCreateUserFlowService) BuildCreateUserFlowFunc {
	return func(ctx context.Context, UserAttributesVal UserAttributes) error {
		UserVal := service.BuildUser(UserAttributesVal)
		err := service.CreateUser(ctx, UserVal)
		if err != nil {
			return err
		}
		return nil
	}
}
func NewBuildCreateUserFlowImpl(userRepo UserRepository) *BuildCreateUserFlowImpl {
	return &BuildCreateUserFlowImpl{buildUserFieldFunc: buildUser(), createUserFieldFunc: createUser(userRepo)}
}

type BuildCreateUserFlowService interface {
	BuildUser(uAttrs UserAttributes) User
	CreateUser(ctx context.Context, user User) error
}
type BuildCreateUserFlowImpl struct {
	buildUserFieldFunc  func(uAttrs UserAttributes) User
	createUserFieldFunc func(ctx context.Context, user User) error
}
type BuildCreateUserFlowFunc func(ctx context.Context, UserAttributesVal UserAttributes) error

func (b *BuildCreateUserFlowImpl) BuildUser(uAttrs UserAttributes) User {
	return b.buildUserFieldFunc(uAttrs)
}
func (b *BuildCreateUserFlowImpl) CreateUser(ctx context.Context, user User) error {
	return b.createUserFieldFunc(ctx, user)
}
```
