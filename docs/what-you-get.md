# What you get

## 26 tools

| Area | Tools |
| --- | --- |
| **Operators** | create, wire, move, delete, annotate |
| **Parameters** | read, set, pulse, add custom pars, look up names |
| **Content and inspection** | DAT text, CHOP channels, TOP pixels, geometry, screenshots, Python |
| **Query and navigation** | browse, search, connections, TD documentation |
| **Project** | info, save, errors, cook performance |

Every tool returns structured errors, so the agent can tell an invalid menu value
from a missing operator and correct itself rather than guessing again.

## Documentation, from your build

Two tools keep the agent on your installed version instead of its training data.

**`get_help`** returns parameter names and types for any operator type from the
running build, and menu values from a catalog extracted per build.

**`get_docs`** serves the Python reference and operator help articles from the
offline documentation installed with TouchDesigner, or from the wiki — whichever
you select on the [Docs page](parameters.md#docs-documentation-source).

## Undo on edits

Changes made through the editing tools are wrapped in undo blocks, so Ctrl+Z
reverses them.

!!! note "One exception"

    Creating an annotation tears down the open undo block inside TouchDesigner,
    so annotation changes are not reliably undoable.

One tool call is one undo step. An agent that creates ten operators in ten calls
gives you ten separate undos.

## Visual feedback

The network editor pans to whatever the agent is touching and flashes it: amber
for edits, blue for inspections, violet for views. Configurable on the
[Tune page](parameters.md#tune-rate-limiting-and-visual-feedback).

## Skills

A library of TouchDesigner conventions the agent loads on demand — builder guides
per operator family, layout and performance conventions, and workflow skills for
planning, reviewing and cleanup. See [Skills](skills.md).
