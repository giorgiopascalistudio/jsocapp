# JSOC – Setup Firebase Auth (Google Sign-In)

Questa guida ti porta dall'attuale Realtime DB aperto a un sistema sicuro con login Google.
Tempo stimato: **15 minuti**.

---

## 1. Configurazione Firebase Console

### 1.1 Apri il tuo progetto
- Vai su https://console.firebase.google.com
- Seleziona `jsoc-map` (il progetto del Realtime DB esistente)

### 1.2 Abilita Authentication
1. Menu sinistra → **Build → Authentication**
2. Tab **Sign-in method**
3. Click su **Google** → toggle **Enable**
4. Imposta un **nome pubblico** (es: "JSOC SUITE") e un **email di supporto**
5. **Salva**

### 1.3 Aggiungi i domini autorizzati
- In Authentication → Settings → **Authorized domains**
- Aggiungi:
  - `localhost` (per test locali)
  - `<tuo-username>.github.io` (per GitHub Pages)
  - Eventuale dominio custom

### 1.4 Recupera la Web Config
1. ⚙️ → **Project settings**
2. Scorri fino a **Your apps**
3. Se non hai una "Web app", click sull'icona `</>` per aggiungerla. Nome: `JSOC Web`. **Non** registrare hosting.
4. Copia il blocco `firebaseConfig`. Ti servono questi valori:
   - `apiKey`
   - `authDomain`
   - `projectId`

---

## 2. Incolla la config nel codice

Nel file `index.html`, cerca la funzione `_initAuthApp()` (verso fondo, riga ~3070 circa).
Sostituisci i placeholder con i valori reali:

```js
_authApp = firebase.initializeApp({
    databaseURL: 'https://jsoc-map-default-rtdb.europe-west1.firebasedatabase.app',
    apiKey: 'AIzaSy...IL_TUO_API_KEY_QUI...',     // ← incolla qui
    authDomain: 'jsoc-map.firebaseapp.com',         // ← se è diverso, modifica
    projectId: 'jsoc-map'                           // ← idem
}, 'jsoc_auth');
```

⚠️ **L'apiKey NON è un segreto** — è visibile lato client. La sicurezza viene dalle Security Rules + dai domini autorizzati.

---

## 3. Carica le Security Rules

1. Console Firebase → **Build → Realtime Database**
2. Tab **Rules**
3. Cancella tutto e incolla il contenuto di `firebase-rules.json` (questo repository)
4. **Publish**

### Cosa fanno queste regole
- 🔒 Solo utenti autenticati possono leggere/scrivere
- 🔒 Ogni utente può modificare SOLO il suo profilo `/users/{uid}`
- 🔒 La posizione live `/squads/{sid}/players/{uid}` può essere scritta solo dall'utente stesso E solo se è membro della squadra
- 🔒 La chat è append-only: un messaggio inviato non può essere modificato
- 🔒 Solo il leader può modificare le `info` della squadra
- 🔒 I marker condivisi possono essere modificati solo da chi li ha creati

---

## 4. (Opzionale) Migrazione dati esistenti

Se hai dati nel vecchio path `/jsoc/{squad}/...` puoi:
- **Opzione A**: lasciarli — il nuovo sistema usa `/squads/{sid}/...`, sono separati
- **Opzione B**: cancellare il vecchio nodo manualmente da console
- **Opzione C**: ri-aprire temporaneamente le regole per esportare i dati

Le regole sopra **bloccano** il vecchio path `/jsoc` per sicurezza.

---

## 5. Test

1. Apri `index.html` (o il tuo URL GitHub Pages)
2. Vedrai l'overlay LOGIN
3. Click su **ACCEDI CON GOOGLE** → flusso OAuth standard
4. Dopo il login vedi la HOME con:
   - In alto a sx: barra squadra (vuota)
   - In alto a dx: il tuo nome
5. Apri **PROFILO** → modifica callsign, colore, ruolo
6. Apri **SQUADRA** → crea squadra → ottieni codice 6 caratteri
7. Su un altro dispositivo/account: login → SQUADRA → unisciti col codice
8. Apri **MAPPA TATTICA** → vedi posizioni e icone con ruolo+stato
9. Apri la sidebar `☰` → scorri fino a **CHAT SQUADRA** → invia messaggio

---

## 6. Limiti del piano Spark (gratuito)

- **Auth**: 50.000 utenti attivi/mese
- **Realtime DB**: 1 GB storage, 10 GB/mese download
- **100 connessioni simultanee** → per una squadra airsoft basta abbondantemente

Per usi più intensi → upgrade al piano **Blaze** (pay-as-you-go, ma il gratis incluso è generoso).

---

## 7. Troubleshooting

| Problema | Causa probabile | Soluzione |
|---|---|---|
| Popup login bloccato su iOS | Safari blocca popup non da tap diretto | ✅ Già gestito: il codice rileva iOS/PWA e usa `signInWithRedirect` automaticamente |
| `auth/unauthorized-domain` | Dominio non in lista | Aggiungilo in Authentication → Settings |
| `PERMISSION_DENIED` su write | Regole troppo strette / auth scaduta | Verifica che l'utente sia membro della squadra |
| Mappa vuota dopo login | Listener vecchio path `jsoc/...` | Il nuovo codice usa `squads/{sid}/...` — verifica di aver creato/unito una squadra |
| Redirect Google torna ma resta su login | `getRedirectResult` non risolve | Controlla che `authDomain` nella config sia corretto e nei domini autorizzati |

---

## 8. Cose che NON ho ancora implementato (roadmap)

- 📸 Foto/note geolocalizzate (richiede Firebase Storage)
- 🥖 Breadcrumb trail (richiede ottimizzazione DB)
- 📜 Cronologia missioni / log dettagliato
- 🔄 Trasferimento leadership a un altro membro
- ✅ Approvazione richieste di ingresso (deciso di fare dopo)
- 🔇 Mute/disattivazione utenti dal leader

Sono tutte funzioni "additive": il sistema attuale è il fondamento.
