---
layout: post
title: "JUnit Flux Eclipse plugin"
date: 2011-09-12 00:00:00 +0000
tags: ["eclipse", "java", "junit", "junitflux"]
---

Last week during the Xebia Knowledge Exchange I discovered a very useful Eclipse Plugin called *junitflux*.

JUnit Flux Eclipse plugin (*junitflux*) runs JUnit tests every time you save a file. After saving a file containing a Java class, the plugin searches for the corresponding test class by adding the word 'Test" as a prefix or suffix of class name. Once the code change is made, the unit test result will immediately blow up in the screen.

![junitflux in action](/assets/img/junit-flux-eclipse-plugin/junitflux-in-action.png)

During the code retreat session, I installed the JUnit Flux Eclipse plugin in order to have quick feedback while coding. It works pretty well! Since feedback is an essential element in Agile Development, *junitflux* turns out to be very useful.

*junitflux* can be installed using Eclipse "Install New Software…" feature. To activate the plugin right-click your project and chose "Add JUnit Flux Nature" option. For more information visit the *junitflux* [project page](http://code.google.com/p/junitflux/).
