# LipariBank — il progetto su cui lavori

Backend REST Spring Boot 3.3 su Java 21, con i tre concetti minimi di una banca: `Customer`, `Account`, `Movement`. Gira, ha un database vero e un'autenticazione vera, ed è la codebase che i tuoi agent leggeranno, recensiranno e interrogheranno per tre giorni.

Il bootcamp **non insegna Spring e non ti chiede di scrivere Java**. Questo progetto è il bersaglio: quello che scrivi tu sta in `.claude/`, in `.opencode/` e negli script che orchestrano gli agent. Il LipariBank è ciò su cui li fai lavorare.

Se arrivi dal Bootcamp Microservizi del catalogo Lipari e hai il tuo LipariBank Multi-Service, usa quello: è più ricco e va benissimo. Questo serve a chi arriva senza un progetto banking in mano, e nessuna giornata dà per scontato che tu abbia l'uno o l'altro.

---

## Farlo partire

Serve Docker. Maven no: il progetto porta il **wrapper**, che al primo uso scarica da sé la versione giusta.

```bash
docker compose up -d          # MySQL 8 sulla 3306, con volume persistente
./mvnw spring-boot:run           # l'applicazione sulla 8080
```

Liquibase crea lo schema e i dati di esempio al primo avvio: due clienti, due conti con saldo, due utenti.

Verifica che risponda:

```bash
curl -s localhost:8080/actuator/health
# → {"status":"UP"}
```

### Autenticarsi e fare un bonifico

Tutto ciò che non è `/api/auth/**` o `/actuator/**` vuole un token.

```bash
TOKEN=$(curl -s -X POST localhost:8080/api/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"username":"alice","password":"alice123"}' \
  | sed -E 's/.*"token":"([^"]+)".*/\1/')

curl -s -X POST localhost:8080/api/movements/transfer \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"fromAccountId":1,"toAccountId":2,"amount":100.00,"description":"prova"}'
```

Il secondo utente è `bob` / `bob123`, e i due conti di partenza hanno 1000,00 e 500,00 euro.

### Gli endpoint, tutti quelli che ci sono

| Metodo e percorso | Cosa fa | Token |
|---|---|:-:|
| `POST /api/auth/login` | restituisce un JWT dato username e password | no |
| `POST /api/movements/transfer` | trasferisce fra due conti e registra i movimenti | sì |
| `GET /api/movements?accountId=1` | i movimenti di un conto, dal più recente | sì |
| `GET /api/accounts/{id}` | un conto | sì |
| `GET /api/accounts/with-movements` | tutti i conti, ciascuno con i suoi movimenti | sì |
| `GET /actuator/health` | stato dell'applicazione | no |

---

## Com'è fatto

```
progetto-di-partenza/
├── pom.xml
├── Dockerfile
├── docker-compose.yml
├── .gitignore
└── src/
    ├── main/
    │   ├── java/com/lipari/bank/
    │   │   ├── LipariBankApplication.java
    │   │   ├── common/
    │   │   │   ├── CorrelationIdFilter.java      un id di correlazione nell'MDC dei log
    │   │   │   └── GlobalExceptionHandler.java   @RestControllerAdvice
    │   │   ├── customer/       Customer, CustomerRepository
    │   │   ├── account/        Account (saldo in BigDecimal), Repository, Controller
    │   │   ├── movement/       Movement, Repository, Service, Controller, dto/
    │   │   ├── security/       SecurityConfig, JwtFilter, JwtService
    │   │   └── web/            AuthController
    │   └── resources/
    │       ├── application.yml
    │       └── db/changelog/   lo schema e i dati di esempio, in Liquibase
    └── test/
        └── java/com/lipari/bank/TransferIT.java  il caso felice del bonifico
```

Diciannove classi Java, due changeset Liquibase. Il cuore è `MovementService.transfer()`: è il metodo transazionale che sposta il denaro, ed è il posto da cui conviene partire a leggere.

---

## Il repository git

Il progetto arriva come cartella, non come repository, e il `.gitignore` c'è già. Inizializzalo prima di cominciare, perché ogni giornata ti chiede di lavorare per commit:

```bash
git init
git add -A
git commit -m "LipariBank: punto di partenza del bootcamp"
```

---

## Cosa non è

Non è un sistema production-grade, e non pretende di esserlo: non ha circuit breaker, né retry, né audit trail completo, né multi-valuta. Non è multi-servizio: c'è un solo Spring service, che non parla con nessun altro. E non è pronto per un cluster: niente profili per ambiente, niente telemetria, niente probe pensate per la produzione.

Soprattutto: **non è un'architettura modello.** È un backend scritto come lo scrive una squadra sotto scadenza, con le scorciatoie che una squadra sotto scadenza prende. È esattamente per questo che serve: i tuoi reviewer avranno qualcosa di vero da trovare, e quello che troveranno non te l'ha suggerito nessuno.

Per la stessa ragione, se lo riusi fuori dal bootcamp trattalo come codice da recensire, non come codice da mostrare.

---

## Claude Code Assets

Questi controlli sono stati adattati a **OpenCode** a partire dallo starter di Gino. Il codice Java del LipariBank non è stato modificato.

