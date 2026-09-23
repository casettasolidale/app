# Anagrafica Casetta Solidale — istruzioni per Claude

App usata dai volontari di una distribuzione alimentare a Garbatella (Roma).
La responsabile non è una tecnica: **rispondi sempre in italiano, in modo semplice, senza gergo.**

## Com'è fatta l'app

- Un solo file: `anagrafica_casetta_solidale.html` (HTML + CSS + JS inline, niente framework, niente build).
- Pubblicata con GitHub Pages dal ramo `main`:
  https://casettasolidale.github.io/app/anagrafica_casetta_solidale.html
- Dati su Google Sheets (API v4). Login con Google Identity Services, token model
  (`initTokenClient`), solo nel browser, nessun backend.
- La usano più volontari contemporaneamente, soprattutto da telefoni Android con Chrome.
- Schede dell'app: Cerca, Messaggi (15 lingue), Turni (presenze, extra olio/assorbenti/pannoloni,
  note, stampa), Statistiche.

### Fogli Google

- **Anagrafica** (colonne A–T): codice, codice anon, attivo, turno, orario, nome e cognome,
  esigenze particolari, note, cellulare, NO WA, indirizzo, autocert, tot. N, ad, min, ol, ass, pan,
  banco, situazione banco/autocert.
- **giorni turni**: B2 = data di inizio del giro di turni ("turni a partire da");
  intestazioni in riga 4, dati dalla riga 5.
- **presenze**: oggi colonne A–H: codice, turno, data turno, SI/NO, note, olio, ass, pan.
  (Verrà ristrutturato con il compito 4: vedi "Lavori in corso".)

### Turni

- MER, GIO, VEN, SAB nelle settimane 1–4 (es. "MER 2" = mercoledì della seconda settimana del giro)
  + D (domicilio).
- Un "giro di turni" parte dalla data in `giorni turni!B2` e comprende tutti i turni fino al giro
  successivo: ogni persona ritira il pacco **una volta per giro**.

## Regole già decise (da non cambiare)

- `isAttivo` conta solo "sì" / "si" / "s".
- Il prefisso +39 si aggiunge solo ai numeri che rispettano `/^3\d{8,9}$/`.
- Persone totali = adulti + minori.
- Niente invio di massa né invio guidato su WhatsApp (tolti di proposito).
- Non cambiare `SPREADSHEET_ID`, `CLIENT_ID` o le colonne del foglio "Anagrafica" senza chiedere.
- Permessi Google (`SCOPES`): solo `https://www.googleapis.com/auth/spreadsheets`
  (tolto `drive.readonly`, l'app non usa Drive).

## Regole di lavoro

- Mai lavorare direttamente su `main`: usa un ramo e apri una pull request. Nella descrizione
  spiega in poche righe semplici cosa cambia e cosa provare.
- Alla fine di ogni pull request dai una lista breve di prove da fare sul telefono.
- Prima di proporre modifiche controlla che il JavaScript non abbia errori di sintassi:

  ```sh
  sed -n '/^<script>$/,/^<\/script>$/p' anagrafica_casetta_solidale.html | sed '1d;$d' > /tmp/app.js
  node --check /tmp/app.js
  ```

- **Attenzione agli apostrofi** nei testi dei messaggi in 15 lingue (`MSGS`): le stringhe sono tra
  apici singoli, un apostrofo non protetto (`'` invece di `\'` o `’`) rompe tutta l'app.
  È già successo.
- Il repository è **pubblico**: mai dati reali delle persone nel codice, nei test o negli esempi.
  Usa solo dati inventati.
- Per le scelte che cambiano come si usa l'app o come sono organizzati i dati: prima spiegare le
  opzioni e aspettare l'ok.
- Lavori lunghi: dividerli in più pull request, nell'ordine dei compiti.
- Claude non vede il foglio Google. Quando serve modificarlo (nuove schede, intestazioni, spostamento
  dati) si guida la responsabile passo passo in chat, un passaggio alla volta, aspettando conferma.
- Tenere aggiornato questo file con le decisioni prese.

## File nel repository

- `anagrafica_casetta_solidale.html` — l'app.
- `test.html`, `app2.html` — copie vecchie, cancellate (2026-09, con conferma). Si recuperano dalla
  storia del repository se servissero. `test.html` puntava a un altro foglio Google.

## Decisioni prese

- 2026-09 — Tolto il permesso `drive.readonly`: resta solo `spreadsheets`.

## Lavori in corso (compiti concordati)

1. CLAUDE.md — fatto.
2. Permessi e pulizia — fatto (permesso Drive tolto, copie vecchie cancellate).
3. Restare collegati dopo un refresh e rinnovo del token scaduto con nuovo tentativo del salvataggio.
4. Nuovo modo di registrare i ritiri ("Conferma ritiro" per persona, niente più "Chiudi turno"),
   nuova struttura del foglio "presenze" (una riga per ritiro), vecchi dati in "presenze_archivio",
   assenti registrati alla chiusura del giro.
5. Persone che vengono una tantum in un turno diverso dal proprio (solo per il giro in corso).
6. Orari fissi ogni 10 minuti con orario suggerito; elenco "Orari da sistemare" nelle Statistiche.
