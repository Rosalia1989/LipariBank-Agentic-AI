---
name: review-format
description: Formato condiviso dei report di code review per i tre reviewer LipariBank
---

# Formato dei rilievi

Ogni reviewer scrive un report Markdown in italiano con questa forma:

## Review
- Reviewer: `nome-reviewer`
- File esaminati: `percorsi reali`
- Esito: `FINDINGS` oppure `NESSUN_RILIEVO`

## Rilievi
Per ogni rilievo, in ordine di gravita:

### F-01 - Titolo del problema
- Gravita: `CRITICAL / HIGH / MEDIUM / LOW`
- File e riga: `src/.../File.java:numero`
- Evidenza: cosa si vede nel codice
- Conseguenza: cosa potrebbe succedere
- Correzione proposta (non applicata): cosa cambiare

Se non trovi problemi verificabili, scrivi `Nessun rilievo verificabile` e non crearne per riempire il report. Mantieni il report breve (massimo 5 rilievi utili). Non applicare correzioni e non inventare righe o file.
