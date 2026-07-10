# Visual language

Use an ASCII visualization when it materially clarifies hierarchy, sequence, mapping, ownership, or state. Keep it readable in plain Markdown.

## Preferred forms

### Hierarchy

```text
NVM Subsystem
`-- Domain
    `-- Endurance Group
        |-- NVM Set
        |   `-- Namespace
        `-- Reclaim Group
            `-- Reclaim Unit
```

### Sequence

```text
Host             Submission Queue       Controller       Completion Queue
 |  write SQE ---------->|                   |                    |
 |  ring doorbell ------>|------------------>|                    |
 |                       |      execute      |                    |
 |                       |                   |---- write CQE ---->|
 |<---------------------- interrupt / poll -----------------------|
```

### State machine

```text
[Disabled] -- CC.EN=1 --> [Enabling] -- CSTS.RDY=1 --> [Ready]
     ^                                                   |
     `---------------------- CC.EN=0 --------------------'
```

### Bit field

```text
 31                16 15                 0
+--------------------+--------------------+
|     upper field    |     lower field    |
+--------------------+--------------------+
```

## Rules

- Label whether a diagram is explanatory or reconstructed from a numbered Figure.
- Cite the source pages directly below every diagram.
- Preserve direction, multiplicity, optionality, and ownership; do not simplify these away.
- Prefer a Markdown table for exact values and an ASCII diagram for relationships.
- Do not redraw decorative figures or make a diagram for a self-contained one-line definition.
