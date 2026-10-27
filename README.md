# TryHackMe – Wonderland (Writeup)

![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange)
![OS](https://img.shields.io/badge/OS-Linux-blue)
![Platform](https://img.shields.io/badge/Platform-TryHackMe-red)

Writeup personal de la máquina **[Wonderland](https://tryhackme.com/room/wonderland)** de TryHackMe, una sala de dificultad media ambientada en *Alicia en el País de las Maravillas*. Cubre enumeración web, esteganografía, *Python library hijacking*, *PATH hijacking* sobre un binario SUID y abuso de *Linux capabilities* para escalar a root.

> ℹ️ Las flags se muestran censuradas en este repo. Despliega la máquina en TryHackMe y síguela paso a paso para obtenerlas tú mismo.

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

## Writeup detallado

El desarrollo paso a paso está dividido por fases en [`docs/`](./docs):

1. [Reconocimiento y enumeración web](./docs/01-recon-and-web.md)
2. [Esteganografía y acceso inicial (alice)](./docs/02-stego-and-foothold.md)
3. [Escalada: alice → rabbit](./docs/03-privesc-alice-to-rabbit.md)
4. [Escalada: rabbit → hatter](./docs/04-privesc-rabbit-to-hatter.md)
5. [Escalada: hatter → root](./docs/05-privesc-hatter-to-root.md)

## Lecciones aprendidas

1. **El código fuente HTML siempre es accesible**, aunque el CSS oculte visualmente un elemento (`display: none`).
2. **Esteganografía** es una técnica habitual en CTFs; `steghide` y `binwalk` deben ser parte rutinaria del análisis de imágenes.
3. **El orden de búsqueda de módulos en Python es peligroso**: si un script privilegiado importa módulos sin ruta absoluta, se puede secuestrar el import colocando un archivo con el mismo nombre en el mismo directorio.
4. **Binarios SUID que invocan comandos sin ruta absoluta** son vulnerables a *PATH hijacking*. Mitigación: usar siempre rutas absolutas (`/bin/date`, no `date`).
5. **Las Linux capabilities se pasan por alto a menudo**: `cap_setuid` en un intérprete como `perl` o `python` equivale, en la práctica, a darle privilegios de root. Ejecutar `getcap -r / 2>/dev/null` en toda enumeración Linux.

## Herramientas utilizadas

| Herramienta | Uso |
|---|---|
| `nmap` / `rustscan` | Descubrimiento de puertos y servicios |
| `gobuster` | Fuerza bruta de directorios web |
| `steghide` | Extracción de datos ocultos en imágenes |
| `strings` | Análisis estático de binarios |
| `sudo -l` | Enumeración de privilegios sudo |
| `getcap` | Detección de Linux capabilities |

## Referencias

- [TryHackMe – Wonderland](https://tryhackme.com/room/wonderland)
- [GTFOBins – Perl](https://gtfobins.github.io/gtfobins/perl/)
- [HackTricks – Linux Capabilities](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities)

---

*Writeup realizado con fines educativos tras completar la sala en mi propia instancia de TryHackMe.*
