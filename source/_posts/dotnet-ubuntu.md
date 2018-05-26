---
title: How to Run .Net Framework on Ubuntu
date: 2017-06-21 17:15:04
categories: Server
tags:
	- Ubuntu
	- .Net Framework
---
## Add the Mono repository to your system
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
echo "deb http://download.mono-project.com/repo/ubuntu xenial main" | sudo tee /etc/apt/sources.list.d/mono-official.list
sudo apt-get update
```
## Install Mono
```
sudo apt-get install mono-complete 
```
## Verify Installation (optional)

```csharp
using System;

public class HelloWorld
{
    static public void Main ()
    {
        Console.WriteLine ("Hello Mono World");
    }
}
```

To compile, use mcs:
```
mcs hello.cs
```
The compiler will create “hello.exe”, which you can run using:
```
mono hello.exe
```
The program should run and output:
```
Hello Mono World
```