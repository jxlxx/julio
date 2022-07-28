---
title: Golang Essentials
date: "2022-04-01"
description: A short list of the essential things to know about golang to have a good time.
tldr: Channels, goroutines, waitgroups
draft: true
tags: ["golang"]
---

This is a short and simple of the most essential concepts you to know about go to have a good time.

My notes are in my notebook, I'm gonna put them in here I promise.


# Goroutines

```go
go func () {
	// do stuff
}
```


# Channels

What exactly is a channel

You can have two kinds of channels: buffered and unbuffered.

this is how you read and write to a channel

Channels are really useful to make goroutines able to sort of share information with eachother


# WaitGroups

A waitgroup is a really simple idea but really powerful.

It allows you to safely control the behaviour of your goroutines in a really clean way


# Contexts

Contexts are another way to control the behavious of you goroutines

adding a time out

you need to check if the context has actually expired or been cancelled, otherwise nothing happens


# Interfaces

Accept Interfaces, Return Structs - Jim

Interfaces in go are essential

an example of how you can use an interface for testing



# Project Structure

command
	main.go

internal
	handlers.go
	
# Bazel
 
create a workspace file

then create a build file for the app

this makes it easier to run and share code i think





