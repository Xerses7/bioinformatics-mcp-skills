# PPI Compare — RHO (Fototransduction)

> **Skill specifica per RHO e la cascata di fototransduction nei bastoncelli.**
> Include gli interattori di default della cascata visiva rod dalla letteratura consolidata.
> Per analisi comparativa generica usa `/ppi-compare GENE INTERACTORS`.

Confronta gli interattori di RHO tra letteratura (cascata di fototransduction) e database STRING, identificando gap e candidati per validazione sperimentale nel contesto di adRP (RP4).

## Come usare
`/rho-ppi-compare GENE_SYMBOL [INTERACTOR1,INTERACTOR2,...]`

Esempio con default: `/rho-ppi-compare RHO`
Esempio custom: `/rho-ppi-compare RHO GRK1,SAG,GNAT1,PDE6A,PDE6B,CNGA1,RCVRN`

Se non vengono forniti interattori, usa il **set di default dalla cascata visiva rod**:
`GRK1,SAG,GNAT1,GNB1,GNGT1,PDE6A,PDE6B,PDE6G,CNGA1,CNGB1,RCVRN,GUCY2D,GUCY2F,GUCA1A`

## Istruzioni per Claude

Analizza gli argomenti: $ARGUMENTS

- Il primo token è il GENE (es. RHO)
- Il secondo token (se presente) è la lista CSV degli interattori da letteratura

Se manca la lista interattori, usa il set default della cascata rod sopra indicato.

Esegui questi passi:

1. **Esegui lo script**:
   ```
   python ppi_analyzer/main.py --gene GENE --literature INTERACTORS
   ```

2. **Presenta il report di confronto** strutturato in tre sezioni:

   ### Interattori Confermati (overlap letteratura ∩ STRING)
   - Lista geni presenti in entrambe le fonti
   - Score STRING e tipo di evidenza
   - Posizione nella cascata: attivazione / recupero / regolazione Ca²⁺

   ### Gap nei Database (solo in letteratura)
   - Interattori della cascata rod non trovati in STRING
   - Per ognuno: perché l'interazione è nota e perché STRING può non averla
     - Es: interazioni transitorie (transducina) difficili da catturare con textmining
     - Es: interazioni tessuto-specifiche (bastoncelli) non rilevate in modelli cellulari generici

   ### Nuove Interazioni da Database (non nella cascata classica)
   - Top 10 per score con evidenza
   - Evidenzia interattori GPCR generici vs specifici dei fotorecettori
   - Commento: potenziali nuovi regolatori del segnale visivo?

3. **Gap Analysis**:
   - Jaccard similarity tra cascata rod classica e STRING
   - Top 3 candidati per validazione (preferendo evidenza sperimentale diretta)
   - Nota: interazioni transitorie della cascata (es. RHO*-transducina) sono per natura difficili da rilevare con Co-IP standard → suggerire crosslinking MS o FRET

4. **Genera export Cytoscape**:
   ```
   python ppi_analyzer/main.py --gene GENE --literature INTERACTORS --cytoscape RHO_network.json
   ```

### Contesto biologico degli interattori default

| Gene | Ruolo nella cascata rod | Step |
|------|------------------------|------|
| GRK1 | Rhodopsin kinase — fosforila RHO* attivata | Recupero |
| SAG | Arrestina-1 — lega RHO*-P, spegne il segnale | Recupero |
| GNAT1 | Transducina α rod — attivata da RHO* | Amplificazione |
| GNB1 | Transducina β — complesso βγ | Amplificazione |
| GNGT1 | Transducina γ rod — complesso βγ | Amplificazione |
| PDE6A/B | Fosfodiesterasi cGMP — idrolizza cGMP | Effettore |
| PDE6G | Subunità inibitoria γ di PDE6 | Regolazione PDE6 |
| CNGA1/B1 | Canale CNG — chiusura per ↓cGMP | Segnale finale |
| RCVRN | Recoverin — inibisce GRK1 in presenza Ca²⁺ | Regolazione Ca²⁺ |
| GUCY2D/F | Guanilato ciclasi — risintesi cGMP | Recupero cGMP |
| GUCA1A | GCAP1 — attiva guanilato ciclasi a basso Ca²⁺ | Regolazione Ca²⁺ |

Nota: distingui sempre tra interazione **transitoria** (RHO*-transducina: millisecondi) e **stabile** (RHO-SAG in condizioni di luce intensa).
