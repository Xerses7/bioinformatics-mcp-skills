# Gene Expression — RHO (Fototransduction)

> **Skill ottimizzata per RHO e i geni della cascata di fototransduction rod.**
> L'analisi biologica interpreta l'espressione nel contesto della biologia dei bastoncelli: segmenti esterni (OS), ciglio di connessione, corpo cellulare, sinapsi. Focus su adRP (RP4) e ciclo visivo rod.
> Per profilo di espressione su geni retinici generici usa `/eys-gene-expression GENE`.
> Per profilo di espressione su qualsiasi gene usa `/gene-expression GENE`.

Recupera il profilo di espressione rod-specifico, la localizzazione subcellulare nei bastoncelli e la partecipazione ai pathway della fototransduction per RHO e i geni della cascata visiva.

## Come usare
`/rho-gene-expression GENE_SYMBOL`

Esempi:
- `/rho-gene-expression RHO`
- `/rho-gene-expression GRK1`
- `/rho-gene-expression SAG`
- `/rho-gene-expression GNAT1`
- `/rho-gene-expression PDE6B`

## Istruzioni per Claude

Il gene da analizzare è: $ARGUMENTS

### Step 1: Recupera espressione tessutale da UniProt

Esegui su `https://sparql.uniprot.org/sparql`:

```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?comment
WHERE {
  ?protein a up:Protein ;
           up:organism taxon:9606 ;
           up:encodedBy ?gene ;
           up:annotation ?annotation .
  ?gene skos:prefLabel ?geneName .
  ?annotation a up:Tissue_Specificity_Annotation ;
              rdfs:comment ?comment .
  FILTER (?geneName = "GENE_SYMBOL")
}
```

### Step 2: Recupera localizzazione subcellulare da UniProt

Esegui su `https://sparql.uniprot.org/sparql`:

```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?locComment
WHERE {
  ?protein a up:Protein ;
           up:organism taxon:9606 ;
           up:encodedBy ?gene ;
           up:annotation ?annotation .
  ?gene skos:prefLabel ?geneName .
  ?annotation a up:Subcellular_Location_Annotation ;
              rdfs:comment ?locComment .
  FILTER (?geneName = "GENE_SYMBOL")
}
```

### Step 3: Recupera pathway da WikiPathways (focus fototransduction)

Esegui su `https://sparql.wikipathways.org/sparql`:

```sparql
PREFIX wp: <http://vocabularies.wikipathways.org/wp#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?wpid ?pathwayTitle
WHERE {
  ?geneProduct a wp:GeneProduct ;
               rdfs:label ?geneLabel ;
               dcterms:isPartOf ?pathway .
  ?pathway a wp:Pathway ;
           dc:title ?pathwayTitle ;
           dcterms:identifier ?wpid ;
           wp:organismName "Homo sapiens" .
  FILTER (UCASE(?geneLabel) = UCASE("GENE_SYMBOL"))
}
ORDER BY ?pathwayTitle
```

### Step 4: Mappa dell'espressione nel bastoncello

Costruisci la mappa anatomica del bastoncello con la localizzazione del gene:

```
BASTONCELLO — Compartimenti e geni principali:

  ┌──────────────────────────────┐
  │  SEGMENTO ESTERNO (OS)       │  ← RHO (dischi), PDE6A/B (dischi),
  │  Dischi membranosi           │     CNGA1/B1 (membrana plasmatica OS)
  │  Membrana plasmatica OS      │
  └──────────────┬───────────────┘
                 │ Ciglio di connessione
  ┌──────────────┴───────────────┐  ← RPGR, CEP290, CNGB1
  │  SEGMENTO INTERNO (IS)       │
  │  Ellissoide (mitocondri)     │  ← metabolismo energetico
  │  Mioide (ribosomi, RE, Golgi)│  ← biosintesi RHO, GRK1, SAG, GNAT1...
  └──────────────┬───────────────┘
                 │ Nucleo
  ┌──────────────┴───────────────┐
  │  CORPO CELLULARE             │
  │  Nucleo                      │
  └──────────────┬───────────────┘
                 │ Assone
  ┌──────────────┴───────────────┐
  │  SINAPSI (Sferula)           │  ← trasmissione segnale a cellule bipolari
  └──────────────────────────────┘
```

### Step 5: Profilo di espressione per i geni della cascata rod

