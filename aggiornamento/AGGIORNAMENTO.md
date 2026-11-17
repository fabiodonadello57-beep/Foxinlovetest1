# Fox in Love — aggiornamento 2 (novità aggiunte dopo il primo prototipo)

> Da dare all'assistente AI insieme alla cartella `riferimento/`. È aggiornata con tutte le novità qui sotto: è lo stesso prototipo di prima con queste aggiunte. Il JavaScript dentro `index.html` è già convertito in una forma compatibile con i telefoni vecchi (vedi punto 7).

Il sito su foxinlove.vercel.app è già ok. Mancano solo queste novità:

## 1. Taglie
- Nuovo campo **taglia** su ogni capo. Di default **donna 42/44**, **uomo 50** (già impostato in `catalogo.js`).
- Nella gestione, dentro "Modifica" e "Nuovo capo", **menu a tendina** che cambia secondo Donna/Uomo:
  - **Donna:** 38/40 · 40/42 · 42/44 · 44/46 · 46/48 · 48/50
  - **Uomo:** 46 · 46/48 · 48 · 48/50 · 50 · 50/52 · 52 · 52/54 · 54 · 54/56 · 56
- Nella scheda del capo, accanto al codice: "Rif. FIL-0011 · Taglia 42/44".
- In "Altri filtri": gruppo **Taglia** con i pulsanti delle taglie presenti e il numero di capi.

## 2. Etichetta "Novità"
- Nuovo campo `nuovo` (sì/no), con il pulsante "Novità" nel modulo del capo.
- Sulla foto in vetrina compare il badge dorato **"Novità"**.
- I capi nuovi la prendono da soli per 30 giorni dalla creazione, a meno che Fabio non la tolga.
- In "Altri filtri → Dettagli" c'è anche il filtro **Novità**.

## 3. Pulsante WhatsApp
- Nella scheda del capo, sotto la descrizione: pulsante verde **"Chiedi informazioni su WhatsApp"**, tradotto nelle 8 lingue.
- Apre `https://wa.me/393482595447?text=...` con un messaggio già scritto **nella lingua del cliente**: saluto + codice + nome del capo. Esempio: "Buongiorno, vorrei informazioni su FIL-0013 – Giacchino in visone a zigzag bicolore".
- Nel piè di pagina, nei Contatti: **WhatsApp +39 348 259 5447**, con link a wa.me.

## 4. Anteprima del link (WhatsApp, Facebook, ecc.)
- Meta tag Open Graph nella pagina:
  - `og:title` "Fox in Love – Pellicce e giacche di lusso"
  - `og:description`
  - `og:image` (1200×630): c'è già pronta `riferimento/og.jpg`
  - `og:url`
  - `twitter:card` "summary_large_image"
- Per i link cliente `/v/...`, se possibile, generare i meta lato server con la **prima foto della selezione**.

## 5. Prezzi nei link cliente
- Capo **senza prezzo**: nel link compare **"Prezzo su richiesta"**, in corsivo.
- Capo **con prezzo e senza sconto**: sotto il prezzo una riga piccola **"Sconto su richiesta"**.
- Con uno sconto: resta come ora, prezzo pieno barrato e accanto quello scontato.
- Nelle 8 lingue (chiavi `ask` e `scAsk` in `ui.js`).

## 6. Scorrimento infinito più affidabile
Il solo IntersectionObserver, se le foto non sono ancora caricate (le schede sono basse), può non scattare più: dopo il primo blocco non arrivano altri capi. Nel riferimento c'è la funzione `checkMore()`:
- dopo ogni blocco ricontrolla se la fine della lista è ancora vicina allo schermo e, in quel caso, carica il blocco successivo (con `setTimeout`, non `requestAnimationFrame`);
- c'è anche un ascoltatore `scroll` (con pausa di 150 ms) e uno `resize`, come rete di sicurezza.

## 7. Compatibilità con telefoni vecchi (iPhone con iOS 12–14, Android economici)
- JavaScript convertito in **ES2017**: niente `?.` e `??` nel codice che arriva al browser. Con esbuild: `--target=es2017`.
- CSS: `inset:0` sostituito con `top/right/bottom/left:0`; per `aspect-ratio` c'è un fallback con `@supports not (aspect-ratio:3/4)` e il trucco del `padding-top`.
- `matchMedia(...).addEventListener('change')` con il ripiego `addListener`.

## 8. Listino
- In allegato c'è **Listino_semplice.xlsx** (colonne Codice e Prezzo): 29 prezzi, gli altri vuoti, cioè "su richiesta".
- Fabio deve poterlo caricare da "Listino prezzi → Carica Excel" e poi correggere i prezzi a mano nella tabella.
- Se l'importazione Excel non c'è ancora, va aggiunta (con SheetJS).

