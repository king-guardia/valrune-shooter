# UML

Mermaid diagrams, which render in both Cursor and GitHub. Miro only if a diagram
outgrows text.

Planned (Phase 3): entity model · ability model · status model with immunity resolution ·
damage-flow sequence for one gun-shot · baddie action state machine · drone behavior ·
deferred-damage queue.

Design constraint: composition over inheritance, maximum two levels. `Baddie` is one
class with a role-tag component, not five subclasses.
