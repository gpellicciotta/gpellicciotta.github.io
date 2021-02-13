# Development Project Ideas

## Annotation Glas Pane

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