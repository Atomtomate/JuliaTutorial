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
Generally, the code developed here will follow the principles of (Test Driven Development [TDD](https://en.wikipedia.org/wiki/Test-driven_development).
This style lends itself very well to functional languages, such as Julia.
One dezisive advantage for programmers without much experience with traditional software engineering practises is the automatic emergence of decoupled, pure functions.\
This effect is due to the cycle of having to define a test before being able to implement the feature.
TODO: finish this, explain TDD

# External Tools

We will employ a number of external services and tools in order to automate everything in the test and deployment cycle.
There are services known as continuous integration providers available, which check every git push that you make to a publicly available page like GitHub and perform serveral actions on that push.
Two well known services are [TravisCI](https://travis-ci.com/) and [GitHub Actions](https://github.com/features/actions). 
Here, we will focus on GitHub Actions, but the general setup remains similar between services.

One important thing to keep in mind is, that the services may need ssh keys to communicate with each other. 
So in addition to some dot-file placed in your repository, that the associated service will read and follow, we will also need to provide authentification keys to Github Actions or TravisCI.


## Continuous Integration

As explained above, we will focus on GitHub Actions as a continuous integration provider.
Furthermore, we can make use of already existing actions, as defined in [julia-actions](https://github.com/julia-actions).
Actions itself are packages of instructions that are cloned from github repositories on demand.
Many of these actions will be automatically employed after generating the GitHub Actions `.yml` files in section [Setting up a project](#markdown-header-setting-up-a-project).
One often needs to hand tune the setting, adding additional actions to the `.github/workflows/ci.yml` files.



## Version Control

Out version controll of choice will be [git](https://git-scm.com/) with [GitHub](https://github.com/) as a hosting provider.
GitHub is useful for a small project, because it also offers free static website hosting and a continous integration system, keeping a bunch of services under one name.


## Test Coverage

Providing a metric for test coverage is a form of common courtesy, providing some sort of guarantee, that users can rely on.
We will use two main projects, [coverall](https://coveralls.io/) and [codecov](https://codecov.io/).
Both coverage analysis tools will be triggered through our continuous integration provider and build into the documentation.


## Documentation (generation and deployment)

One of the most useful tools for the automated generation of documentation is [Documenter.jl](https://juliadocs.github.io/Documenter.jl/stable/). 
We will use the GitHubActions deployment of the documentation pages for our purposes, different deployment paths are given on the website for Documenter.jl.
An example configuration is given in the [Setting up a project](#markdown-header-setting-up-a-project) section.


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
In order to have an automated build and deploy process for the documentation, we will also add a `docs/make.jl` file with the following content (replace `YOUR NAME`, `TestPackage`, `YOUR_ACCOUNT` and `Test Package Name`):
```
using TestPackage
using Documenter

push!(LOAD_PATH, "../src/")
makedocs(;
    modules=[TestPackage],
    authors="YOUR NAME",
    repo="https://github.com/YOUR_ACCOUNT/TestPackage.jl/blob/{commit}{path}#L{line}",
    sitename="Test Package Name",
    format=Documenter.HTML(;
        prettyurls=get(ENV, "CI", nothing) == "true",
        canonical="https://YOUR_ACCOUNT.github.io/TestPackage.jl",
        assets=String[],
    ),
    pages=[
        "Home" => "index.md",
    ],
)

deploydocs(;
    repo="github.com/Atomtomate/EquivalenceClassesConstructor.jl",
```
Also make sure to create a file in `.github/workflows` called `Documenter.yml`. This configures the GitHub actions to build and deploy the documentation. This can also be done with different services, e.g. TravisCI.
Remember to check the `DocumenterTools` section on how to add the `DOCUMENTER_KEY` in the settings/sectrets tab of your GitHub repository, as well as the ssh key in the deploy keys tab.
The content of the workflow file should look something like this (for details, especially the `GITHUB_TOKEN` generation, visit [Documenter.jl/hosting](https://juliadocs.github.io/Documenter.jl/stable/man/hosting/)):
```
name: Documentation

on:
  push:
    branches:
      - main
    tags: '*'
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: julia-actions/setup-julia@latest
        with:
          version: '1.5'
      - name: Install dependencies
        run: julia --project=docs/ -e 'using Pkg; Pkg.develop(PackageSpec(path=pwd())); Pkg.instantiate()'
      - name: Build and deploy
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # For authentication with GitHub Actions token
          DOCUMENTER_KEY: ${{ secrets.DOCUMENTER_KEY }} # For authentication with SSH deploy key
        run: julia --project=docs/ docs/make.jl
```

Lastly make sure, that the github pages are enabled for the repository in question and the default pages point to the `gh-pages` branch. Also make sure to merge the pull request, that `Documenter.jl` opens ofter the first push.
