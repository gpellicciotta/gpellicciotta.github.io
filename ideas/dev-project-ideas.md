# Development Project Ideas

## Ideas List
- [Annotation Glass Pane](#annotation-glass-pane)
- [Parser Building Blocks Library](#parser-building-blocks-library)
- [Counters App](#counters-app)
- [Hiscore App](#hiscore-app)
- [TimeTrack App](#timetrack-app)
- [RailRoad Diagrams](#railroad-diagrams)

Annotation Glass Pane
=====================

### What?
A project that works on desktop and mobile to put up a glass pane in front of whatever is on the screen.
The glass pane can then be used to:
- make annotations (draw arrows, highlight regions, type some text) + delete them again
- take a screenshot

The glass pane should have two modes:
- inactive: it just sits there, showing any annotations already made but all click and other events
            are passed-through to the underlying apps
- active: click and other events are handled to draw            

### Why?
- To make annotations during a video conference as the tools to do that don't seem readily available
- There is software like Snippet or Cloud App that can also do this, but that's always embedded in a larger
  app while I just want to have annotations + screenshot, and that's it.

### Technology Stack?
Preferably a PWA so it works on mobile and on desktop.
For desktop possibly based on Electron.


Parser Building Blocks Library
==============================

### What?
A library offering a set of classes/functions to help building a parser.

### Why?
Because parser generators are great for small projects but start to show limites for large projects.
Building a parser from scratch is really not that hard with a limited set of reusable functions.

While the resulting parser will be harder to read for somebody new (that's the _forte_ of parser generators),
it will be easier to understand for people that have created it.

Moreover, it will be understandable also for new people after a first introduction as it will rely
completely on normal programming concepts: no need to understand _shift/reduce conflicts_, _semantic vs syntactic lookahead_, etc.

### Technology Stack?
Preferably multiple stacks: Java, JavaScript, C#, Python, ...
It should become something 


Counters App
============

### What?
An online app to keep track of daily habits by counting stuff, e.g. number of coffees, number of exercises done, ...

### Why?
Because it should be super easy to keep track of habits, to help keep up with good intentions (# of work-outs/week) 
but also to understand bad habits better (# of cigarettes/day).

### Technology Stack
Progressive web app with cloud-based back-end


Hiscore App
============

### What?
An online app to keep track of game hiscores.

### Why?
As a personal web dev. exercice in which I can involve my kids too.

### Technology Stack
Progressive web app with cloud-based back-end


TimeTrack App
=============

### What?
An online app to keep track of time spent on tasks.

### Why?
Because it should be super easy to keep track of how much time was spent on certain tasks
and all existing timesheet or similar apps seem way too complicated for how I would want to use it.

### Technology Stack
Progressive web app with cloud-based back-end


RailRoad Diagrams
=================

### What?
Make it super easy to turn a Backus-Naur form of syntax into a railroad diagram, that
can then be copy-pasted into documentation, or just used directly as a link.

### Why?
Because it seems fun to do.

There are existing apps that do this already:
  - https://www.bottlecaps.de/rr/ui
  - http://tabatkins.github.io/railroad-diagrams/

### Technology Stack
Pure Javascript web-app