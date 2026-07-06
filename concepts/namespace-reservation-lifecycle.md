# Namespace Reservation Lifecycle

Namespace reservations coordinate registrants and one reservation holder through host-owned 64-bit keys, typed access policies, optional preemption, and queryable generation state. [PDF pp. 489-495](../_source/pages/page-489.md)

## Mental model

```text
[unregistered]
     | Register key
     v
[registrant] -- Acquire(type) --> [reservation holder]
     |                |                 |
     | Replace key    | Preempt         | Release
     | Unregister     | Preempt+Abort   v
     v                v            [registrant]
 [unregistered]   competing key removed
                         |
                     Clear -> reservation + registrations cleared
```

This explanatory state machine groups Register, Acquire, Preempt, Release, and Clear while preserving that keys belong to hosts and reservations belong to namespaces. [PDF pp. 489-493](../_source/pages/page-489.md)

## Registration and persistence

Reservation Register registers, unregisters, or replaces the submitting host's key. `IEKEY` permits applicable actions to skip validation of the current key. The same command may leave PTPL unchanged, clear it, or set it; when the Reservation Persistence Feature is saveable, this side effect updates both current and saved Feature values. [PDF pp. 490-492](../_source/pages/page-490.md)

```text
PTPL=0 -- power loss --> reservation released + registrants cleared
PTPL=1 -- power loss --> reservation and registrants persist
```

This explanatory persistence split is reconstructed from the Register command and Reservation Status fields. [PDF pp. 491, 494-495](../_source/pages/page-491.md)

## Acquire, preempt, and release

Acquire selects one of six reservation types: write-exclusive or exclusive-access, each scoped to the holder, registrants only, or all registrants as defined by its type. It validates the current reservation key. Preempt additionally names the key to unregister; Preempt and Abort also requests abort of outstanding commands for the namespace. `IEKEY=1` is invalid for Acquire. [PDF pp. 489-490](../_source/pages/page-489.md)

Release removes the current reservation while retaining registrations and requires the supplied type to match. Clear removes the reservation and registration state. Both validate the current key, and `IEKEY=1` is invalid. [PDF pp. 492-493](../_source/pages/page-492.md)

## Dispersed-namespace handshake

When the host has enabled Host Dispersed Namespace Support, each reservation command uses `DISNSRS` to affirm that it can interpret cross-participant controller sentinel `FFFDh`. Clearing that bit on a dispersed namespace is rejected as Namespace Is Dispersed. [PDF pp. 490-494](../_source/pages/page-490.md)

## Reservation Report

Reservation Report returns a prefix-length-controlled snapshot with a wrapping generation counter, reservation type, registrant count, PTPL state, and one record per registrant. The host-selected 64-bit or 128-bit Host Identifier width must match the basic or extended report format. [PDF pp. 493-495](../_source/pages/page-493.md)

Generation increments after successful registration changes, Clear, and Preempt/Preempt-and-Abort. In registrant records, `CNTLID=FFFFh` means no controller association in this subsystem and `FFFDh` means the controller lies in another participating subsystem of a dispersed namespace. [PDF pp. 494-495](../_source/pages/page-494.md)

Basic records are 24 bytes and carry a 64-bit Host Identifier; extended records are 64 bytes and carry the 128-bit Host Identifier. Both identify controller association, whether that registrant's host holds the reservation, and the registrant's reservation key. [PDF pp. 495-496](../_source/pages/page-495.md)

## Relationships

- [Reservation Notification Log](reservation-notification-log.md) reports unmasked registration-preempted, reservation-released, and reservation-preempted events after lifecycle changes. [PDF pp. 301-302, 424-425](../_source/pages/page-301.md)
- [Identity, Name, and List Formats](identity-name-and-list-formats.md) defines the Host Identifier width that selects the report record form. [PDF pp. 422-424, 493-495](../_source/pages/page-422.md)
- [Domains and Divisions](domains-and-divisions.md) describes dispersed-namespace recovery and cross-participant controller identity. [PDF pp. 104-107, 490-495](../_source/pages/page-104.md)

## Evidence

- [Acquire actions, keys, dispersed support, and reservation types, PDF pp. 489-490](../_source/pages/page-489.md)
- [Register actions and PTPL side effects, PDF pp. 490-492](../_source/pages/page-490.md)
- [Release/Clear actions and report format selection, PDF pp. 492-494](../_source/pages/page-492.md)
- [Reservation Status header and registrant records, PDF pp. 494-495](../_source/pages/page-494.md)
- [Basic and extended registrant record completion, PDF pp. 495-496](../_source/pages/page-495.md)
