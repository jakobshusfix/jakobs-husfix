# Jakobs HusFix

Webbplatsen för Jakobs HusFix – fönsterputs i Stockholm – med bokning av tider
direkt på sidan.

## Innehåll

| Fil / mapp | Vad den innehåller |
| --- | --- |
| `index.html` | Hela sajten: innehåll, tjänster, priser, lediga tider, bokningsformulär och kontoflikarna. Inställningarna ligger i `window.JAKOBS_HUSFIX_CONFIG` längst ned i filen. |
| `css/` | Stilarna: `base.css` (färger, typografi, knappar), `layout.css`, `components.css` och `effects.css`. De laddas i den ordningen av `index.html`. `styles.css` är en äldre samlad fil som inte längre laddas – den kan tas bort. |
| `js/main.js` | Allt som händer i webbläsaren: lediga tider, ägarpanel, formulär, utskick och reservknappar. |
| `js/effects.js` | Småeffekter vid rullning och inläsning. |
| `images/` | Bilder och ikoner. |
| `google-apps-script/` | "Servern": bokningar till Gmail, tider, konton och ägarpanelens data. Har en egen README med installationen steg för steg. |

## Rutan med tider

På sajten finns tre flikar:

1. **Lediga tider** – de tider som går att boka. Grön = ledig, röd = upptagen
   (bokad av kund eller markerad av Jakob).
2. **Konto** – logga in och ut. Krävs för att ägarpanelen ska synas.
   **Logga ut**-knappen står under inloggningsfältet och, när du är inloggad,
   även högst upp i ägarpanelen – så att den alltid finns inom räckhåll.
3. **Ägare** – bara synlig för ett inloggat ägarkonto. Där markerar Jakob varje
   tid som **Bokad**, **Ledig** eller **Auto** (Auto = följ ordinarie schema).
   Markeringarna syns direkt för alla besökare, kundnamn visas bara för ägaren
   och en tid som en kund har bokat är låst. Under tiderna listas dagens
   kundbokningar, och en förfrågan som väntar på svar har knapparna **Godkänn**
   och **Neka** – samma beslut som mejlets länkar, men utan att lämna sajten.
   Anropet pekar ut förfrågan med sitt id, och om dagens bokningar skulle ha
   hämtats utan id (en äldre version av sajten) hittar servern förfrågan på
   datum, tid och namn i stället – knapparna fungerar alltså ändå.

Flikarna byts på samma plats i sidan och tonar in med en liten glidning medan
tiderna kommer in en efter en, så att bytet syns. Ägarkontots flik lyser till
när den låses upp efter inloggningen.

Några saker som är gjorda för att listan alltid ska vara kopplad och uppdaterad:

* **Listan är aldrig tom – och aldrig fel.** Det senaste svaret från servern
  sparas i webbläsaren, så den som kommer tillbaka ser tiderna direkt, även om
  webbappen hos Google precis startar om. Är det första besöket visas de
  ordinarie öppettiderna med en gång, med texten "de bokade tiderna kontrolleras
  just nu", och de riktiga tiderna tar över så snart svaret kommer.
* **Sajten kopplar upp sig igen av sig själv.** Går ett anrop inte igenom
  (kallstart eller tillfälligt fel hos Google) görs ett nytt försök efter 3, 6,
  12, 20 och sedan var 30:e sekund – tills svaret kommer. Ingen behöver trycka på
  något, och tiderna som redan visas ligger kvar under tiden.
* **Statuspricken visar läget.** Grön och stilla = kopplad till bokningsservern.
  Gul och pulserande = sajten kopplar upp sig just nu. Texten bredvid säger från
  vilket klockslag tiderna kommer.
* **Inga felsökningstexter på sajten.** Skulle ett meddelande om publicering,
  `Anyone` eller `/exec` dyka upp är det fel – sådant hör bara hemma i
  webbläsarens konsol (F12) och i `google-apps-script/README.md`. Besökaren får i
  stället en vänlig mening och telefonnumret.
* **Bokningen tappas aldrig bort.** Anropet till bokningssystemet talar om vilken
  åtgärd det gäller (`action=book`), så att förfrågan hamnar rätt. Kan servern
  ändå inte ta emot den – en kallstart, en äldre version av webbappen eller ett
  svar som inte kommer från bokningssystemet – visas ingen teknisk text. Sajten
  öppnar i stället **kundens eget e-postprogram** med hela förfrågan färdigifylld,
  och nödluckans knappar (där *Öppna i e-postprogrammet* nu får samma fullständiga
  mejl) finns kvar som vanligt. Uppgifterna ligger kvar i formuläret hela tiden.
  Svaret om en tid som faktiskt inte går att boka visas däremot som vanligt.
