---
name: compliance-aml-check
description: Checklist AML di Gino, da usare durante le review degli importi e delle regole di compliance del LipariBank
---

# Compliance AML Check - LipariBank

Questi cinque punti derivano dalla skill originale di Gino. Sono controlli da verificare, non norme di legge da dare per scontate.

1. **Soglie operative** - Gino indicava 10.000 EUR per segnalazione e 5.000 EUR per audit: confrontarle con la policy reale prima di considerarle obbligatorie.
2. **PEP screening** - Controllare l'eventuale flag `isPep` e gli alert. Lo starter proponeva 1.000 EUR: vale solo se confermato dai requisiti.
3. **Watchlist** - Verificare se e quando il progetto richiede lo screening del cliente (Gino citava `watchlistService.screen(fiscalCode)`).
4. **Audit trail** - Verificare gli identificativi utili all'audit: `correlationId`, `userId`, `ipAddress` ed `executedAt`, secondo quanto richiede il progetto.
5. **Operazioni sospette** - Cercare controlli su frammentazione, giri circolari, conti dormienti e anomalie geografiche, se previsti dalla policy.

Quando una regola non e documentata, indicare che e da verificare: non inventare violazioni. Restituire i rilievi con il formato della skill `review-format`. Solo lettura, nessuna correzione applicata.
