# Minions

## TL;DR

Minions is a filesystem-based, microservice oriented security scanner.
It supports distributed security checks and isolates testing and
data access via [gRPC](https://grpc.io), can be *easily* extended and is privacy mindful.

![High level schema of Minions](https://docs.google.com/drawings/d/e/2PACX-1vQubAAchbeeLMqjq0-uRYeMH4IFKQOoe8bYlHLtImGgidVWBD8UXWmvWyD9v6fHYxCpHs8s3OxY8HSJ/pub?w=363&amp;h=150)

## Status

We are actively opensourcing existing code and building new one, but the project is yet to hit the first full release (0.1).

Full roadmap [here](roadmap.md).

## Core concepts

Much like ancient Gaul, a Minions infrastructure is divided in three main components: the Goblins, the Overlord and the Minions.

A **Goblin** is responsible for data access: it reads filesystem data and metadata and makes it available to the scanners. A Goblin is entirely independent from the rest of the scanning infrastructure, and as such can take privacy preserving decisions: for example, never let the scanners access a Home directory, or source code.

A **Minion** proper (I know, the project is also called Minions) is the actual scanner. It receives file data and metadata, does whatever it needs to and returns back - if any were found - vulnerabilities. A minion has only as much context as it specifically asks for about the target of a scan (more about this below). This allows Minions to be laser-focused on the tasks of detecting vulnerabilities, without all the classic overhead that comes with any, even trivial, scanner.

Finally, the **Overlord** is the orchestrator of the infrastructure, in charge of managing incoming scan requests, routing them to Minions and so forth.

## Why does this project matter

Unlike traditional on-host security scanners, Minions minimizes the amount of code that needs to be executed on the target, and it's very easy to implement a new Goblin for a specific environment. All the complex logic is in the Minions, and users can maintain control of what goes where by running their own Goblin and Overlord.

Minions also easily supports non-public scanners: adding a new tester using custom technology is as easy as implementing a well defined gRPC API.

### Interests

Separating data-gathering and actual testing of the data seems like a good idea on paper, but in practice has (at least) two main problems:

* It can be unnecessarily very expensive, as a lot of data that
  might or might not be useful need to be gathered upfront.
* The set of data to gather might be different depending on
  properties of the data itself. For example, a config file at a
  standard location might point to another configuration file in some directory hidden in a dark corner of the disk.

Minions solves this problem with the use of *Interests*. An interest is a way for a Minion to tell a Goblin what it cares about at a given moment. All Minion instances start with a set of initial Interests they always care about, but the list is iteratively updated as they process files they have ingested.

THe way this works in practice is that every time a Goblin sends files to an Overlord, it waits until the backend Minions have processed it and can be served back a new list of files to provide - and so forth until all minions have completed.

## Building and running

Minions is a set of microservices. You'll have to run at least 2 components to get aything useful: an Overlord, and one or more Minions.

### Overlord

Start by running the overlord, the orchestrator of the system.

TODO(paradoxengine): Complete this section :-)

### Minion

You can run one or more Minions - and you can run as many replicas of a minion as you want, spreading the load.

Each Minion carries its own set of flags and configs, but all need to be pointed to the Overlord, where they'll register.

TODO(paradoxengine): Complete this section.

#### Replicas

If you run more than a single replica of a minion, and if a minion keeps state, you'll want to have a shared backend.

TODO(paradoxengine): explain how one can know and what to do :)

#### Minion details: Vulners

The Vulners minion parses package databases on Linux systems to identify the presence of outdated software that carries security vulnerabilities. To do so it needs to parse the RPM backend - which it does using the RPM libraries.
Sadly, this means that building it is non hermetic, as the system will have to provide the rpm lib. On Debian/Ubuntu system, that means you need to make sure you have the librpm-dev package installed to build it.

### Goblin

Once you have the address of an Overlord with a set of working Minions you can run a Goblin to feed data to it.

The simplest Goblin available is the Local Goblin, which fetches files from the filesystem of the box it runs on. To run it, enter the main src directory and run the following:

```bash
bazel run //go/goblins:goblins
```

## Developers

We warmly welcome contributions, in particular of additional detectors (which are hopefully fairly easy to write once you get the hang of the APIs). Please read the [contributing](CONTRIBUTING.md) policy first.

### Build environment

Minions has been developed using [Bazel](https://bazel.io), an opensource build infrastructure by Google. Bazel can compile multiple languages using a common idiom cross platform, which is a nice property to have for Minions.

The Go code also builds and runs with the native compiler. In fact, one can have both working at the same time - which is particularly useful if one wants to develop with something like VS Code - with 2 tricks:

* Have a symlink from somewhere in the gopath from src/github.com/google/minions to the src directory where the code is checked out.
* Set gopath to include /src/bazel-bin/gopath, which is where the go dependencies will be copied by gopath (see below)

Now, simply blaze building the gopath target:

```bash
bazel build //:gopath
````

## Notice of affiliation 

This is *not* an official Google product.
