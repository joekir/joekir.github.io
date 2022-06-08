---
layout: post
published: true
status: publish
title: 'Protecting data in a runtime environment: Overview'
toc: true
type: post
largeimage: /assets/cat_bag_1.jpg
---

People (me) often forget that <u>Information Security</u> (the field that I work in) has the word "**Information**" at the front of it. The reason it's there is that _at the end of the day_ the only thing that matters is ensuring that only the authorized people or services have access to the data that they're supposed to.

There are seemingly many attributes that go into determining whether a "piece" of data can be accessed, to name a few:

- who does this data belong to? Is this customer [PII](https://en.wikipedia.org/wiki/Personal_data)?
- what entities has the owner consented the permissions to read this data? (ideally explicitly, but in most cases in software implicitly)
- what is the entity trying to access the data? Where are they? etc

Let us propose that we have some way to express these rules and attach suitable attributes to data in our `<data stores>` (e.g. MySQL) and can enforce the controls on access.
What happens next?

_Usually what happens next:_ 
!["cat out of the bag"](/assets/cat_bag_1.jpg)

The data is relinquished to the caller and relying on hope to protect from accidental or malicious disclosure of the data to an unauthorized party.
An additional pain point is that all the data attributes that we described above usually do not travel with the data.

When it's in the database:
 
<div class="mermaid">
classDiagram
	class Data{
		data: Social Insurance Number
		attribute: Sensitivity Classification
		attribute: Owner
		attribute: Storage Region
		attribute: ...
	}
</div>

Once it's downloaded and being accessed by code:

<div class="mermaid">
classDiagram
	class data {
		Social Insurance Number
	}
</div>

Worse still, it's very common in technology businesses to have a piece of software download some data from one data store, massage and transform it, then upload it to some other data store.
This target data store would then have all the risk and non of the declarative attributes needed so that security systems can protect the data effectively.

## What's better?

Firstly it would be good if we could find out what the caller plans to do with the data, as that could allow us to provide a more useful menu of options.

### They want to reference the data but don't need to immediately do anything with it

Use an authenticated pointer. 

I don't mean this at the language level, like in C. But it's a good analogy.
This design seems to be reinvented countless times over in cloud software and therefore should probably be just called a pointer even though it's not in the same memory space.

<div class="mermaid">
classDiagram
	Data <|-- Pointer

	class Data{
		data: Social Insurance Number
		attribute: Sensitivity Classification
		attribute: Owner
		attribute: Storage Region
		attribute: ...
	}

	class Pointer{
		value: '0xf00B4D'
		pointer: *Data
		attribute: expiry
	}
</div>

Here the authorization occurs at time of "dereferencing" the pointer to access the data, so whatever system does that for you will need to know how to apply the authorization policy based upon the declarative attributes of the `Data`.

### They want to move it to some other data store that is aware of your data-store's existence

Again, use an authenticated pointer.

Here the code is simply passing on the message to say "here go get this thing from this place directly" and maintain all the properties surrounding it.

### They want to read, maybe modify some aspects of the data and then store elsewhere

There are some special use cases where you may be able to use homomorphic or partially [homomorphic encryption](https://en.wikipedia.org/wiki/Homomorphic_encryption) to allow some blinded manipulation of data. 
But here we're focussing on the day-to-day business cases that likely do not fit in that simple mathematical manipulation.

A desirable property is that at all times in our software runtime lifecycle the metadata stays with the data. 
Languages like C# and Java have this notion of ["boxing" and "unboxing"](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/types/boxing-and-unboxing), and this does seem to be like another useful analogy that could be used to describe this proposed design.

It should be possible to embed the data policy logic into the runtime itself such that when one of these objects is accessed, there can be an implicit check done against the boxed data object to confirm that the context trying to access the data is authorized. e.g.

```ruby
data = download(some_data)

# at this stage where the data needs to be accessed the runtime will ask
# 1. what context are we in as the caller?
# 2. what are the attributes associated with the data we're about to print?
#
# and then finally will determine if the security policy agrees this should proceeed 
print(data)
```
    
Similarly if this data should be written to some other data store, that data store will need an adapter that can authorize the sharing of this data and store it and all its associated attributes.

## Conclusion 

In reviewing available tooling to support this last case described I've come up empty.
I had a hunch that maybe some RASP ([Runtime Application Self Protection](https://en.wikipedia.org/wiki/Runtime_application_self-protection)) might have something like this or a way to extend to add, but nothing I found covered this angle.
If the reader is aware of such a framework please comment and share the knowledge! 
