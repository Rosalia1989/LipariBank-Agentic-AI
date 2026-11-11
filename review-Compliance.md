# 3. Importi e AML

**Richiesta:** Fai una review della validazione degli importi dei bonifici e dei controlli AML.

**Delega:** Compliance-Reviewer Task — Review importi e AML bonifici

Review completata (compliance-reviewer). Esito: FINDINGS.
Rilievi
F-01 — CRITICAL: Assenza totale di controlli AML
MovementService.java:24-62 (assenza su tutta la codebase) — Nessuna soglia di segnalazione, screening PEP o watchlist: la ricerca di PEP/watchlist/soglia/threshold non produce match in src/main, e Customer.java:10-25 non ha flag isPep. Bonifici di qualsiasi importo passano senza segnalazioni né registrazione delle operazioni sopra soglia. Non applicata correzione: componente di screening invocato in transfer() prima dell'esecuzione + registrazione sopra soglia.
F-02 — HIGH: Nessun limite di importo per operazione o giornaliero
MovementService.java:24-41 — Unico controllo: la disponibilità (riga 34). Un bonifico dell'intero saldo è sempre possibile e non c'è aggregazione giornaliera né anti-frammentazione (punto 5 della checklist AML). Non applicata correzione: limite per operazione e controllo sull'aggregato giornaliero dei TRANSFER_OUT.
F-03 — MEDIUM: Scala dell'importo non vincolata
TransferRequest.java:16-18, MovementService.java:38-39 — amount ha solo @NotNull/@DecimalMin("0.01"), senza @Digits; le colonne sono precision = 19, scale = 2 (Account.java:21-22, Movement.java:21-22). Un importo come 0.011 arriva al calcolo con scala > 2: differenza tra saldo in memoria (restituito in TransferResponse) e persistito. Non applicata correzione: @Digits(integer = 17, fraction = 2) e/o setScale(2, HALF_UP) nel service.
F-04 — MEDIUM: Valuta non verificata, cambio implicito 1:1
MovementService.java:38-39, Account.java:24-25 — Il service non confronta mai from.getCurrency() con to.getCurrency(): bonifico tra valute diverse eseguito al cambio 1:1 implicito. Non applicata correzione: rifiutare se le valute non coincidono (o flusso FX dedicato).
F-05 — LOW: Segno dell'importo non ricontrollato nel service
MovementService.java:24-36 — amount > 0 garantito solo da @DecimalMin nel DTO: un importo negativo che bypassi @Valid passerebbe il check di riga 34 e subtract (riga 38) aumenterebbe il saldo del mittente. Non applicata correzione: difesa esplicita signum() <= 0 nel service.
Nota di conformità: le soglie citate (10.000/5.000/1.000 EUR) derivano dalla checklist dello starter — nessuna policy documentata nel progetto, vanno confermate contro i requisiti reali.
Codice non modificato, come da regola della review.
