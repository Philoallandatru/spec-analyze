# Reservation Notification Log

The Reservation Notification log is a controller-local, destructive-read queue of unmasked reservation changes for namespaces attached to that controller. Each successful read returns and removes the oldest queued notification. [PDF pp. 301-302](../_source/pages/page-301.md)

## Mental model

```text
unmasked reservation changes
          |
          v
   +------+------+------+       Get Log Page LID 80h
   | LPC n| LPC n+1| ... |  ---------------------------->
   +------+------+------+       return + remove oldest
          ^
          `-- overflowed changes advance newest queued LPC
```

This explanatory queue diagram summarizes the read/removal and loss-accounting rules; it is not a reconstruction of Figure 290. [PDF pp. 301-302](../_source/pages/page-301.md)

## Contract and invariants

| Field or condition | Meaning |
|---|---|
| `LPC` | 64-bit notification sequence, reset to zero by Controller Level Reset; wraps from all-ones to `1h`, reserving zero for an empty result |
| `RNLPT` | registration preempted, reservation released, or reservation preempted |
| `NALP` | additional unread pages after this one, saturated at 255 |
| `NSID` | namespace associated with the notification |
| Empty queue | all-zero 64-byte log page |

[PDF p. 302](../_source/pages/page-302.md)

Every event that causes a reservation notification advances the sequence even when the controller cannot enqueue it. On overflow, the newest queued page's count is advanced through the lost notifications, so a reader can detect a gap without receiving the discarded details. [PDF pp. 301-302](../_source/pages/page-301.md)

## Relationships

- The log is populated only for unmasked reservation notifications on namespaces attached to the controller. [PDF p. 301](../_source/pages/page-301.md)
- Reservation Notification Mask (`82h`) independently masks registration-preempted, reservation-released, and reservation-preempted records per namespace. Broadcast Set updates every attached reservation-capable namespace, but broadcast Get is invalid. [PDF p. 424](../_source/pages/page-424.md)
- Reservation Persistence (`83h`) controls the namespace PTPL state: when set, reservations and registrants survive power loss; when clear, both are discarded. It is non-saveable, supports broadcast Set, and rejects broadcast Get. [PDF pp. 424-425](../_source/pages/page-424.md)
- Unlike retained snapshots, reading this LID consumes one record; `NALP` tells the host whether to continue draining. [PDF pp. 301-302](../_source/pages/page-301.md)

## Evidence

Reservation Notification Mask (`82h`) independently suppresses registration-preempted, reservation-released, and reservation-preempted entries per namespace. Broadcast Set applies to every attached reservation-capable namespace; broadcast Get is invalid. Masking prevents creation of the entry rather than filtering it afterward. [PDF p. 424](../_source/pages/page-424.md)

Reservation Persistence (`83h`) controls each namespace's Persist Through Power Loss state. Broadcast Set updates attached reservation-capable namespaces; broadcast Get is invalid. `PTPL=1` preserves reservations and registrants across power loss, while zero releases/clears them. [PDF pp. 424-425](../_source/pages/page-424.md)

- [Queue creation, destructive read, empty behavior, and overflow accounting, PDF p. 301](../_source/pages/page-301.md)
- [Rendered Figure 290 field layout and counter rollover, PDF p. 302](../_source/pages/page-302.md)
