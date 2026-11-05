# Cuina sense el Mac: fluxos de n8n

Aquests fluxos fan amb n8n les feines que ara depenen del Mac: el Cookidoo, el carro del Mercadona i els avisos del calendari. Claude els fa servir com a eines (MCP), i el procés del dissabte pot funcionar amb el Mac apagat.

**El sistema actual no ha canviat.** La skill i les tasques del Mac continuen igual fins que tot estigui provat i diguis «endavant».

| Fitxer | Què fa |
|---|---|
| `1-cuina-cookidoo.json` | Llegeix i omple «Mi semana», llegeix receptes (amb el filtre de llegum) i col·leccions, i gestiona la llista de la compra. |
| `2-cuina-mercadona.json` | Cerca productes, llegeix el carro i hi posa o en treu productes. **No pot fer ni pagar comandes.** |
| `3-cuina-calendari-google.json` | Crea avisos amb notificació a l'hora exacta al Google Calendar i no en repeteix cap. Substitueix l'osascript i l'iCloud. |
| `4-cuina-eines-claude-mcp.json` | El connector: posa els 3 fluxos a disposició de Claude, protegit amb un token. |

---

## Estat (5-11-2026)

**Tot és pujat a `jimixp.app.n8n.cloud`, i els 3 fluxos de servei estan publicats i provats amb les teves comptes reals:**

| Servei | Prova real |
|---|---|
| Cookidoo | ✅ Fa login des de n8n i llegeix «Mi semana» |
| Mercadona | ✅ Afegeix 1 sal al carro, la treu i el carro queda igual |
| Google Calendar | ✅ Llegeix el calendari principal. Encara no s'ha creat cap avís real |

**iCloud:** Apple rebutjava la contrasenya d'aplicació (error 401, també fent la prova des del Terminal). Per això s'ha canviat pel Google Calendar. El flux «Cuina · Calendari iCloud» queda sense publicar: el pots esborrar, i també la credencial «iCloud (calendari)» i l'antiga «Mercadona (token)».

**Pendent:** afegir el connector a claude.ai (pas 3). El connector ja està publicat: rebutja les peticions sense token (403) i accepta les que porten el token (200).

## Credencials (n8n → Credentials)

- **`Cookidoo (usuari i contrasenya)`**, tipus *Custom Auth*. JSON: `{"body": {"username": "…", "password": "…"}}`.
- **`Mercadona token (només eyJ…)`**, tipus *Bearer Auth*.
  - Hi va **només** el token, que comença per `eyJ`. n8n ja hi afegeix «Bearer».
  - Per copiar-lo sencer: a la consola del Chrome, amb `tienda.mercadona.es` oberta, escriu `copy(JSON.parse(localStorage.getItem('MO-user')).token)`.
  - Dura unes 6 setmanes.
- **`Google Calendar (cuina)`**, tipus *Google Calendar OAuth2 API*: es connecta amb «Sign in with Google».
- **`MCP per a Claude (x-auth-token)`**, tipus *Header Auth*: Name `x-auth-token`, Value un token llarg i aleatori.

Perquè els avisos de Google sonin a l'iPhone, ves a Configuració → Calendari → Comptes i afegeix-hi el compte de Google, o fes servir l'app Google Calendar.

> A n8n, si canvies un flux, **torna'l a publicar** perquè el canvi s'apliqui quan el crida Claude.
>
> **Per provar un flux a mà**, executa'l des del node **«Prova manual»**, no des de «Publicació (no fa res)». Aquest segon activador només serveix perquè n8n deixi publicar el flux.

## 3. Connecta-ho amb Claude
El connector ja està publicat i protegit amb el token de la credencial «MCP per a Claude (x-auth-token)».

A claude.ai, ves a **Configuració → Connectors → Afegeix un connector personalitzat**:
- Tria **«Sin inicio de sesión»**.
- Enganxa-hi la URL `https://jimixp.app.n8n.cloud/mcp/cuina-ccda21bc1c04`.
- A les capçaleres, afegeix `x-auth-token` amb el token com a valor (sense «Bearer»).

La primera versió, amb OAuth, quedava oberta a la teva versió de n8n i es va tancar de seguida.

## Artefacte: «Fes-ho tot en segon pla»
El botó obre una sessió de Claude Code al núvol (`create_session`, entorn «Claude-Code») amb tot l'encàrrec i les dades del formulari. Fa la feina encara que tanquis l'app i, quan acaba, envia una notificació (`PushNotification`).
- Les sessions creades així tenen els connectors «Cuina n8n» i Notion (provat el 5-11-2026).
- Gasta l'ús del teu pla de Claude, igual que un xat llarg.
- La rutina «Cuina: menú i compra (en segon pla)» ja no es fa servir: la pots esborrar.

## Riscos coneguts
- **Totes dues són API internes no oficials:** el Cookidoo i el Mercadona les poden canviar sense avís.
- **El token del Mercadona caduca** cada 6 setmanes aproximadament. El flux t'avisa amb un missatge clar.

## Què no fa
- **No paga ni confirma cap comanda.** Només omple el carro.
- **No envia cap missatge**, ni a la Lupe ni a ningú.
- **No llegeix la llista de l'Alexa.** Amazon va tancar l'accés a les llistes l'1-7-2024.
