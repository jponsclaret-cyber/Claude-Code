# Cuina sense el Mac: fluxos de n8n

Aquests 4 fluxos fan amb n8n les feines que ara depenen del Mac: el Cookidoo, el carro del Mercadona i els avisos del calendari. Després, Claude els fa servir com a eines (MCP), i el procés del dissabte pot funcionar amb el Mac apagat.

**Encara no canvien res del sistema actual.** La skill i les tasques del Mac continuen igual fins que tot estigui provat i diguis «endavant».

| Fitxer | Què fa |
|---|---|
| `1-cuina-cookidoo.json` | Llegeix i omple «Mi semana», llegeix receptes (amb el filtre de llegum) i col·leccions, i gestiona la llista de la compra. |
| `2-cuina-mercadona.json` | Cerca productes, llegeix el carro i hi posa o en treu productes. **No pot fer ni pagar comandes.** |
| `3-cuina-calendari-icloud.json` | Crea avisos amb alarma als calendaris d'iCloud («Personal», «Família») i no en repeteix cap. Substitueix l'osascript. |
| `4-cuina-eines-claude-mcp.json` | El connector: posa els 3 fluxos a disposició de Claude. |

No he tocat cap dels fluxos que ja tens a n8n.

---

## Pas 1. Crea les 3 credencials (abans d'importar)

A n8n: **Overview → Create → Credential**. Posa-hi **exactament aquests noms**, perquè els fluxos les trobin sols quan els importis.

### a) `Cookidoo (usuari i contrasenya)`
- **Tipus:** *Custom Auth*.
- **JSON:** `{"body": {"username": "el-teu-correu", "password": "la-teva-contrasenya"}}`
- **Recomanat:** a *Allowed HTTP Request Domains*, posa-hi només `ciam.prod.cookidoo.vorwerk-digital.com`.

### b) `Mercadona (token)`
- **Tipus:** *Header Auth*.
- **Name:** `Authorization`
- **Value:** `Bearer ` seguit del token. Hi ha d'haver un espai després de «Bearer».
- **Recomanat:** a *Allowed HTTP Request Domains*, posa-hi `tienda.mercadona.es`.

**On trobes el token (i l'ID de client, que et caldrà al pas 2):**
1. Obre `tienda.mercadona.es` al Chrome, amb la sessió iniciada.
2. Obre les eines de desenvolupador (Cmd+Opt+I) i ves a la pestanya **Network**.
3. Recarrega la pàgina i escriu `cart` al filtre.
4. Clica la petició `…/api/customers/XXXXXXXX-…/cart/`.
   - **ID de client:** és la part `XXXXXXXX-…` de l'adreça.
   - **Token:** a *Request Headers*, la capçalera `authorization: Bearer eyJ…`. Copia'n el valor sencer.

El token dura unes **6 setmanes**. Quan caduqui, el flux t'ho dirà clarament i només caldrà repetir aquests passos.

### c) `iCloud (calendari)`
- **Tipus:** *Basic Auth*.
- **User:** el teu Apple ID.
- **Password:** una **contrasenya d'aplicació**, no la teva contrasenya normal.
  - Es fa a appleid.apple.com → Inici de sessió i seguretat → Contrasenyes d'aplicació.
  - Posa-li un nom, per exemple «n8n cuina».

## Pas 2. Importa els fluxos

Per a cada fitxer, en aquest ordre (1, 2, 3 i 4): **Create → Workflow → ⋯ → Import from File**, i desa'l.

- **Mercadona:** obre el node **«Entrada i config»** i enganxa l'ID de client on diu `ENGANXA-AQUI-EL-TEU-ID-DE-CLIENT`.
- **Connector (flux 4):** obre cadascuna de les 3 eines (`cookidoo`, `mercadona` i `calendari_icloud`) i, a **Workflow**, tria el flux que li correspon.
- **Publica (Publish) els 4 fluxos.** Si els fluxos 1, 2 i 3 no estan publicats, Claude no els pot fer servir. Ho he comprovat.
- Si després canvies alguna cosa dels fluxos 1, 2 o 3, torna'ls a publicar.

## Pas 3. Prova'ls d'un en un

Obre cada flux i clica **Execute workflow**. Fes-ho en aquest ordre, de més segur a menys:

1. **Calendari:** només llegeix els avisos dels 7 dies vinents del calendari «Personal».
2. **Cookidoo:** fa login i llegeix «Mi semana». Si va bé, ha de dir «Login i lectura de "Mi semana" correctes».
3. **Mercadona:**
   - ⚠️ **Toca el teu carro real.** Afegeix 1 paquet de sal i el torna a treure.
   - Si va bé, ha de dir «Prova correcta… El carro ha quedat com estava».
   - Si el carro ja té coses, també les deixa igual.

Si alguna prova falla, mira'n el missatge d'error a l'últim node vermell i passa-me'l.

## Pas 4. Connecta-ho amb Claude

1. Al flux 4, obre el node **«MCP Server Trigger»** i copia la **Production URL**.
2. A claude.ai, ves a **Configuració → Connectors → Afegeix un connector personalitzat** i enganxa-hi la URL.
3. Et demanarà que entris amb el teu compte de n8n (OAuth). Així no cal compartir cap clau.

---

## Com l'he provat i què falta provar

**Com l'he provat:**
- He fet servir n8n 2.46 i un servidor fals que imita el Cookidoo, el Mercadona i l'iCloud.
- He fet més de 30 proves:
  - el login i el token;
  - «Mi semana», les receptes amb llegum i les col·leccions;
  - la llista de la compra;
  - la cerca i el carro;
  - la prova d'afegir i treure;
  - els avisos amb accents, emojis i alarma, i que no es repeteixin;
  - els errors de token caducat i de contrasenya incorrecta.
- També he cridat les 3 eines a través del connector MCP, igual que ho farà Claude.

**Què falta: provar-ho amb els serveis reals.** Els riscos coneguts són aquests:
- **Cookidoo:** el login passa per Cloudflare. Pot ser que bloquegi els servidors de n8n (error 403).
- **Mercadona:** té un sistema antibots (Akamai) que pot rebutjar els servidors de n8n. A més, el token s'ha de copiar a mà cada 6 setmanes, perquè el login sense navegador demana reCAPTCHA.
- **Totes dues són API internes no oficials**, i poden canviar sense avís.

## Què no fa

- **No paga ni confirma cap comanda.** Només omple el carro.
- **No envia cap missatge**, ni a la Lupe ni a ningú.
- **No llegeix la llista de l'Alexa.** Amazon va tancar l'accés a les llistes l'1-7-2024. Les alternatives no oficials necessiten guardar una sessió d'Amazon de llarga durada, i no ho recomano.
- **El flux de la nit (estoc de Notion)** no necessita n8n. Es pot passar a una tasca de Claude al núvol, que ja té accés a Notion.
