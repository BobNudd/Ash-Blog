---
title: Method Chaining & Fluent Interfaces
date: 2021-04-18 18:36:00
tags: c sharp, c#
---


By sheer frequency of use, the more classes and objects are consumed in conjunction with one another, the more inherit complexity and noise is created.
A saviour in the mist of object use are `fluent interface` and `method chaining` techniques. Let's dive into what these are.

### Chaining

If you've used anything from the `System.Linq` namespace, it's highly likely you've used method chaining, an example of this can be found below:

```csharp
            var people = peoples.Where(p => p.Age >= 30)
                .OrderByDescending(p => p.Name)
                .Take(10)
                .ToList();
```

The best use cases for method chaining are:

- Form a chain of available commands which are tied to an object of `T`
- Create a sequence of processing 'steps' which are optional and require no particular order

#### Example

Carrying on from our people example, lets create a `Person` entity with a few properties:

```csharp
    class PersonAmend
    {
        public void Name(People people) { }
        public void Age(People people) { }
        public void Telephone(People people) { }
        public void Address(People people) { }
        
    }
```

To use the above example methods for our `Person` object we would do the following:

```csharp
var personAmend = new PersonAmend();

personAmend.Name(myPerson);
personAmend.Telephone(myPerson);

personAmend.Age(myPerson);
personAmend.Address(myPerson);
```

Whilst not the most complex example, each method is invoked separately on a new line referencing the `personAmend` object per invocation, let's fix this by enabling method chaining for our `PersonAmend` class.

```csharp
    class PersonAmend
    {
        public Person Name(People people) 
        { 
            // implementation omitted
            return this;
        }

        public Person Age(People people) 
        {
            // implementation omitted
            return this;
        }

        public Person Telephone(People people)
        {
            // implementation omitted
            return this;
        }

        public Person Address(People people)
        {
            // implementation omitted
            return this;
        }
        
    }
```

Rather than `void` or some other type, all methods return the `Person` object, therefore we're always returning the same context and object.

We can now do the following:

```csharp
var personAmend = new PersonAmend();

personAmend.Name(myPerson)
.Telephone(myPerson);
.Age(myPerson);
.Address(myPerson);

// or 

var personAmend = new PersonAmend()
.Name(myPerson)
.Telephone(myPerson);
.Age(myPerson);
.Address(myPerson);
```

Great, but what about if there's a required sequence to these events, how would be ensure methods are called in a specific, necessary order? That's where fluent interfaces helps out.



