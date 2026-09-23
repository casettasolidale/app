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
- **presenze**: una riga per ogni ritiro (dal compito 4). Colonne A–L:
  codice | inizio giro (valore di B2) | turno previsto (da Anagrafica) | turno effettivo |
  fuori turno (SI se effettivo ≠ previsto) | data ritiro (giorno in cui è venuta; vuota se assente) |
  esito (SI/NO) | olio | ass | pan (SI/NO solo se la persona ne ha diritto, altrimenti vuoto) |
  nota (automatica: correzioni, "assente: giro chiuso il …") | registrato il (gg/mm/aaaa hh:mm).
  Scritte con `valueInputOption=RAW` (testo così com'è). Se la riga 1 non ha "inizio giro" in B1,
  l'app considera il foglio di vecchia struttura e blocca le conferme.
- **presenze_archivio**: i dati della vecchia scheda "presenze" (colonne A–H: codice, turno,
  data turno, SI/NO, note, olio, ass, pan). L'app non la legge.

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
- 2026-09 — Accesso Google: token e scadenza in `sessionStorage` (chiave `casetta_sessione`),
  non in `localStorage`. Dopo un refresh si resta collegati; chiudendo la scheda o con "Esci"
  l'accesso sparisce (più sicuro sui telefoni condivisi). Tutte le chiamate ai fogli passano da
  `apiFetch`: se il token è scaduto o Google risponde 401, compare la barra gialla "Continua"
  (un tocco, `requestAccessToken({prompt:''})` con `login_hint` se l'email è nota) e la chiamata
  viene ripetuta. Non usare più `fetch` diretto verso sheets.googleapis.com.
- 2026-09 — Dopo un refresh l'app riapre la stessa vista (scheda, turno aperto, turno e lingua dei
  Messaggi), salvata in `sessionStorage` (chiave `casetta_vista`). Ogni accesso nuovo dalla
  schermata "Accedi con Google" (e "Esci") la cancella: si parte da Cerca. Deciso: se il telefono
  resta collegato e passa da un volontario all'altro, va bene che resti sull'ultima scheda.
- 2026-09 — Telefoni condivisi: dalla schermata di accesso Google mostra sempre la scelta dell'account
  (`prompt:'select_account'`, senza `login_hint`). La barra "Continua" (rinnovo a sessione in corso)
  usa invece `prompt:''` con `login_hint` della persona collegata.

- 2026-09 — Registrazione ritiri (compito 4): niente più "Chiudi turno". Ogni persona ha le sue
  spunte (restano solo sul telefono) e "✓ Conferma ritiro", che rilegge il foglio e fa una sola
  scrittura (se c'è già un SI nel giro non scrive; se c'è un NO aggiorna quella riga). "Correggi"
  riscrive la stessa riga (la data del ritiro resta quella originale). Il pulsante note si chiama
  "Registra nota" (scrive in Anagrafica!H come prima). Non si registra il nome del volontario
  (deciso: non serve).
- 2026-09 — Chiusura del giro: cambiando la data "Turni a partire da" l'app, dopo conferma, segna
  esito NO per le persone attive **con un turno** (domicilio D compreso) senza ritiro nel giro che si
  chiude, poi scrive B2 e mostra presenti/assenti. Ripetibile senza doppioni.
- 2026-09 — Orari (compito 6): configurazione `FASCE_ORARIE` e `PASSO_MINUTI` in cima allo script
  (MER/GIO 18:00–19:50, VEN/SAB 10:00–12:40, D nessun orario). Nel modulo l'orario è un menu con i
  soli orari validi del turno e quante persone attive ci sono già; scegliendo il turno si preseleziona
  l'orario con meno persone (a parità il più presto). Nessun limite di persone per orario. Un orario
  salvato non valido (o mancante, per turni con orari) si mostra con un avviso e va scelto un orario
  valido prima di salvare; il foglio non viene mai corretto in automatico. Orari come "18.00" o
  "18:00:00" sono considerati validi (stesso orario). In Statistiche: elenco "Orari da sistemare".

## Lavori in corso (compiti concordati)

1. CLAUDE.md — fatto.
2. Permessi e pulizia — fatto (permesso Drive tolto, copie vecchie cancellate).
3. Restare collegati dopo un refresh e rinnovo del token scaduto — fatto (vedi Decisioni).
4. (fatto, vedi Decisioni) Nuovo modo di registrare i ritiri ("Conferma ritiro" per persona, niente più "Chiudi turno"),
   nuova struttura del foglio "presenze" (una riga per ritiro), vecchi dati in "presenze_archivio",
   assenti registrati alla chiusura del giro.
5. Persone che vengono una tantum in un turno diverso dal proprio (solo per il giro in corso).
6. (fatto, vedi Decisioni) Orari fissi ogni 10 minuti con orario suggerito; elenco "Orari da sistemare" nelle Statistiche.
