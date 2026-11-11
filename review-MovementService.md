# 1. Contabilizzazione dei bonifici

**Richiesta:** Fai una review di MovementService.java sulla contabilizzazione dei bonifici.

**Delega:** Movement-Reviewer Task — Review contabilizzazione MovementService

Review completata (movement-reviewer). Esito: FINDINGS.
Rilievi
F-01 — HIGH: Nessuna idempotenza → doppio bonifico sui retry
MovementService.java:23-62 — Il metodo transfer non ha chiave di idempotenza: TransferRequest non ha requestId/clientRef, Movement non ha vincoli di unicità dedicati. Un retry (timeout di rete, doppio submit) esegue due bonifici distinti: doppio addebito e doppio accredito. Non applicata correzione: chiave di idempotenza persistita con vincolo di unicità, rilevata dentro la transazione.
F-02 — HIGH: Race condition sul saldo (lost update)
MovementService.java:29-41 — Saldi letti con findById, check fondi in memoria (riga 34) e aggiornamento read-modify-write via save (righe 38-41). AccountRepository non ha lock pessimistico e Account non ha @Version: due trasferimenti concorrenti dallo stesso conto possono entrambi passare il check e sovrascriversi il saldo. Non applicata correzione: @Lock(PESSIMISTIC_WRITE) o optimistic locking con retry.
F-03 — MEDIUM: Bonifico fra valute diverse senza controllo
MovementService.java:34-41 — Account ha un campo currency (righe 25-27) ma il service non confronta mai from.getCurrency() e to.getCurrency(): lo stesso importo numerico viene addebitato e accreditato senza conversione. Non applicata correzione: validare coincidenza valute o conversione esplicita.
F-04 — LOW: Gambe del bonifico non collegate da un identificativo
MovementService.java:43-59 — TRANSFER_OUT e TRANSFER_IN sono record indipendenti, con due Instant.now() distinti e nessun transferId su Movement: in caso di più bonifici fra gli stessi conti non si ricostruisce univocamente la contropartita. Non applicata correzione: identificativo di operazione condiviso + timestamp unico.
Nota positiva: la transazionalità è corretta — @Transactional a riga 23 copre addebito, accredito e le due scritture, e le eccezioni sono runtime, quindi compatibili con il rollback automatico.
Come da regola della review, il codice non è stato modificato.
