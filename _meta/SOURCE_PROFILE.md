# Source profile

- Title: NVM Express Base Specification
- Revision: 2.1, ratified August 5, 2024
- Length: 707 PDF pages; normative page 1 begins at PDF page 23
- Structure: unencrypted native text, 892 bookmarks
- Visual inventory: 762 uniquely numbered Figures
- Normative density: more than 2,100 occurrences of `shall`

## Why extraction must be concept-led

This document is simultaneously an architecture description, register contract, command protocol, data-structure catalog, and collection of state machines. Many numbered Figures are actually bit-field tables or command/data layouts.

The wiki therefore separates:

1. architecture concepts and ownership boundaries;
2. transport, queue, and command lifecycles;
3. commands, features, log pages, and status values;
4. registers, properties, structures, and bit fields;
5. state machines, error recovery, power, sanitize, and security behavior.

Definitions are not promoted blindly into standalone pages. Low-level fields remain nested under the concept or protocol operation that gives them meaning.
