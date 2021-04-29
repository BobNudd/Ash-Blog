---
title: Method Chaining with C#
date: 2021-04-18 18:36:00
tags: c sharp, c#
---

## Scenario

Jim 🧔 is making a sandwich, he already knows which ingredients his fridge contains which he'll be using:
- Mayonnaise ⬜
- Lettuce 🍃
- Cheese 🧀
- Ham 🥓

Jim will need to visit the fridge for each ingredient added to the sandwich. Each trip requiring a reference to the fridge, and removal of an item

*Could Jim, make this sandwich creation operation better?*

When we think about this scenario as code, it would be something like:

```csharp
    var fridge = new Fridge();

    fridge.RemoveMayonnaise();
    fridge.RemoveCheese();
    fridge.RemoveHam();
    fridge.RemoveLettuce();
```
<sub>Figure 1: fridge and removal of items</sub>

This pattern of calling multiple methods from an object is something very common in Object-oriented programming (OOP). Whilst perfectly valid, this suffers from some issues:

* We're *re-referencing* the object again and again, typing the word `fridge` and invoking a method each line
* We're creating a lot of noise and <ins>lines</ins>

Let's look at how method chaining can help us here.

<escape><!-- more --></escape>

If you've used anything from the `System.Linq` namespace, it's highly likely you've used method chaining, an example of this can be found below:

```csharp
            var people = peoples.Where(p => p.Age >= 30)
                .OrderByDescending(p => p.Name)
                .Take(10)
                .ToList();
```

<sub>Note: the example above uses fluent interfaces too, but we'll get into that another time.</sub>


Some benefits for method chaining are:

- To form a chain of available commands which are tied to an object
- Create a sequence of processing 'steps' which are optional and require no particular order
- Simpler and cleaner calling code

Sounds good? Let's get started.

### Improving the code in Figure 1

To create method chaining, there's two requirements for our code:

- Each method will need to have a return type of the class we're using
- At the end of each method, we'll need to return the current object instance

Let's modify the `RemoveMayonnaise` method with these changes:

```csharp
     public Fridge RemoveMayonnaise()
     {
         // implementation omitted for brevity
         return this;
     }
```

Line 1 - Here we're stating the object type which will be return
Line 3 - Here we're returning the instance of itself `this` from the method

Because we're returning the object `Fridge` and we're returning an instance, we can then access any public method within the `Fridge` instance, welcome to method chaining. Let's add the remaining methods for illustration below:

```csharp
   class Program
    {
        static void Main(string[] args)
        {
            // ### Previous code
            // var fridge = new Fridge();

            // fridge.RemoveMayonnaise();
            // fridge.RemoveCheese();
            // fridge.RemoveHam();
            // fridge.RemoveLettuce();

            // ### New code
            var fridge = new Fridge();

            fridge.RemoveMayonnaise()
                .RemoveHam()
                .RemoveCheese()
                .RemoveLettuce();
        }
    }


    class Fridge
    {
        public Fridge RemoveMayonnaise()
        {
            // implementation omitted for brevity
            return this;
        }

        public Fridge RemoveLettuce()
        {
            // implementation omitted for brevity
            return this;
        }

        public Fridge RemoveCheese()
        {
            // implementation omitted for brevity
            return this;
        }

        public Fridge RemoveHam()
        {
            // implementation omitted for brevity
            return this;
        }
    }
```

Our calling code under the `### New code` comment is smaller, more fluid and looks much more clean in comparison to `### Previous code`. It we wanted to, we could example put all the methods calls on one line.

### But Wait...


Jim is back 🧔, the <ins>**good news**</ins> is now thanks to our method chaining, Jim can gather his ingredients in one trip to the fridge. 

Now the <ins>**bad news**</ins>, from his last sandwich creation, placing the lettuce at the bottom of the bread makes it moist and threatens the structural integrity of the sandwich, therefore we need to use the ingredients in a specific order.



*But how can we enforce a sequence of events?*


Let's cover that in the next blog post, fluent interfaces!


