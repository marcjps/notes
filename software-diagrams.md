# Software Diagrams

Software diagrams, in order of usefulness.

# Architecture diagram

The c4model is a good way to describe the architecture of complex software systems.  https://c4model.com/  This is the diagram to use when the boss wants to understand what the components of the system are.

The concept of the c4 model is to have multiple diagrams.  Each diagram has boxes, called "containers", and shows the dependency or data flow between them using arrows.  For each container you then have another diagram if necessary, showing further detail of the parts inside the container.

The first level is usually a System Context Diagram which shows how different systems relate to each other.  There are usually no components or databases shown at this level, just the systems themselves.  Then the second level shows the "containers" which make up one of the systems.  A container is usually a large component, like a web application, an API, or a database.   Finally, if you need to break down further, you can do further diagrams explaining the components within a container.  (E.g. an Web Application may contain several functions or pages, and a database may contain several tables)  However I find it is never necessary to draw at the lowest levels of detail.  (like class diagrams, nobody really needs that diagram)

Another useful feature of the c4model is it has a text description of each container as well as it's name.  If you write an "explain like i'm five" in each box then the diagram is extremely accessible to anyone who might not recognise the names of the components.

c4model diagrams are fantastic because they are easy to draw and easy to understand, and extremely useful for describing software architecture.

I found the following ideas useful when drawing this sort of diagram:

* It's a good idea to understand what the diagram needs to show to the reader.  It's probably not necessary to include every detail on the diagram to convey the core information we need to convey.  For example, if there are 20 different user roles, we probably don't need to show them all in the diagram, we can condense them down to particularly relevent ones.
* It's best to limit the diagram to around 20 boxes to remain legible.
* Check the diagram prints correctly on a single piece of paper.  Don't send out diagrams that can't be printed.
* Using different coloured or styled lines is probably too confusing, use the simplest syntax possible.
* Always include a key for the shapes on the diagram.
* It's a good idea to include a title.

# Activity Diagram with Swimlanes

If your software is part of a process that involves some manual steps and some automated ones, then the Activity Diagram with Swimlanes is a great way to describe the process.

This diagram is a UML diagram, but we can use the diagram format without obeying UMLs technical syntax (which is only understood by someone who studies UML).

Use horizontal swimlanes, with each lane representing an actor in the system, or a high level component of the system.  Then use a box for each activity performed (performed either by a human or a software system)  You can describe simple branching and loops in the diagram using an approach similar to a flowchart.  I like to use flowchart shapes in Activity Diagrams.

![Activity diagram with swimlanes](http://agilemodeling.com/images/style/activityDiagramProcessExpenses.gif)

I found the following ideas useful when drawing this sort of diagram:

* As shown in the example above you can describe simple branching and loops in the diagram.
* Showing parallel operations gets too messy so try to avoid it.
* Keep the number of steps shown pretty low.  If the diagram is too long, perhaps you can split it into separate processes in separate diagrams.  It's probably better to do that than have a diagram that spans pages.
* Having the start and the stop point lends clarity to the diagram. 

# Sequence Diagram

A UML Sequence Diagram is about the only low-level code diagram I feel is somewhat useful.  It conveys how components or classes call each other.  I may use it if I need to write technical documentation of existing code.  In such documentation I'd probably also write text that goes with the diagram, explaining something about the components shown in it.  (e.g where they live in source control, etc)

In all likelyhood any developer would explore the code and ignore the diagrams, so the best technical documentation is really in the code itself.  (the readme.md and the comments)

![Sequence diagram](http://agilemodeling.com/images/models/sequenceDiagramEnrollInSeminar.jpg)