## Controllo finale
- [ ] In "Modifica" la taglia si cambia dal menu e si vede nella scheda; il filtro Taglia funziona.
- [ ] Il badge "Novità" si accende e si spegne, e il filtro Novità funziona.
- [ ] Il pulsante WhatsApp apre la chat con il messaggio giusto in italiano e in inglese.
- [ ] Mandando il link su WhatsApp compaiono foto e titolo.
- [ ] In un link con i prezzi: "Prezzo su richiesta" e "Sconto su richiesta" compaiono dove devono.
- [ ] Da telefono, scorrendo velocemente, arrivano tutti i capi fino in fondo.
- [ ] Carica Excel con `Listino_semplice.xlsx`: entrano 29 prezzi.

---

# Parte B — Nuove funzioni per lavorare con i clienti

Il contesto: Fabio vende in chat (WhatsApp) a clienti italiani e stranieri. Manda un link, il cliente sceglie, poi si tratta. Queste funzioni servono a quel flusso. Non sono nel prototipo `riferimento/`: vanno costruite nel sito vero, usando il backend già presente (link salvati, sessione di gestione, storage).

## B1. "Mi piace" del cliente → lista su WhatsApp
- In vetrina e nei link `/v/...`: un **cuoricino** su ogni scheda e nella scheda aperta. Si accende e si spegne, e viene ricordato nel browser del cliente (localStorage, per ogni link).
- In basso compare una barra fissa: **"♥ 3 capi scelti · Invia a Fox in Love"**.
- Premendo si apre `wa.me/393482595447` con un messaggio nella lingua del cliente, per esempio: *"Buongiorno, mi interessano questi capi: FIL-0012 Cappotto in visone rasato verde oliva, FIL-0045 …, FIL-0080 …"* più il link della selezione.
- Se possibile, salvare la lista anche sul server, legata al link, così Fabio la ritrova in gestione: "Link creati → Maria → ♥ 3 capi".

## B2. Sapere se il cliente ha aperto il link
- Ogni volta che si apre un link `/v/...`, il server registra **data e ora**, senza dati personali: niente IP salvati in chiaro, al massimo un contatore.
- Registra anche i **tocchi sul pulsante WhatsApp** e gli **invii della lista ♥**.
- In "Link creati", per ogni link: **"Aperto 3 volte · ultima ieri 21:40 · ♥ 2 · WhatsApp 1"**.
- Non contare le aperture fatte da Fabio stesso quando è collegato alla gestione.

## B3. Link con il nome del cliente
- In "Crea link", nuovo campo **"Nome del cliente (lo vede il cliente)"**, facoltativo e distinto dal "nome del link" che vede solo Fabio.
- Se compilato, il titolo della vetrina diventa **"Selezione per Maria"**, tradotto: "A selection for Maria", "Selección para Maria", ecc.

## B4. Stato "Riservato"
- Oltre a Disponibile / Venduto / Nascosto: **Riservato**.
- Sulla foto compare il badge "Riservato", tradotto, in un colore diverso da Venduto.
- Il capo resta visibile ma non si può mettere tra i ♥, oppure si può con un avviso.
- Il filtro "Solo disponibili" esclude anche i riservati.
- In gestione, pulsante "Segna riservato" nella scheda del capo. Facoltativo: un campo nota "riservato per Maria", visibile solo a Fabio.

## B5. Link di un solo capo
- Nella scheda del capo, in gestione: pulsante **"🔗 Link di questo capo"**. Crea un link `/v/...` con solo quel capo, con le stesse scelte di lingua, prezzi e sconto, e lo copia.
- Il cliente apre il link e vede direttamente la scheda del capo aperta, con le 3 foto.

## B6. Misure in centimetri
- Nuovi campi facoltativi per capo: **lunghezza**, **spalle**, **manica**, **circonferenza petto** (cm).
- Nella scheda del capo, sotto la taglia, una piccola tabella "Misure", tradotta. Compare solo se c'è almeno una misura.
- Si compilano dalla gestione, in "Modifica" e "Nuovo capo".

## Controllo finale Parte B
- [ ] Da un telefono senza login: ♥ su 3 capi → "Invia" apre WhatsApp con i 3 codici, nella lingua scelta.
- [ ] Il link aperto dal cliente risulta "Aperto 1 volta" in "Link creati". Aprirlo da Fabio collegato non conta.
- [ ] Un link con il nome "Maria" mostra "Selezione per Maria".
- [ ] Un capo segnato "Riservato" mostra il badge ed è escluso da "Solo disponibili".
- [ ] "Link di questo capo" apre la scheda di quel capo.
- [ ] Le misure inserite compaiono nella scheda, nelle lingue.
