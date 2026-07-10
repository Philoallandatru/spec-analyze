# Controller Ready Modes and Timeouts

Controller readiness may mean either that command processing and media are both usable, or that non-media command processing is usable while namespaces and media continue initialization. `CAP.CRMS` advertises the modes, `CC.CRIME` selects between them when both are supported, and `CAP.TO` plus `CRTO` bound the waits. [PDF pp. 128-131](../_source/pages/page-128.md)

## Mental model

```text
CC.EN: 0 -> 1
      |
      +-- Ready With Media (CC.CRIME=0)
      |      `-- by CRTO.CRWMT: RDY=1 + commands/media ready
      |
      `-- Ready Independent of Media (CC.CRIME=1)
             |-- by CRTO.CRIMT: RDY=1 + non-media commands ready
             `-- by CRTO.CRWMT: attached namespaces/media ready
```

This explanatory timeline reconstructs the two readiness contracts and their deadlines. [PDF pp. 128-131](../_source/pages/page-128.md)

## Mode selection

| `CAP.CRMS` | Supported behavior | `CC.CRIME` |
|---|---|---|
| `00b` | Legacy/non-selectable; readiness bounded by `CAP.TO` | read-only `0`, host ignores |
| `01b` | Ready With Media only | read-only `0` |
| `11b` | Both modes | host selects before enable |

Revision 2.1 controllers set Ready With Media support; `00b` remains meaningful for controllers compliant with specifications before Revision 2.0. Changing `CC.CRIME` does not change `CRTO.CRWMT` or `CRTO.CRIMT`, though it may affect `CAP.TO`. [PDF pp. 130-131](../_source/pages/page-130.md)

## Command availability contract

In Ready With Media mode, `CSTS.RDY=1` means all attached namespaces, media needed by Admin commands, and command processing are ready. In Ready Independent of Media mode, commands not needing attached namespaces or media must work by `CRTO.CRIMT`; media-dependent Admin and I/O commands may return retryable Namespace Not Ready or Admin Command Media Not Ready until `CRTO.CRWMT`. [PDF pp. 128-131](../_source/pages/page-128.md)

Figure 84 is an exact cross-page allowlist of Admin commands that may report Admin Command Media Not Ready, with narrower restrictions for Device Self-test and Get Log Page. The PDF table remains authoritative rather than duplicating the full list here. [PDF pp. 129-130](../_source/pages/page-129.md)

## Timeout failure

If a non-zero `CAP.CRMS` controller misses the applicable processing or media deadline, it must nevertheless set `CSTS.RDY=1` no later than `CRTO.CRWMT`. If the Persistent Event Log is supported, it also records an NVM Subsystem Hardware Error event whose code is Controller Ready Timeout Exceeded. [PDF pp. 131-132](../_source/pages/page-131.md)

## Evidence

- [Discovery initialization and ready-mode definition, PDF p. 128](../_source/pages/page-128.md)
- [Ready-independent command restrictions and Figure 84 start, PDF p. 129](../_source/pages/page-129.md)
- [Figure 84 continuation, mode selection, and timeout definitions, PDF p. 130](../_source/pages/page-130.md)
- [Timeout mapping and failure handling, PDF pp. 131-132](../_source/pages/page-131.md)
