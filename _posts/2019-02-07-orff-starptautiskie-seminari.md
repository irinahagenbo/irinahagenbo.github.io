---
layout: post
title: orff-starptautiskie-seminari
categories:
  - orff
---

In go shallow copying of _struct values_ can be considered as a feature built in the language. It is pretty obvious from language specification, that whenever you assign _struct_ to a new variable or pass it to a function _by value_ it gets copied.

So in simplest case you have to dereference pointer to a _struct_ type and assign it to a local variable.

Here is an example of how to achieve this.


<!--content-->


```go
package main

import "testing"

type simple struct {
	fld string
}

func (s *simple) copy() *simple {
	clone := *s   // This is where we essentially make a new struct
	return &clone // Return reference to a new struct
}

func TestCopy(t *testing.T) {
	original := &simple{"original"}
	copy := original.copy()
	copy.fld = "copy"

	if original.fld != "original" {
		t.Error("Original struct should retain it's state")
	}
	if copy.fld != "copy" {
		t.Error("Copy struct should have new state")
	}
}
```

But what if the actual type of the value that has to be cloned is not known, because it is passed as an _interface type_?

Since interface polymorphism does not discern value types and pointers it is impossible to simply dereference _interface type_, so the following code would not even compile:

```go
func copy(i interface{}) interface{} {
    clone := *s   // interface{} is not a pointer type, so it can't be dereferenced
    return &clone
}
```

This is when reflection in go comes handy!

Instead of trying directly trying to dereference _interface value_, we can use reflect.Indirect function, which will return dereferenced value in case argument is a pointer or argument as is if it is a value.

```go
package main

import (
	"testing"
	"reflect"
)

type simple struct {
	fld string
}

func clone(i interface{}) interface{} {
	// Wrap argument to reflect.Value, dereference it and return back as interface{}
	return reflect.Indirect(reflect.ValueOf(i)).Interface()
}

func TestCopy(t *testing.T) {
	original := &simple{"original"}

	copy := clone(original).(simple)
	copy.fld = "copy"

	if original.fld != "original" {
		t.Error("Original struct should retain it's state")
	}
	if copy.fld != "copy" {
		t.Error("Copy struct should have new state")
	}
}

```

In this example clone function will work with both pointers and values and return _interface type_, which essentially is a pointer to whatever is actually returned.