* **Ändringar syns direkt.** Efter en bokning, ett svar på en förfrågan (i
  ägarpanelen eller i mejlet) eller en markering hämtas listan om på en gång,
  och servern glömmer sin cache i samma stund, så nästa besökare ser ändringen
  direkt.

Sedan senaste uppdateringen går **Uppdatera** snabbt och kvitterar direkt:

* Knappen svarar i samma sekund som du klickar – den snurrar, byter text till
  "Uppdaterar…" och visar "Uppdaterad" när svaret har kommit. Under tiden står
  den gamla listan kvar, så inget försvinner medan servern arbetar.
* Sajten håller listan färsk av sig själv: var 45:e sekund medan den syns i vyn,
  och direkt när fliken kommer tillbaka. Är listan nedskrollad skickas i stället
  ett minimalt varmhållningsanrop, så att webbappen hos Google inte somnar och
  nästa blick på listan får svar på under en sekund.
* Servern svarar från en delad cache i fem minuter (se `google-apps-script/README.md`),
  så flera besökare delar på samma uträkning och svaret kommer på en bråkdel av
  tiden. En bokning eller markering tömmer cachen direkt.
* Flera samtidiga uppdateringar slås ihop till ett enda anrop, så
  bokningsservern aldrig får flera förfrågningar från samma besökare.
* Bara de dagar som har ändrats ritas om. De tider som bytte läge blinkar till en
  gång, resten av listan står still – därför hoppar sidan aldrig och besökaren
  ser ändå exakt vad som hände.
* Klockslaget för senaste kontrollen står under listan ("Uppdaterad kl 13:45")
  och läses inte upp av skärmläsaren, som bara får höra när antalet tider ändras.
  Står det i stället "Visar tiderna från kl 13:45 – uppdaterar automatiskt…" är
  tiderna de senaste vi har medan sajten försöker få ett färskt svar.

Hur ägarkontot skapas och vad de olika lägena gör i detalj: se
`google-apps-script/README.md` (steg 6 och 7). Ägarkontot är förberett med
användarnamnet `jakob` och lösenordet `JakobsHusFix2026` – kör funktionen
`skapaAgarkontot` en gång i Apps Script och logga sedan in under fliken
**Konto**. Byt lösenordet när du har loggat in första gången.

## Publicera sajten

1. Ladda upp `index.html`, `css/`, `js/` och `images/` till webbhotellet (eller
   GitHub Pages).
2. Koppla bokningarna till mejlen en gång: följ
   `google-apps-script/README.md` och klistra in webbappens `/exec`-adress i
   `bookingApiUrl` i `index.html`. Ladda upp `index.html` på nytt efteråt.
3. **Lägg ut den senaste `Code.gs`** i Apps Script och kör `installeraVarmhallning`
   en gång (se **Snabbare svar** i `google-apps-script/README.md`). Utan det
   steget fungerar sajten också – den visar ordinarie öppettider och fyller på när
   svaret kommer – men den delade cachen och varmhållningen, som gör listan
   snabb, är inte på plats förrän då. Godkänn/neka-knapparna i ägarpanelen hör
   också till den nya koden: ligger en äldre version ute gör de ingenting, och då
   lägger du ut den på nytt med
   **Deploy → Manage deployments → pennan → Version: New version → Deploy**.
4. Sajten fungerar även utan steg 2 – då skickas förfrågan via kundens eget
   e-postprogram i stället. Se `google-apps-script/README.md`.
5. **Logga in kräver steg 2.** Är `bookingApiUrl` tom svarar fliken **Konto**
   med en hänvisning till telefonnumret – fyll i adressen, ladda upp filen
   och öppna sajten med `Ctrl+F5`, kör sedan `skapaAgarkontot` i Apps Script.
   Går inloggningen ändå inte igenom: kör funktionen `kollaInloggningen` och läs
   loggen – den säger exakt vad som saknas.

## Testa backend lokalt (frivilligt)

```powershell
cscript //nologo google-apps-script\test-backend.js
```

Skriptet kör den riktiga `Code.gs` mot ett låtsaskalkylark och kontrollerar
bokningar, godkännande/nekande (både från mejlets länkar och från ägarpanelen,
även när anropet pekar ut förfrågan med datum och tid i stället för id), konton
och inloggning, ägarpanelens markeringar, den delade cachen, dubblettskyddet vid
ett nytt försök, att ett anrop från sajten utan `action` ändå behandlas som en
bokning samt att gäster aldrig ser kunduppgifter. Sista raden ska vara
`223 godkanda, 0 misslyckade`.

Inget behöver installeras – skriptet körs med Windows inbyggda `cscript` från
projektets rotmapp.
