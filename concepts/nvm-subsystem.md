# NVM Subsystem

An NVM subsystem is the system boundary that contains controllers, ports, namespaces, and the storage resources those namespaces expose. Hosts interact with its storage through controllers rather than directly with underlying media. [PDF pp. 46-52](../_source/pages/page-046.md)

## Mental model

```text
                 +----------- NVM Subsystem -----------+
Host A <-------->| Controller A                        |
Host B <-------->| Controller B ----+                  |
                 |                  +--> Namespaces    |
                 |                       |             |
                 |                 storage resources   |
                 +-------------------------------------+
```

This is explanatory; controller/namespace visibility and sharing depend on subsystem configuration. [PDF pp. 52-57](../_source/pages/page-052.md)

## Relationships

- A controller is the interface between a host and the subsystem.
- A subsystem may contain multiple controllers and namespaces.
- Domains, Endurance Groups, NVM Sets, and Reclaim Groups organize underlying resources.

## Evidence

- [Storage entities, PDF pp. 46-52](../_source/pages/page-046.md)
- [Controller architecture, PDF p. 58](../_source/pages/page-058.md)
