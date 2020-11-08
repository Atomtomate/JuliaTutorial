---
slug: "/JuliaTutorial/Introduction"
date: "2020-11-01"
title: "Introduction"
categories: ["Programming", "Julia"]
---

# Motivation

# Programming conventions

When it comes to developing and maintaining code, there are a lot of different viewpoints and conventions out there.
The field of software engineering has come up with a lot of development cycles, idoms, design patterns and project structures over the years.\
Since Julia packages are typically small in scale and the code can stay relatively concise, we will pick a single approach, that seems to fit this use case.
Generally, the code developed here will follow the principles of (Test Driven Development (TDD))[https://en.wikipedia.org/wiki/Test-driven_development].
This style lends itself very well to functional languages, such as Julia.
One dezisive advantage for programmers without much experience with traditional software engineering practises is the automatic emergence of decoupled, pure functions.\
This effect is due to the cycle of having to define a test before being able to implement the feature.
TODO: finish this, explain TDD

# External Tools

We will employ a number of external services and tools in order to automate everything in the test and deployment cycle.

## Version Control

## Test Coverage

## Documentation (generation and deployment)

## Continuous Integration


# Setting up a project

Julia is deployed in form of packages, that can be either obtained by sharing the source code or registering the package globally, making it available in the Julia package manager.
In order to provide guarantees about the functionality of our code and make sure it actually runs on all target platforms, we will use continuous integration in 

```
using Pkg
Pkg.add("PkgTemplates")
using PkgTemplates
t = Template(;dir="./test", plugins=[Git(;manifest=true, ssh=true), GitHubActions(), Documenter{GitHubActions}(), Develop(), Codecov(;file=".codecov.yml"), Coveralls(;file=".coveralls.yml")],)
t("TestPackage.jl")
```
