# 2. Contratto API

**Richiesta:** Review il contratto di TransferRequest

**Delega:** Api-Contract-Reviewer Task — Review contratto TransferRequest

Review completata (api-contract-reviewer). Esito: FINDINGS.
Rilievi
F-01 — HIGH: Errori di validazione fuori dal formato di errore applicativo
GlobalExceptionHandler.java:14-23 (rel. a TransferRequest.java:10-17, MovementController.java:20) — Il DTO ha @NotNull/@DecimalMin e il controller applica @Valid, ma l'handler copre solo IllegalArgumentException e RuntimeException: un fallimento di bean validation (MethodArgumentNotValidException) non passa da errorBody(). Risultato: due formati di errore diversi per lo stesso 400 sullo stesso endpoint. Non applicata correzione: @ExceptionHandler(MethodArgumentNotValidException.class) con lo stesso body.
F-02 — MEDIUM: Nessun identificatore di richiesta/idempotenza nel contratto
TransferRequest.java:9-21, MovementController.java:19-22 — Il DTO ha solo fromAccountId, toAccountId, amount, description; né header né campo requestId, e TransferResponse non riferisce la richiesta: un client che ritenta non può distinguere un trasferimento duplicato da uno nuovo. Non applicata correzione: requestId/idempotencyKey nel DTO o header, riportato in risposta.
F-03 — MEDIUM: GET /api/movements espone l'entità JPA
MovementController.java:25-28, Movement.java:8-11 — listByAccount restituisce List<Movement> serializzata da Jackson: il contratto della GET è accoppiato al modello di persistenza, e ogni futuro campo dell'entità finirebbe nel JSON. Non applicata correzione: un MovementResponse DTO, come già fatto con TransferResponse.
F-04 — LOW: description senza limite dimensionale
TransferRequest.java:20 (rel. Movement.java:27) — Nessun @Size, colonna default 255: una descrizione troppo lunga passa la validazione ma fallisce al salvataggio, trasformando una richiesta "valida" in un 5xx. Non applicata correzione: @Size(max = 255).
F-05 — LOW: Messaggi di errore interni esposti nel body
MovementService.java:26,30,32,35, GlobalExceptionHandler.java:15-16 — ex.getMessage() è restituito tale e quale (es. "source account not found", "insufficient funds"): il contratto rivela esistenza dei conti e stato fondi, senza codici errore strutturati. Non applicata correzione: campo errorCode standardizzato, dettagli nei log.
Coerenza positiva verificata: tipi idonei (Long, BigDecimal), JSON camelCase standard, e il vincolo fromAccountId != toAccountId applicato nel service produce comunque 400.
Codice non modificato, come da regola della review.
