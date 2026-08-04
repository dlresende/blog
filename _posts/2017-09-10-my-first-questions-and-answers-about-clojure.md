---
layout: post
title: "My first questions (and answers) about Clojure"
date: 2017-09-10 00:00:00 +0000
tags: ["clojure", "faq", "fp", "functional programming"]
---

One of the services my current team maintains was written in Clojure. Being a big fan of meetups, user groups, conferences, and [brown-bag lunches](http://www.brownbaglunch.fr/), I have had the occasion to read some Clojure code in the past and write some hello worlds.

Given today we have this project, it is certainly a good time to learn Clojure for real!

I started then doing some research about Clojure, mainly about things one needs to know to get started. I turned some questions I've asked myself into a FAQ, which I'm publishing here mainly for self reference. If my initial research ends up helping someone else as a side effect, even better 😀

If you think something is not quite accurate or something really important is missing, let me know in the comments.

# What are the language's fundamentals?

- was created by [Rich Hickey](https://twitter.com/richhickey)
- is open source
- runs on the JVM
- has dynamic type system
- it is a functional programming language
- it is a general purpose language
- Clojure is based on [Lisp](https://common-lisp.net/)

# Does it have a build automation system?

[Leiningen](https://leiningen.org/) is for the Clojure ecosystem what [Maven](https://maven.apache.org/) and [Gradle](https://gradle.org/) are for Java. It helps developers to scaffold new projects, resolve dependencies, run tests, etc. Leiningen has a command line interface called lein.

```text
$ lein help
Leiningen is a tool for working with Clojure projects.

Several tasks are available:
change Rewrite project.clj by applying a function.
check Check syntax and warn on reflection.
classpath Print the classpath of the current project.
clean Remove all files from project's target-path.
compile Compile Clojure source into .class files.
deploy Build and deploy jar to remote repository.
# ...and many other tasks
```

Most Leiningen tasks apply in the context of a project. A project is a directory containing all the resources of an application, like source files, scripts, etc. The project's metadata is stored in a file called project.clj, which lives in the project's root directory.

[Here](https://github.com/technomancy/leiningen/blob/stable/doc/TUTORIAL.md#what-this-tutorial-covers)'s a nice tutorial about Leiningen.

There's also this other tool called [Boot](http://boot-clj.com/), but I haven't look at it yet.

# How do I install Clojure?

Clojure is a library managed as part of a project and [Leiningen](https://leiningen.org/) is the user interface to that library.

So if you are using MacOS, you can use [Homebrew](https://brew.sh/) to install it:

```bash
brew install leiningen
```

# How do I create a Clojure project?

You can use Leiningen to generate the scaffolding of a new Clojure application:

```text
$ lein new app my-project
Generating a project called my-project based on the 'app' template.
```

The example above uses a template called app, which creates applications. It is also possible to use templates to create libraries.

# Where can I learn Clojure?

To get a quick overview, I'd recommend [Learn X In Y Minutes](https://learnxinyminutes.com/docs/clojure/). Next, I'd recommend this post [Clojure From The Ground Up: Welcome](https://aphyr.com/posts/301-clojure-from-the-ground-up-welcome), by Kyle Kingsbury, a.k.a "Aphyr". Clojure From The Ground Up has [other chapters](https://aphyr.com/tags/Clojure-from-the-ground-up) worth going through as well.

When it comes to books, I've heard good things about [Living Clojure](https://www.goodreads.com/book/show/24701168-living-clojure), by Carin Meier (haven't read yet). There's also the [Clojure For The Brave and True](https://www.braveclojure.com/clojure-for-the-brave-and-true/) online book by Daniel Higginbotham.

# How do I get my feet wet?

If you cannot install Clojure/Leiningen, you can go to [tryclj.com](http://www.tryclj.com/) and start playing online.

There's also the [4Clojure](http://www.4clojure.com/) website where you can learn and practice by solving problems.

Otherwise, Leiningen has a REPL (Read-Evaluate-Print-Loop):

```text
$ lein repl
nREPL server started on port 55517 on host 127.0.0.1 - nrepl://127.0.0.1:55517
REPL-y 0.3.7, nREPL 0.2.12
Clojure 1.8.0
Java HotSpot(TM) 64-Bit Server VM 1.8.0_144-b01
 Docs: (doc function-name-here)
 (find-doc "part-of-name-here")
 Source: (source function-name-here)
 Javadoc: (javadoc java-object-or-class-here)
 Exit: Control+D or (exit) or (quit)
 Results: Stored in vars *1, *2, *3, an exception in *e

user=>
```

If you want to start a REPL without using Leiningen, check this [link](https://clojure.org/reference/repl_and_main#_launching_a_repl).

# Where can I find documentation?

[clojure.org](https://clojure.org/api/api) regroups a bunch of useful resources about Clojure. [This particular link](https://clojure.github.io/clojure/) is very useful during development, since it describes vars, types, macros and functions by namespace.

There's also the popular [ClojureDocs](https://clojuredocs.org/). ClojureDocs is a community driven website where you can find documentation and examples.

# How do I write tests in Clojure?

Clojure has its own unit testing library [clojure.test](https://clojure.github.io/clojure/clojure.test-api.html). It is somehow limited IMHO, but it does the job.

There's another library called [Midje](https://github.com/marick/Midje), which, from what I've understood, it's quite popular within the Clojure community. Midje aims to provide a more flexible and readable style of testing compared to clojure.test.

# How do I get code coverage?

[Cloverage](https://github.com/cloverage/cloverage) seems to be the most popular tool to extract code coverage.

# From where Clojure libs come from?

As of today, [Leiningen defaults to two repositories](https://github.com/technomancy/leiningen/blob/master/leiningen-core/src/leiningen/core/project.clj#L293-L297):

1. [Clojars](https://clojars.org/)
2. [Maven Central](http://search.maven.org/)

Clojars is the default repository for Clojure libraries.

I found some interesting points about Clojars in this [InfoQ interview with Alex Osborne](https://www.infoq.com/news/2009/11/clojars-leiningen-clojure), the creator of Clojars.

# Which text editor or IDE should I use?

There's [this answer on StackOverflow](https://stackoverflow.com/a/4248749/1779658) that lists the most used editors and IDEs. [This discussion on Reddit](https://www.reddit.com/r/Clojure/comments/2ovpmt/ask_reddit_which_editoride_you_write_clojure_code/) also gives an idea about what the Clojure community is using.

Being a Vim user for a while, I am going with [Neovim](https://neovim.io/)+ a couple of plugins:

- [vim-sexp](https://github.com/guns/vim-sexp)
- [vim-sexp mappings for regular people](https://github.com/tpope/vim-sexp-mappings-for-regular-people)
- [salve.vim](https://github.com/tpope/vim-salve)
- [fireplace.vim](https://github.com/tpope/vim-fireplace)

# Are there nice talks I can watch?

Among the talks I watched, I would recommend:

- [Radical Simplicity – Stuart Halloway](https://skillsmatter.com/skillscasts/2302-radical-simplicity)
- [Clojure for Java Programmers Part 2 – Rich Hickey](https://www.youtube.com/watch?v=hb3rurFxrZ8)
- [Clojure for Java Programmers Part 1 – Rich Hickey](https://www.youtube.com/watch?v=P76Vbsk_3J0&t=4804s)

The Changelog podcast has also compiled this [required viewing list](https://changelog.com/posts/rich-hickeys-greatest-hits) of videos from Rich Hickey.

And finally, there's this [ClojureTV channel on Youtube](https://www.youtube.com/user/ClojureTV/videos) where you might find some interesting stuff as well.
