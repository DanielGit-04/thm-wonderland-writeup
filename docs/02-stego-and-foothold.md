# Fase 2 — Esteganografía y acceso inicial (alice)

[← Volver al README](../README.md) · [← Fase anterior](./01-recon-and-web.md)

## Esteganografía

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

## Siguiendo al conejo

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

## Acceso por SSH

```bash
ssh alice@<TARGET_IP>
# Password: HowDothTheLittleCrocodileImproveHisShiningTail
```

Dentro del home de `alice` aparece `root.txt`, pero pertenece a `root` y no es legible — la sala juega con la idea de que "todo está del revés". La flag de usuario (`user.txt`) se encuentra en realidad en `/root`:

```bash
cat /root/user.txt
thm{FLAG_CENSURADA}
```

➡️ Siguiente: [Fase 3 — alice → rabbit (Python library hijacking)](./03-privesc-alice-to-rabbit.md)
