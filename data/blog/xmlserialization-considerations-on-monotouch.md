---
title: 'XmlSerialization considerations (on MonoTouch)'
date: 2012-10-29 23:27
draft: false
tags: ['DotNet', 'MonoTouch', 'Xamarin']
summary: 'Xml serialization on MonoTouch for data storage.'
---

I recently was making an app which needed to store data on the client. The obvious choices were:

1. Local file storage (e.g. serialise a class to `XML` and store it in `Documents`)
2. Use an `SQLite` database to store the data and read/write when needed

I have done both before, and generally find that the `XML` storage is a quick and easy method of storing data on the client app - however it has it's problems...

## Why XmlSerialization is good

As stated above, `XmlSerialization` is quick and easy to implement.

Mono (.Net) has out of the box support for `XmlSerialization` and `System.IO` to read and write files, so it's very simple to create multiple files to store your data in and be done with it.

`XmlSerialization` is a perfectly natural, acceptable and encouraged way of saving files to file system. In fact, `XmlSerialization` works fine for _simple_ data storage where objects are individual, and belong to their parent object, and do not need to be referenced by any other object.

Here is a simple helper class I knocked up in 30 seconds:

```
using System;
using System.Xml.Serialization;
using System.IO;
using System.Xml;

public class FileHelper
{
	public void WriteToFile<T> (string filename, T item)
	{
		var serializer = new XmlSerializer (typeof (T));
		using (var stream = File.OpenWrite (filename))
		{
			serializer.Serialize(stream, item);
		}
	}

	public T ReadFromFile<T> (string filename) where T : class
	{
		var serializer = new XmlSerializer (typeof (T));
		using (var xreader = XmlReader.Create(File.OpenRead(filename)))
		{
			T item = serializer.Deserialize(xreader) as T;
			return item;
		}
	}
}
```

Consider the following simple forum domain model&trade;:

```
public class Topic
{
    public string Subject {
        get;
        set;
    }

    public List<Post> Posts {
        get;
        set;
    }
}

public class Post
{
    public string Body {
        get;
        set;
    }
}
```

Basically, a `Topic` contains an `List<Post>`, in Domain language `Topic.Posts` - logical, huh?

We can `XmlSerialize` that model, and we will get a nice `XML` file which looks like:

```
<?xml version="1.0" encoding="utf-8"?>
<Topic xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Subject>My first topic</Subject>
  <Posts>
    <Post>
      <Body>Hello world!</Body>
    </Post>
  </Posts>
</Topic>
```

OK. So we have a nice `XML` file which is easy to read and write to, easy to pass around, easy to code. And we have a nice little helper method to use to save this file.

## What's wrong with XmlSerialization

As I said before, nothing is wrong with `XmlSerialization`. Use it when the problem you need to solve is solvable by using it, but understand the drawbacks (a very non-committal tongue twister for you).

`XmlSerialization` has a series of pre-requisite rules which must be met in order to serialize a file. I'm not going to go into them all, but if you are interested here is the [MSDN article](http://msdn.microsoft.com/en-us/library/ms950721.aspx).

For me, the big ones have been:

1. Properties must be implementations, not interfaces. e.g. `List<T>` not `ICollection<T>` or `IList<T>`.
2. Relationships are not easily supported (without extra attributes)

Consider the following, more realistic domain model&trade;:

```csharp
public class User
{
    public string Username {
        get;
        set;
    }
}

public class Topic
{
    public string Subject {
        get;
        set;
    }

    public User Owner {
        get;
        set;
    }

    public List<Post> Posts {
        get;
        set;
    }
}

public class Post
{
    public User Owner {
        get;
        set;
    }

    public string Body {
        get;
        set;
    }
}
```

It's virtually the same as before, except it's slightly more realistic:

1. There is a new class, `User`.
2. `Topic` contains a new property `Owner` of type `User`.
3. `Post` contains a new property `Owner` of type `User`.

What we now have is a relational data structure.

O.M.G.!!

Using the same `FileHelper` let's save the model to an `XML` file, which results in:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Topic xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Subject>My first topic</Subject>
  <Owner>
    <Username>Macropus</Username>
  </Owner>
  <Posts>
    <Post>
      <Owner>
        <Username>Macropus</Username>
      </Owner>
      <Body>Hello world!</Body>
    </Post>
  </Posts>
</Topic>
```

At first glance, it looks all ok. But it's not...

## Users are not the same instantiation

Even though the .Net code uses the same instance (reference) of `User`, when the `XmlSerializer` serializes the data to the `XML` file, an element `<Owner>` is created in both `<Topic>` and `<Post>`.

This will result in the deserialization process instantiating 2 different `User` objects, which do not share the same reference.

You can get around this by writing some logic that stores only the User ID, and peppering your domain objects with business logic - but that's bad practise.

## Performance and Disk usage

Secondly - Imagine the size of the `XML` files (and the time to serialize and deserialize) if you had 1,000 topics, each with 100 posts and just 1 user...

(I originally tried 10,000 topics, but got bored of waiting)

iOS Simulator on my MacBook:

- Serialize and Write: 1671 milliseconds, filesize 15 megabytes
- Deserialize and Read: 4679 milliseconds

iPhone 4:

- Serialize and Write: 34400 milliseconds, filesize 15 megabytes
- Deserialize and Read: 83712 milliseconds

## Circular references

So, to extend our example further - we would now like to be able to reference all of the `User` objects `Post` objects, and we express this in our Domain Model as:

```
public class User
{
    public string Username {
        get;
        set;
    }

    public List<Post> Posts {
        get;
        set;
    }
}
```

Now if we try and serialize our model:

**_BANG_**

    System.InvalidOperationException: There was an error generating the XML document. ---> System.InvalidOperationException: A circular reference was detected while serializing an object of type User.

We have a circular reference, which `XmlSerialization` has no clue what to do with:

- `User.Posts.Owner`
- `Post.Owner.Posts[i].Owner`

There is no easy way to get around this using `XmlSerialization` without creating `Id` properties on your classes, and creating a pseudo-relational database in `XML` using `[XmlIgnore]` on your Domain Model. Again this is bad practise on a domain model (I have a thing for keeping my Domain Models 'pure').

To retain transparency, you could use a Binary Serializer to mitigate... But that's not the title of this post.

To [quote](http://lists.ximian.com/pipermail/monotouch/2009-September/000866.html) [Frank Krueger](http://praeclarum.org/) (Author of iCircuit and [SQLite-Net](https://github.com/praeclarum/sqlite-net)):

> BinarySerialization solves this problem but breaks when you start using events (it serializes your events which results in a large part of the heap put into the binary). You can't tell it to ignore events, instead, you have to play a game with events as fields - which takes about 8 lines of boilerplate per event.

> .NET serialization seems to have been developed with the idea of message passing and that's it. Serializing graphs is a miserable experience.

## Conclusion

So, what was all of this about? Well... I was rambling. This post was originally meant to be a comparison of `SQLite` and using `XmlSerialization` to store data in `XML` files on your iOS device.

I failed...

Instead, it's a dissection of the reasons why I use `XmlSerialization` in some projects, and not in others - the comparison will have to wait for another post in the near future!

For what it's worth, (and just to confuse) in my latest app (due to be released soon) I use both of these methods to store data, and files... More to come!

As always, [here is the source](https://github.com/darbio/Examples/tree/master/BlogPostExample)