| Gene | Localizzazione rod | Rod/Cone | Espressione extra-retinica |
|------|-------------------|----------|--------------------------|
| RHO | Dischi OS (rod) | Rod-specifica | Praticamente assente altrove |
| GRK1 | IS + OS | Rod >> Cone (nei mammiferi) | Pineale (ritmo circadiano) |
| SAG | IS (citoplasma) | Rod >> Cone | Limitata |
| GNAT1 | OS (dischi + membrana) | Rod-specifica | Assente altrove |
| GNB1 | OS | Rod (+ espressione ubiquitaria) | Cervello, altri tessuti |
| PDE6A | OS (dischi) | Rod-specifica | Assente altrove |
| PDE6B | OS (dischi) | Rod-specifica | Assente altrove |
| CNGA1 | OS (membrana plasmatica) | Rod-specifica | Assente altrove |
| RCVRN | IS + OS | Rod >> Cone | Limitata |
| GUCY2F | OS (membrana) | Rod-specifica | Limitata |

Posiziona il gene richiesto in questa tabella e verifica la coerenza con i dati UniProt.

### Step 6: Report strutturato

#### A. Profilo di Espressione Tessutale
- Espressione sistemica (da UniProt)
- **Specificità per bastoncelli vs coni**: il gene è rod-specifico o espresso in entrambi i tipi?
- Espressione extra-retinica se presente (es. GRK1 nella ghiandola pineale)
- Onset dello sviluppo: quando inizia l'espressione nella retina in via di sviluppo?

#### B. Localizzazione Subcellulare nel Bastoncello
Mappa il gene sul compartimento del bastoncello:
- **OS (Outer Segment)**: molecole della trasduzione del segnale — RHO, transducina, PDE6, CNGA1
- **IS (Inner Segment)**: biosintesi, chaperone, recupero — GRK1, SAG, RCVRN
- **Ciglio di connessione**: trasporto IFT verso OS
- **Sinapsi**: rilascio di glutammato, RIBEYE

Comportamento dinamico: alcune proteine (GRK1, SAG, GNAT1) si **traslocano** tra IS e OS in risposta alla luce — indicare se questo è noto per il gene richiesto.

#### C. Pathway (WikiPathways) con Focus Fototransduction
Evidenzia in particolare:
- **WP2485**: Visual Transduction — il pathway centrale
- **WP4239**: Retinal degeneration pathways
- Pathway GPCR generici (evidenzia che RHO è un GPCR ma agisce in contesto rod-specifico)
- Pathway del visual cycle (11-cis retinale, ABCA4, RPE65)

#### D. Pattern di Degenerazione atteso per adRP-RHO
Nell'adRP da RHO:
- **Perdita rod predominante**: nictalopia (cecità notturna) come primo sintomo
- **Progressione**: rod → cone dystrophy (perdita campo periferico → centrale)
- **Rod-cone vs cone-rod**: adRP-RHO è tipicamente **rod-cone** dystrophy
- **Esordio**: prima-seconda decade (nictalopia), perdita campo visivo in terza-quarta decade
- **Fattore modificante**: livello di espressione dell'allele WT può influenzare la severità (eterozigosi funzionale)

#### E. Confronto con Geni RP Correlati per Localizzazione
```
RHO (adRP):   OS dischi → rod-cone dystrophy, esordio precoce
EYS (AR-RP):  IPM + OS → rod-cone dystrophy, esordio variabile
RPGR (X-RP):  Ciglio → rod-cone/cone-rod, grave nei maschi
PRPF31 (adRP): Nucleo → espressione variabile (penetranza incompleta)
ABCA4 (AR):   OS dischi → cone-rod dystrophy (Stargardt + RP)
```

### Note specifiche su RHO
- Espressione **esclusivamente rod-specifica**: RHO è la molecola più abbondante nei bastoncelli (~108 molecole/cella, copre ~50% delle proteine OS)
- Sintesi: avviene nel RE del segmento interno → vesicole di trasporto → ciglio → OS
- **Turnover dei dischi**: i dischi OS vengono continuamente rinnovati (apicale → basale in ~10 giorni) → RHO viene costantemente sintetizzata e fagocitata dall'RPE
- Mutazioni che alterano il trafficking C-terminale (P347L) interrompono questo ciclo → accumulo nel IS → tossicità
- Per analisi varianti: `/rho-variant-annotate RHO`
- Per analisi PTM nel ciclo di segnalazione: `/rho-ptm-analyze RHO`
