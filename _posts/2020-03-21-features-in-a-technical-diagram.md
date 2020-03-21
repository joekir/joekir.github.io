---
layout: post
title: The features I look for in a technical diagram
comments: true
type: post
published: true
status: publish
---

I've seen so much variance on this in reviewing designs for software systems that it would be good to have a reference to point to of what I like to see and why I think it's beneficial.

## What diagram type should I chose? component or sequence?

![morpheus combine them](/assets/morpheus-what-if-i.jpg)

The value a sequence diagram provides is to show both space and time, as concurrency of requests is very important in software protocol design.The value of a component diagram is showing the ontology of the system.

To combine them, we can simply number each arrow in a component diagram.

## Detail the relationship cardinality

The icing on this cake is to collapse yet another dimension: "[Cardinality](https://en.wikipedia.org/wiki/Cardinality_(data_modeling))" into the diagram using [Crow's foot notation](https://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model#Crow's_foot_notation). This allows us to show what the relationship of each "entity" in the diagram should be, e.g. 1-to-many but the inverse could be simply only-1 or zero-to-1. 

## Generate the diagrams from text

Here's an example illustrating all this using my favourite diagramming tool PlantUML. ([Entity Diagrams - PlantUML](https://plantuml.com/ie-diagram))
It's my favourite tool as you can quickly modify or maintain a complex diagram by making very small changes to a text file. Additionally it's open-source meaning if you're diagramming sensitive intellectual-property you don't have to worry about it being stored by some other system.

I didn't want to make the example trite or stupid so I've kept it generic, but this wouldn't be an unusual diagram for some sort of web authentication flow or something.

```text
@startuml

entity "entity A" as A
 package foo {
  class "entity B" as B
  class "entity C" as C
}
entity "entity D" as D

A }o--> B: 1. Makes request to B for access
B --> C: 2. checks if this is ok?
C --> A: 3. asks A for information
A --> C: 4. A responds with request
C ||--> D: 5. C forwards info to D 

@enduml
```
<br>
which results in a diagram like this

![diagram](/assets/plantuml_entity_example.png)

Walking through the nice features we have in the above diagram
1. Time: thanks to the numbering we can follow the order of operation 
2. Hierarchy and ownership: entity B and C are in the same grouping "foo"
3. Cardinality: there can be zero-or-more connections to B, and there is one-and-only-1 connection to D for C 

There's one thing I did not capture here which is a slight limitation on either PlantUML system or my knowledge of it. You can't combine the crow-feet technique with an additional arrow on the connector to indicate the flow of data. The arrows in the above diagram are just a duplication of the numbering scheme.
