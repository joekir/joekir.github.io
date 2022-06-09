---
layout: post
published: true
status: publish
title: 'Protecting data in a runtime environment: Part 2 - Transparent metadata wrappers'
type: post
largeimage: /assets/box.jpg
---
* Do not remove this line (it will not be displayed)
{:toc}

[Last post](/2022/05/14/protecting-data-in-a-runtime-environment){:target="_blank"} I wrote about some overview thoughts about data-protection in a programming environment, in this one though I want to zoom in on the last item in that post on custom ["boxing"](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/types/boxing-and-unboxing){:target="_blank"} of actual data to store metadata (that we already had at the database layer) on what entities should consume this data if we want to adhere to correct security and privacy practices.

The authentication of data access at runtime based on this metadata would be _icing on the cake_, but the basic goal would simply be that the metadata travels with the data, and any mergers of _datums_ (is that a word?) merges the metadata correctly.

## What would be the pre-requisites for using such a concept?

If we take a software company that might use this type of approach, they would need to be of a certain technology maturity level to be able to adopt. Here's the requirements

1. We have a datastore that has data in it that has privacy annotations (i.e. metadata).
2. There is some global authorization on how a given identity can access some piece of data based upon its metadata.

## What is the problem I'd like to have a better solution for?

_Visually seems to work best for me, this is also duplicated in the [code repo](https://github.com/joekir/data-boxes){:target="_blank"}_.    
I've tried to use the colours green to red to indicate the data-exposure risk, hopefully that is ~intuitive.

<div class="mermaid">
flowchart TB
    classDef vulnerable fill:#f1ac56
    classDef critical fill:#ffcccb
    classDef safe fill:#daf7a6
    classDef resetColor fill:#ffffff

    subgraph UI[frontend]
        raw7[raw-data]
    end
    service ----- raw1[raw-data] --> UI:::vulnerable

    subgraph context[service runtime]
        service --- raw2[raw-data] --> logger:::vulnerable
    end
    class context resetColor

    subgraph service
        raw5[raw-data]
    end
    class service resetColor

    subgraph logger
        raw6[raw-data]
    end
    
    service <-- AuthZ by Service </br> on behalf of User --> DB[(database 1)]
    class DB safe
    subgraph DB
        subgraph wrappedData[wrapped data]
            direction LR
            pii-attributes
            ownership-attributes
            raw4[raw-data]
        end 
    end
    class wrappedData safe
    service ----- raw3[raw-data] -- No User AuthZ because </br> no metadata attached ---> DB2[(database 2)]
    class DB2 vulnerable
    class raw1,raw2,raw3,raw4,raw5,raw6,raw7 critical
</div>

Notice that when the data is in the DB on the left it is appropriately described via metadata attributes, though because this is dropped on the floor when it is fetched the `logger`, `UI` and `database2` all do not benefit from it and hence can expose the data insecurely.


## Solution Overview

Therefore, if instead we required all data to be packaged up with it's metadata, and some ways in which metadata could be combined when data was combined in the code.
Then we could require that all places where raw data access is required e.g. `logger`, `UI`, `database2` need to know how to deal with reasoning about the attached metadata.


<div class="mermaid">
flowchart TB
    classDef wrapped fill:#fffacd
    classDef vulnerable fill:#f1ac56
    classDef critical fill:#ffcccb
    classDef safe fill:#daf7a6
    classDef resetColor fill:#ffffff

    subgraph UI[frontend]
        cleaned1[cleaned-data]
    end
    class UI resetColor
	
    service ----- cleaned2[cleaned-data] ---> UI

    subgraph context[service runtime]
        subgraph PA[Policy Agent]
        end
        service --- cleaned3[cleaned-data] --> logger:::vulnerable
    end
    class context,PA resetColor

	  subgraph service
        WD1[wrapped data]
    end
    class service resetColor
    WD1 <-- raw-data access </br> authenticated ---> PA

    subgraph logger
        cleaned4[cleaned-data]
    end
    class logger resetColor
    
    service <-- AuthZ by Service </br> on behalf of User --> DB1
    subgraph DB1[database 1]
        subgraph WD2[wrapped data]
            direction LR
            pii1[pii-attributes]:::resetColor
            own1[ownership-attributes]:::resetColor
            raw1[raw-data]
        end 
    end
    
	  service ----- WD3[wrapped-data] -- AuthZ by Service </br> on behalf of User ---> DB2[(database 2)]

    class raw1 critical
	  class cleaned1,cleaned2,cleaned3,cleaned4 safe
    class DB1,DB2 resetColor
    class WD1,WD2,WD3 wrapped
</div>

## So what does this solution look like to a developer?

I began to create a toy project around this at [github.com/joekir/data-boxes](https://github.com/joekir/data-boxes){:target="_blank"} with the following ideas to increase developer adoption:

1. The boxed object responds to all same public methods as the contained type, therefore code changes are minimized.
2. There is a negligible performance overhead through opting for compile-time "_magic_" over run-time "_magic_".

#2 needs a lot of work still, [manifold.systems](http://manifold.systems/){:target="_blank"} was the Java framework I found that best achieved the above so far, but this technique would not need to be in Java it's just the language I'm more comfortable with and can accomodate language manipulation more easily than some other options.

Here's an example of how it currently works:

```java
// Before
...
String id = "123456abcd";
Integer foo = fetchDataFromStore(id);

var bar = foo + 10;

System.out.println(bar);
...
```

```java
// After
...
String id = "123456abcd";
DataBox<Integer> foo = fetchDataFromStore(id);

var bar = foo + 10; // any operation on a DataBox<T> would also return a DataBox<T>

System.out.println(bar);
...
```   
<br/>

I've observed that tooling/framework changes that work well, are ones that can be progressively adopted, an example of this is Stripe's [Sorbet](https://sorbet.org){:target="_blank"} type checker for Ruby. 

If the adoption is optional, then each development team within an organization can migrate when it is appropriate based upon their other priorities, this is the antithesis of the "big-bang" approach ([Big-Bang Adoption](https://en.wikipedia.org/wiki/Big_bang_adoption){:target="_blank"}), which I've always seen fail.

## Closing thoughts

Re-stating the key assumptions for why I think a technique like this should be the future for enterprise software development:

1. 99.9999% of software developers are well-intentioned when it comes to security, if you make it easier (and not performance impacting) to adopt an approach they will try it.
2. It's already becoming commonplace to require tagging of PII at a datastore level.
3. The choice of authorization frameworks (cool one I saw recently is [Oso](https://www.osohq.com/){:target="_blank"}) is better and granular data-access is a topic that is more popular than I can ever recall.
4. In technology we see countless breaches of customer information that are the result of some data context (metadata) not being present for some evaluation of the data e.g. logging. Yet collectively as a business they did have that metadata present elsewhere in their systems.