# Cuina sense el Mac: fluxos de n8n

Aquests 4 fluxos fan amb n8n les feines que ara depenen del Mac: el Cookidoo, el carro del Mercadona i els avisos del calendari. Després, Claude els fa servir com a eines (MCP), i el procés del dissabte pot funcionar amb el Mac apagat.

**El sistema actual no ha canviat.** La skill i les tasques del Mac continuen igual fins que tot estigui provat i diguis «endavant».

| Fitxer | Què fa |
|---|---|
| `1-cuina-cookidoo.json` | Llegeix i omple «Mi semana», llegeix receptes (amb el filtre de llegum) i col·leccions, i gestiona la llista de la compra. |
| `2-cuina-mercadona.json` | Cerca productes, llegeix el carro i hi posa o en treu productes. **No pot fer ni pagar comandes.** |
| `3-cuina-calendari-icloud.json` | Crea avisos amb alarma als calendaris d'iCloud («Personal», «Família») i no en repeteix cap. Substitueix l'osascript. |
| `4-cuina-eines-claude-mcp.json` | El connector: posa els 3 fluxos a disposició de Claude, protegit amb un token. |

---

## Estat (5-11-2026)

Claude ja ho ha pujat a `jimixp.app.n8n.cloud` amb l'API de n8n:
- **Els 4 fluxos.** Els 3 primers ja estan publicats.
- **El connector (flux 4) està sense publicar**, a l'espera del teu token.
- **4 credencials amb textos de mostra.** No hi ha cap dada teva: les has d'omplir tu.

No s'ha tocat cap dels teus fluxos anteriors. El flux antic «Estudi de preus Super» només comparava preus: no entrava al compte del Mercadona ni afegia res al carro.

## El que has de fer tu

### 1. Omple les credencials
A n8n, ves a **Overview → Credentials**, obre cadascuna i substitueix els textos de mostra.

**`Cookidoo (usuari i contrasenya)`**
- Hi ha un JSON. Canvia `EL-TEU-CORREU-DE-COOKIDOO` i `LA-TEVA-CONTRASENYA` pel teu correu i la teva contrasenya.

**`iCloud (calendari)`**
- **User:** el teu Apple ID.
- **Password:** una **contrasenya d'aplicació**, no la teva contrasenya normal. Es fa a appleid.apple.com → Inici de sessió i seguretat → Contrasenyes d'aplicació.

**`Mercadona (token)`** (cal fer-ho al Mac)
- **Value:** `Bearer ` seguit del token. Hi ha d'haver un espai després de «Bearer».
- **On trobes el token:**
  1. Obre `tienda.mercadona.es` al Chrome, amb la sessió iniciada.
  2. Obre les eines de desenvolupador (Cmd+Opt+I) i ves a **Network**.
  3. Recarrega la pàgina i filtra per `cart`.
  4. Clica la petició `…/api/customers/XXXX/cart/`. A *Request Headers* hi ha `authorization: Bearer eyJ…`.
- **ID de client:** és la part `XXXX` de la mateixa adreça. Enganxa'l al node **«Entrada i config»** del flux «Cuina · Mercadona», on diu `ENGANXA-AQUI-EL-TEU-ID-DE-CLIENT`. Després torna a **publicar** aquest flux.
- El token dura unes 6 setmanes. Quan caduqui, el flux t'ho dirà.

**`MCP per a Claude (token)`**
- Inventa't un token llarg i aleatori, de 40 lletres i números o més, i posa'l aquí.

> A n8n 2.x, si canvies un flux, **torna'l a publicar** perquè el canvi s'apliqui quan el crida Claude.

### 2. Prova'ls d'un en un
Obre cada flux i clica **Execute workflow**. Fes-ho en aquest ordre, de més segur a menys:

1. **Calendari:** només llegeix els avisos dels 7 dies vinents del calendari «Personal».
2. **Cookidoo:** fa login i llegeix «Mi semana». Si va bé, ha de dir «Login i lectura de "Mi semana" correctes».
3. **Mercadona:**
   - ⚠️ **Toca el teu carro real.** Afegeix 1 paquet de sal i el torna a treure.
   - Si va bé, ha de dir «Prova correcta… El carro ha quedat com estava».

Si alguna prova falla, mira'n el missatge d'error a l'últim node vermell i passa-me'l.

Cada flux té un tercer activador, «Publicació (no fa res)». N'hi ha perquè la teva versió de n8n deixi publicar el flux. S'executa l'1 de gener i no fa res.

### 3. Connecta-ho amb Claude
1. Al flux «Cuina · Eines per a Claude (MCP)», **publica'l** i copia la **Production URL** del node «MCP Server Trigger».
2. A claude.ai, ves a **Configuració → Connectors → Afegeix un connector personalitzat**:
   - Enganxa-hi la URL.
   - A les capçaleres, afegeix `Authorization` amb el valor `Bearer ` seguit del mateix token del pas 1.

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
- També he cridat les 3 eines a través del connector MCP.

**Comprovat al teu n8n:**
- Els fluxos es pugen i es publiquen bé.
- El connector rebutja les peticions sense token o amb un token incorrecte (error 403).

**Què falta: provar-ho amb els serveis reals.** Els riscos coneguts són aquests:
- **Cookidoo:** el login passa per Cloudflare. Pot ser que bloquegi els servidors de n8n (error 403).
- **Mercadona:** té un sistema antibots (Akamai) que pot rebutjar els servidors de n8n. A més, el login sense navegador demana reCAPTCHA, i per això cal copiar el token a mà.
- **Totes dues són API internes no oficials**, i poden canviar sense avís.

## Què no fa

- **No paga ni confirma cap comanda.** Només omple el carro.
- **No envia cap missatge**, ni a la Lupe ni a ningú.
- **No llegeix la llista de l'Alexa.** Amazon va tancar l'accés a les llistes l'1-7-2024.
