# Fase 1 — Reconocimiento y enumeración web

[← Volver al README](../README.md)

## Escaneo de puertos

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

## Enumeración web

Al visitar `http://<TARGET_IP>/` aparece una página temática de *Alicia en el País de las Maravillas*, con una imagen de un conejo blanco y el texto "Follow the White Rabbit". No hay formularios ni enlaces visibles, así que toca enumerar directorios:

```bash
gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirb/common.txt -t 50
```

**Resultado relevante:**

```
/img         (Status: 301)
/r           (Status: 301)
/index.html  (Status: 301)
```

El directorio `/r` llama la atención: sugiere que hay que "seguir" algo letra por letra (la pista de esteganografía, en la siguiente fase, lo confirma).

➡️ Siguiente: [Fase 2 — Esteganografía y acceso a alice](./02-stego-and-foothold.md)
