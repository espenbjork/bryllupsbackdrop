# Bryllupsbackdrop – Eirin & Jonas, 11. juli 2026

Live backdrop for prosjektoren i Høymagasinet, med gjestehilsener i sanntid.

**Tre deler:**

| Fil | Hva | Hvem |
|---|---|---|
| `index.html` | Backdropet på prosjektoren (monogram + animasjoner + hilsener) | Toastmasterne |
| `send.html` | Mobilside der gjestene sender hilsener/emojier (via QR på skjermen) | Gjestene |
| `apps-script/Code.gs` | Backend: lagrer i Google Sheets, genererer gjestebok-doc etterpå | Google |

**Flyt:** Gjest sender hilsen → Apps Script skriver til regneark → skjermen henter nytt hvert 3. sekund → etter bryllupet kjøres `lagGjesteboken()` som lager et ferdig formatert Google-dokument til brudeparet.

---

## Oppsett

### Steg 1: Apps Script-backend (~5 min)

1. Gå til [script.google.com](https://script.google.com) → **Nytt prosjekt**
2. Slett innholdet i editoren og lim inn alt fra `apps-script/Code.gs`
3. Velg funksjonen `setup` i menylinjen → **Kjør** → godkjenn tilgangene
   (dette oppretter regnearket "Eirin & Jonas – hilsener")
4. **Deploy → New deployment → Web app:**
   - *Execute as:* **Me**
   - *Who has access:* **Anyone**  ← viktig, ellers får ikke gjestene sendt
5. Kopier web-app-URL-en (`https://script.google.com/macros/s/.../exec`)

### Steg 2: Lim URL-en inn i `config.js`

```js
const SCRIPT_URL = "https://script.google.com/macros/s/DIN-ID-HER/exec";
```

### Steg 3: Push til GitHub og skru på Pages

```bash
cd bryllupsbackdrop
gh repo create bryllupsbackdrop --public --source=. --push
```

Forventet output:
```
✓ Created repository espenbjork/bryllupsbackdrop on GitHub
✓ Added remote https://github.com/espenbjork/bryllupsbackdrop.git
✓ Pushed commits to https://github.com/espenbjork/bryllupsbackdrop.git
```

Skru på GitHub Pages:
```bash
gh api repos/{owner}/bryllupsbackdrop/pages -X POST -f "source[branch]=main" -f "source[path]=/"
```

Forventet output (utdrag):
```json
{
  "url": "https://api.github.com/repos/espenbjork/bryllupsbackdrop/pages",
  "status": "building",
  "html_url": "https://espenbjork.github.io/bryllupsbackdrop/"
}
```

Etter 1–2 minutter er siden live på `https://<brukernavn>.github.io/bryllupsbackdrop/`.
QR-koden på skjermen genereres automatisk og peker til riktig `send.html`.

---

## På bryllupsdagen

- Åpne `https://<brukernavn>.github.io/bryllupsbackdrop/` på PC-en koblet til prosjektoren
- **Klikk én gang** på siden → fullskjerm (wake-lock hindrer at skjermen sovner)
- Statusprikken øverst til høyre er grønn/diskret når alt virker, oransje ved nettproblemer
- Gjestene skanner QR-koden nederst til venstre på skjermen

**Test på forhånd uten backend:** åpne `index.html?demo=1` – da vises eksempelhilsener.

## Etter bryllupet

I Apps Script-editoren: velg funksjonen `lagGjesteboken` → **Kjør**.
URL til det ferdige Google-dokumentet ligger i loggen (Ctrl+Enter / "Executions").
Dokumentet inneholder alle hilsener med navn og klokkeslett, pluss emoji-statistikk («Jubel fra salen»).

## Verdt å vite

- **Ordfilter:** enkle stygge ord avvises i backend (listen `BLOKKERTE_ORD` i `Code.gs` kan justeres). Ingen manuell moderering – alt annet går rett på skjermen.
- **Dårlig dekning:** gjestesiden køer hilsener lokalt og sender automatisk når nettet er tilbake.
- **Forsinkelse:** opptil ~3 sek fra sendt til synlig (polling). Ved meldingsbyger slippes de på skjermen én og én med ~2,5 sek mellomrom.
- **Historikk:** skjermen viser bare hilsener som kommer inn *etter* at den ble åpnet – en omstart spammer ikke skjermen med gamle meldinger. Alt ligger trygt i regnearket uansett.
