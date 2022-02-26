---
title: Lookup lists (using MongoDB)
date: 2014-11-10
draft: false
tags: ['MongoDb']
summary: 'An example of how to do lookups in MongoDB, without persisting the data or having to maintain relationships etc. in your services.'
---

# Darb.io Lookups (in MongoDB)

An example of how to do lookups in MongoDB, without persisting the data or having to maintain relationships etc. in your services.

## Background

In an object model we often want to specify a domain type (as in 'domain classification', not object `type`) to an object.

### Example

Take the example of an Organization. In our domain, Organizations have a name, and an organization type (classification). The organization types available are:

- Private Enterprise
- Government
- Not for profit

Historically, this relationship would probably have been either:

- Maintained in a lookup table
- Provided as an enumeration

## The problem

Each method has drawbacks:

### Enumeration

- An enumeration is a very basic string representation - what if I need more than a basic string?
- [Attributes on enumerations](http://stackoverflow.com/a/1799401/200653) can work, but are a messy way to persist meta data.

### Lookup table

- An interface is required to maintain the list (CRUD an organization type)
- You need to either:
  - Maintain a relationship on an object. In a RDMS world, this is easy, but what if you are using NoSQL? Relationships in NoSQL are not [wunderbar](http://en.wiktionary.org/wiki/wunderbar).
  - Maintain the object inside the NoSQL entity. This can lead to problems with data being out of sync.

## The solution

### Background

When I'm modelling (domain model, not catwalk... I gave that up) I like to try and keep as much of the domain model in [POCO form](http://en.wikipedia.org/wiki/Plain_Old_CLR_Object).

Relationships are mapped as references to other types, as per this pseudocode, which represents an prganization with an array of employees, and an organization type:

```
class organization {
  string name;
  organization_type type;
  person[] employees;
}

class organization_type {
  string identifier;
  string description;
}

class person {
  string name;
}
```

I also like to follow the following general architecture:

- Core (Interfaces, Domain model) (references only `system`)
- Persistence (Implementation of a repository, Entities) (references `core`)
- Application (references `core` and `persistence`)

### The nitty gritty

The solution in this repository allows us to persist the multiple `organization type domain objects` to a single `organization entity` in the persistence layer, with a `type` identifier on it.

Each individual `organization type` domain object is mapped using [AutoMapper](http://automapper.org/) to the `OrganizationTypeEntity` type in the database. The properties on the type are marked as `BsonIgnore`, which means the properties are not persisted to the datastore.

<script src="http://gist-it.appspot.com/https://github.com/darbio/Lookups/blob/master/darbio.Lookups.Persistence/Entities/OrganizationTypeEntity.cs">
</script>

![RoboMongo view](https://cloud.githubusercontent.com/assets/517620/4974288/5899dde0-691e-11e4-91b8-83836667ef69.png)

When an object is rehydrated from the database, the type mapping set up in the repository maps the object back to the specified type from the doamin model:

<script src="http://gist-it.appspot.com/https://github.com/darbio/Lookups/blob/master/darbio.Lookups.Persistence/Repositories/OrganizationRepository.cs?slice=37:53">
</script>

![Application view](https://cloud.githubusercontent.com/assets/517620/4974303/91e613fc-691e-11e4-9582-c98417c556df.png)

This allows us to add (and remove, with care) `organization types` from our code base, without having to maintain a lookup table. These lookup items can be edited and changed by editing the source code.

This pattern also ensures that there isn't needless replication between documents in the database, helping to keep the document size low and keep queries lean and efficient.
