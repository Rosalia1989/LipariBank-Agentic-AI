# Review LipariBank - Giorno 1

Quando l'utente chiede una review del codice, scegli **un solo reviewer**, senza chiedergli di indicare il nome:

- `movement-reviewer`: contabilizzazione, transazioni, persistenza e idempotenza dei bonifici.
- `compliance-reviewer`: importi, BigDecimal, limiti, AML, PEP e watchlist.
- `api-contract-reviewer`: endpoint REST, Controller, DTO, risposte e contratto HTTP.

Decidi dal **tipo di controllo richiesto**, non dalla cartella. Per `MovementController.java`: se la domanda e sul contratto HTTP usa API; se e sugli importi usa compliance; se e sulla contabilizzazione usa movimenti. Se la domanda non specifica il controllo, per questo Controller prevale API. Se la richiesta mescola piu aree, scegli una sola con questa priorita: API, poi importi/compliance, poi movimenti. Se indica chiaramente un solo tipo di controllo, segui quello.

Per build, `pom.xml`, Docker, stile generico, JWT o altri temi esterni a questi tre ambiti, **non avviare nessuno dei tre reviewer**.

Delega realmente usando `task`: non limitarti a nominare il reviewer. Chiedi una review in sola lettura, con la skill `review-format` e solo evidenze verificabili. Non modificare il codice durante le review, anche se l'utente chiede di correggere i problemi.
