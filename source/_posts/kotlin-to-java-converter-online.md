---
title: How To - Kotlin to Java Converter Online
date: 17/01/2021 17:47:00
tags: java, kotlin, converter
---

So, there's at the time of writing two ways of converting `Kotlin` source files to `Java`. If you've got Android Studio installed, I'd recommend Option 1.

### Option 1

* Open Android Studio
* Menu - Tools > Kotlin > Kotlin Bytecode (assuming you have the Kotlin plugin)
* In the new window that's just opened, click the `Decompile` button
* Et voila! You now have a Java equivalent of your source ✔ 

### Option 2

* Convert your Koltin source file to bytecode 
* Use either IntelliJ Idea (IDE from JetBrains), or a open source decompiler such as [Fernflower](https://github.com/fesh0r/fernflower)


### Notes

* The Java output is likely quite *ugly*, unfortunately that's just the nature of a machine generating code 
* If the option is greyed out, ensure the Kotlin class is compiled
* The code could be misleading, and remove intentional patterns, ideally the code should be used to give you guidance but not as production ready code

