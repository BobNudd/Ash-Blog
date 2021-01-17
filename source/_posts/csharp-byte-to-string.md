---
title: How To - C# Convert Byte to String
date: 13/01/2021 17:47:00
tags: c sharp, c#
---

Fortunately .NET has some very simple API's to deal with most common encoding requirements, all available in the `System.Encoding` namespace.

TL;DR - If you simply want to perform this conversion, you can use the below which accepts 3 overloads

`var str = System.Text.Encoding.Default.GetString(b);`

If you're aware of the encoding of the byte array, you can explicitly use the character set, example:

`var str = System.Text.Encoding.UFT8.GetString(b);`

If you're unsure, UTF8 is most commonly used.

Likewise, to convert the string back into a byte array:

`byte[] byteArr = System.Text.Encoding.UTF8.GetString(str)`

Hope that helps!