### File, quando servono ed esempi

| Percorso | Quando serve (description) | Esempio |
| --- | --- | --- |
| `.opencode/agents/movement-reviewer.md` | Contabilizzazione, persistenza, transazioni e idempotenza; non AML o contratto HTTP | `Fai una review di MovementService.java sulla contabilizzazione dei bonifici` |
| `.opencode/agents/compliance-reviewer.md` | Importi, precisione, scala, limiti, AML/PEP/watchlist; non contabilità interna o endpoint | `Fai una review della validazione degli importi dei bonifici e dei controlli AML` |
| `.opencode/agents/api-contract-reviewer.md` | Endpoint, DTO, JSON, validazioni e codici HTTP; non AML o contabilità | `Review il contratto di TransferRequest` |
| `.opencode/skills/review-format/SKILL.md` | Formato comune dei rilievi per tutti e tre | Richiamata dai reviewer durante le review |
| `.opencode/skills/compliance-aml-check/SKILL.md` | Checklist AML ereditata, solo nel perimetro compliance | Richiamata da `compliance-reviewer` |
| `AGENTS.md` | Regole di scelta e disambiguazione per le deleghe | Si applica a una richiesta di review fatta nella root del progetto |
| `opencode.json` | Protezioni generali e autorizzazione delle deleghe | Si applica durante la sessione `opencode` |

### Regola di separazione dei tre perimetri

Il criterio è **il tipo di controllo richiesto**, non il nome della cartella. Per esempio, `MovementController.java` riguarda tutti e tre i domini, ma una richiesta su HTTP/DTO va ad **API**, una su importi/AML a **compliance** e una sulla registrazione dei movimenti a **movimenti**. Se la richiesta è ambigua o mescola i domini, prevale **API > compliance > movimenti**; gli altri due non vengono avviati. Su `pom.xml`, Maven, JWT o Docker non parte nessuno dei tre.

### Cosa ho cambiato rispetto allo starter di Gino e perché

- Ho trasformato il generico `code-reviewer` nel reviewer dei movimenti: la vecchia descrizione e la checklist mescolavano banking, sicurezza, stile e performance, rendendo poco chiaro il perimetro.
- Ho aggiunto il reviewer importi/compliance e quello del contratto API: ogni controllo ha così una sua responsabilità e non si attivano tutti sulla stessa richiesta.
- Ho portato i subagent da `.claude/agents/` a `.opencode/agents/`, riscrivendo il frontmatter con `mode` e `permission`: OpenCode non usa il formato ereditato di Claude Code. Ho rimosso anche `model: sonnet`.
- Ho portato la skill AML in `.opencode/skills/`, conservandone i cinque temi di controllo, ma togliendo l'allowlist di scrittura: una review non deve apportare correzioni. Le soglie numeriche dello starter vanno confermate con una policy, non sono considerate automaticamente obbligatorie.
- Ho scelto una **skill** per AML, non un MCP server: qui serve una checklist locale condivisa durante le review, non un servizio esterno da interrogare.
- Ho aggiunto la skill comune `review-format`, per avere gravità, file e riga, evidenza e correzione proposta senza duplicare lo schema nei tre reviewer.
- Ho sostituito il vecchio hook `PostToolUse` di Gino con i permessi OpenCode: quell'hook registrava le chiamate, ma non impediva le scritture e non viene applicato da OpenCode. Tutti i reviewer hanno `edit: deny` e la shell negata tranne il comando `git status` per verificare lo stato del repository. Non ho mantenuto lo script di log ereditato.
- Ho aggiunto `AGENTS.md` per rendere esplicita la scelta automatica di **un solo reviewer** e la regola di tie-break; la configurazione `opencode.json` autorizza soltanto queste deleghe.

### Verifiche fatte e prove

Nella stessa sessione OpenCode sono state eseguite due richieste di domini diversi: quella su `MovementService.java` ha avviato `movement-reviewer`, quella su `TransferRequest` ha avviato `api-contract-reviewer`. I risultati originali sono salvati in `review-MovementService.md` e `review-TransferRequest.md`.

Sono state eseguite anche una review degli importi/AML tramite `compliance-reviewer`, una richiesta fuori perimetro su `pom.xml` senza delega specializzata e una prova di sola lettura: il reviewer ha continuato a leggere il codice senza modificarlo.

### Verifica della configurazione aggiornata

Dopo la modifica dei permessi, è stato eseguito un controllo aggiuntivo in OpenCode: `movement-reviewer` ha letto `MovementService.java` e ha riportato l'esecuzione di `git status`; non ha eseguito `git log -1`, che non è tra i comandi autorizzati. Il test non ha modificato file. Questo controllo riguarda i permessi shell, mentre la prova di protezione dalla scrittura del codice è documentata in `evidenze/G1-05-Sola-lettura.png`.

La prova del routing è documentata in `evidenze/run-trace-routing.md`, con due richieste nella stessa sessione, e nei file `review-MovementService.md` e `review-TransferRequest.md`. Gli screenshot delle review si trovano nella cartella `evidenze/`.
