# Firmware Update Lifecycle

A firmware update is a serialized download-and-commit sequence directed through one controller or Management Endpoint. Activation affects a Domain and may occur immediately or at a reset boundary. [PDF pp. 152-154](../_source/pages/page-152.md)

## Mental model

```text
[download contiguous portions] -> [commit + validate] -> activation choice
                                                      |-- immediate
                                                      `-- controller / subsystem /
                                                          conventional reset
                                                                   |
                                                                   v
                                                        [reinitialize queues]
```

This explanatory lifecycle combines the reset and no-reset procedures; the required reset is reported by command status when immediate activation cannot complete. [PDF pp. 152-153](../_source/pages/page-152.md)

## Download and commit contract

Firmware Image Download may send multiple portions with explicit offsets. The completed image should follow Firmware Update Granularity and is invalid if it does not begin at offset zero, has gaps, or has overlapping regions; Firmware Commit may apply additional vendor validation before committing it to a slot. [PDF p. 152](../_source/pages/page-152.md)

The host should keep every command in one firmware/boot-partition update sequence on the same controller or Management Endpoint and must not overlap update sequences. A new download after a completed commit discards remaining old portions, and a reset before commit completes discards all downloaded portions. [PDF p. 154](../_source/pages/page-154.md)

Each Firmware Image Download carries a 0's-based dword count and dword offset. Firmware pieces may arrive out of order, except Boot Partition pieces must be ordered; every sequence starts with an offset-zero piece and should satisfy `FWUG` alignment/granularity without overlapping ranges. Download does not activate the image. [PDF pp. 216-217](../_source/pages/page-216.md)

## Activation paths

| Path | Host action | Result |
|---|---|---|
| Reset activation | Commit a slot, then perform the required Controller Level, NVM Subsystem, or Conventional Reset | firmware activates at that boundary; host reinitializes the controller and reallocates I/O queues |
| Immediate activation | Commit Action `011b` | affected controllers may signal Firmware Activation Starting; the commit completes or reports the reset/time condition that prevented activation |

The reset path and its controller reinitialization are specified on page 152; immediate activation, asynchronous notification, and failure statuses continue on page 153. [PDF pp. 152-153](../_source/pages/page-152.md)

| Commit Action | Command-level effect |
|---|---|
| `000b` | place downloaded image in a slot without activation |
| `001b` | place image and activate at next Controller Level Reset |
| `010b` | activate an existing slot at next Controller Level Reset |
| `011b` | place if downloaded, then activate immediately; command stays outstanding through activation |
| `110b` | replace selected Boot Partition |
| `111b` | mark selected Boot Partition active |

This compact reconstruction follows rendered Figure 181. Slot `0h` lets the controller choose firmware slot 1 through 7 where a slot is applicable. Firmware slots and their image are shared by all controllers in the Domain. [PDF pp. 214-215](../_source/pages/page-214.md)

If activation exceeds `MTFA`, the image remains committed and the host may request activation at Controller Level Reset with Commit Action `010b`. If loading fails, the controller reverts to the most recently activated slot or available baseline read-only image and reports Firmware Image Load Error; changing firmware during sanitize causes sanitize failure. [PDF p. 153](../_source/pages/page-153.md)

All controllers in a Domain share firmware slots and receive the same firmware image. A successful reset-activation Commit is activated by any Controller Level Reset method; immediate activation is a foreground Commit that remains outstanding until activation succeeds or fails. [PDF pp. 214-215](../_source/pages/page-214.md)

## Firmware-slot observation

```text
slot revisions: [1] [2] [3] [4] [5] [6] [7]
                  ^ current          ^ next-reset activation (optional)
```

The Firmware Slot Information log reports the currently active slot, an optional slot scheduled for the next Controller Level Reset, and an eight-byte ASCII revision string for each of seven slots. An unsupported or empty slot has an all-zero revision field. [PDF p. 235](../_source/pages/page-235.md)

## Edge cases

Overwriting the active slot can remove the previously active image and create a pending activation on a later reset. While such activation is pending, subsequent Sanitize commands are aborted with the status for the required reset. A transition to D3 cold while an activating Firmware Commit is outstanding may resume with either the old or newly activated image. [PDF p. 153](../_source/pages/page-153.md)

Immediate activation that could introduce host-incompatible behavior must instead return Firmware Activation Requires Conventional Reset. Overlapping update sequences detected through either a Management Endpoint or an Admin Submission Queue are reported in CQE Dword 0 even when the Firmware Commit is aborted. [PDF pp. 214-215](../_source/pages/page-214.md)

Commit status distinguishes invalid slot/image, required Conventional/NVM Subsystem/Controller Level Reset, MTFA violation, prohibited activation, overlapping range, and Boot Partition write lock. A more-specific required reset leaves the current image running if a narrower reset occurs first. [PDF p. 216](../_source/pages/page-216.md)

## Relationships

- Firmware activation scope is a Domain, while the command sequence is submitted through one controller or Management Endpoint. [PDF pp. 152-154](../_source/pages/page-152.md)
- Reset activation reconnects this lifecycle to controller initialization and queue allocation. [PDF p. 152](../_source/pages/page-152.md)
- Firmware activation and sanitize interact: a revert fails an active sanitize, and a pending reset activation blocks a new Sanitize command. [PDF p. 153](../_source/pages/page-153.md)

## Evidence

- [Reset-based update procedure, PDF p. 152](../_source/pages/page-152.md)
- [Immediate activation and failure handling, PDF p. 153](../_source/pages/page-153.md)
- [Sequence serialization and discard rules, PDF p. 154](../_source/pages/page-154.md)
- [Rendered Figure 181, Firmware Commit action and slot fields, PDF pp. 214-215](../_source/pages/page-214.md)
- [Rendered Figure 182, overlapping-update completion flags, PDF p. 215](../_source/pages/page-215.md)
- [Rendered Figure 183, Firmware Commit status values, PDF p. 216](../_source/pages/page-216.md)
- [Rendered Figures 184-187, Firmware Image Download fields and status, PDF pp. 217](../_source/pages/page-217.md)
- [Rendered Figure 208, Firmware Slot Information log, PDF p. 235](../_source/pages/page-235.md)
