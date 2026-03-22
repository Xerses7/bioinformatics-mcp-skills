# PPI Analyze — RHO (Fototransduction)

> **Skill ottimizzata per la rodopsina (RHO) e i geni della cascata di fototransduction.**
> Il contesto biologico è orientato alla segnalazione nei bastoncelli, al ciclo del retinale e alla patologia adRP (RP4 autosomica dominante).
> Per analisi generica su qualsiasi gene usa `/ppi-analyze GENE`.

Analizza le interazioni proteina-proteina della rodopsina aggregando dati da UniProt, STRING e WikiPathways, con interpretazione nel contesto della cascata visiva dei bastoncelli.

## Come usare
`/rho-ppi-analyze GENE_SYMBOL`

Esempi:
- `/rho-ppi-analyze RHO`
- `/rho-ppi-analyze GRK1`
- `/rho-ppi-analyze SAG`

## Istruzioni per Claude

L'utente vuole analizzare il gene: $ARGUMENTS

Esegui questi passi in sequenza:

1. **Esegui lo script** dalla cartella del progetto:
   ```
   python ppi_analyzer/main.py --gene $ARGUMENTS
   ```

2. **Interpreta l'output** e presenta all'utente:
   - ID UniProt e nome completo della proteina
   - Top 5 GO terms con categoria (Molecular Function / Biological Process / Cellular Component)
   - Malattie associate in forma leggibile
   - Top 10 interattori STRING con score e tipo di evidenza, ordinati per score decrescente
   - Pathway WikiPathways se presenti

3. **Commento biologico nel contesto della fototransduction:**

   La cascata di segnalazione visiva nei bastoncelli segue questo ordine:
   ```
   Luce → RHO* (rodopsina attivata)
            ↓
   GNAT1/GNB1/GNGT1 (transducina) → attivata
            ↓
   PDE6A/PDE6B (fosfodiesterasi cGMP) → idrolizza cGMP
            ↓
   CNGA1/CNGB1 (canali CNG) → chiusura → iperpolarizzazione
            ↓
   Segnale visivo trasmesso alla cellula bipolare

   Recupero del segnale:
   RHO* + GRK1 → RHO*-P (fosforilata)
   RHO*-P + SAG (arrestina) → segnale spento
   RCVRN (recoverin) regola GRK1 in funzione del Ca²⁺
   ```

   Posiziona il gene analizzato in questo schema e spiega quale step della cascata controlla.

4. **Suggerisci un prossimo passo:**
   - "Per vedere le PTM e i siti di fosforilazione/palmitoilazione: `/rho-ptm-analyze $ARGUMENTS`"
   - "Per confrontare gli interattori con la letteratura: `/rho-ppi-compare $ARGUMENTS`"

### Note tecniche
- Lo script usa solo librerie standard Python
- RHO è il gene più espresso nell'intera retina (>90% del contenuto proteico dei segmenti esterni dei bastoncelli)
- Verificare che STRING ritorni interattori retinici specifici e non generici del pathway GPCR
