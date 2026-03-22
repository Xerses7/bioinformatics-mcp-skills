# Variant Annotate — RHO (adRP4)

> **Skill calibrata per la rodopsina e la Retinitis Pigmentosa autosomica dominante (adRP, RP4).**
> A differenza di EYS (AR, perdita di funzione biallelica), RHO causa adRP prevalentemente con meccanismo **dominante negativo o gain-of-function**: una sola copia mutata è sufficiente. La classificazione del rischio e la correlazione genotipo-fenotipo riflettono questo.
> Per annotazione varianti generica usa `/variant-annotate GENE`.

Recupera e classifica le varianti di RHO da UniProt, con classificazione per tipo, localizzazione sulla struttura GPCR, meccanismo patologico dominante (misfolding/ER stress, alterazione trafficking, costitutiva attivazione) e correlazione genotipo-fenotipo adRP.

## Come usare
`/rho-variant-annotate GENE_SYMBOL`

Esempi:
- `/rho-variant-annotate RHO`
- `/rho-variant-annotate GRK1`
- `/rho-variant-annotate SAG`

## Istruzioni per Claude

Il gene da analizzare è: $ARGUMENTS

### Step 1: Recupera tutte le varianti da UniProt

Esegui su `https://sparql.uniprot.org/sparql`:

```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>

SELECT ?comment ?begin
WHERE {
  ?protein a up:Protein ;
           up:organism taxon:9606 ;
           up:encodedBy ?gene ;
           up:annotation ?annotation .
  ?gene skos:prefLabel ?geneName .
  ?annotation a up:Natural_Variant_Annotation ;
              rdfs:comment ?comment ;
              up:range ?range .
  ?range faldo:begin ?beginPos .
  ?beginPos faldo:position ?begin .
  FILTER (?geneName = "GENE_SYMBOL")
}
ORDER BY ?begin
```

### Step 2: Recupera la patologia associata

Esegui su `https://sparql.uniprot.org/sparql`:

```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?diseaseComment
WHERE {
  ?protein a up:Protein ;
           up:organism taxon:9606 ;
           up:encodedBy ?gene ;
           up:annotation ?annotation .
  ?gene skos:prefLabel ?geneName .
  ?annotation a up:Disease_Annotation ;
              rdfs:comment ?diseaseComment .
  FILTER (?geneName = "GENE_SYMBOL")
}
```

### Step 3: Classifica le varianti per tipo molecolare

| Tipo | Pattern nel comment | Frequenza in RHO |
|------|--------------------|--------------------|
| **Missense** | "X → Y" | Dominante (~85% delle varianti adRP) |
| **Nonsense** | "→ Stop" | Rara in adRP; causa AR-RP se biallelica |
| **Frameshift** | "frameshift" | Rara |
| **Delezione in-frame** | "del" | Rara |
| **VUS** | "uncertain" | Presente |

### Step 4: Classifica per meccanismo patologico adRP

Per RHO, la patogenesi dipende da **dove** cade la variante sulla struttura:

#### Classe I — Misfolding e ritenzione nel RE
Varianti nella regione N-terminale intradiscale o nel core TM che alterano il folding:
- **Meccanismo**: proteina mal ripiegata → ritenzione nel RE → ER stress → UPR → apoptosi
- **Dominante negativo**: sequestra chaperone (calnexina, BiP) privando la RHO WT delle risorse di folding
- **Esempi**: P23H (più comune in USA/Europa), T17M, G51V, A164V, G188R
- **Fenotipo**: grave, esordio precoce (infanzia/adolescenza), perdita rapida

```
Regioni a rischio Classe I:
- aa 1-33 (N-terminus intradiscale, contiene N2/N15)
- Loop extracellulari ECL1/2/3
- Core idrofobico TM (specialmente interfacce TM2-TM3-TM4)
```

#### Classe II — Alterazione del trafficking verso i segmenti esterni
Varianti nella regione C-terminale che disturbano il segnale di sorting verso OS:
- **Meccanismo**: proteina correttamente piegata ma non trasportata verso i segmenti esterni → accumulo nel corpo cellulare → tossicità
- **Il segnale VXPX** (aa 334-347) è essenziale per il trasporto mediato da ARF4/ASAP1
- **Esempi**: P347L, P347S, Q344ter, V345M
- **Fenotipo**: moderato-grave, variabile

```
Regione C-terminale a rischio Classe II:
- aa 330-348 (VXPX motif a aa 344-347)
- Vicino ai siti di palmitoilazione C322/C323
```

#### Classe III — Alterazione della fototrasduzione / Costitutiva attivazione
Varianti che modificano il sito di legame del retinale o l'accoppiamento con la transducina:
- **Meccanismo**: attivazione costitutiva (segnale al buio) → esaurimento del sistema di recupero → tossicità per segnalazione cronica
- **Esempi**: K296E, K296M (legame retinale), E113Q (controione)
- **Fenotipo**: variabile, spesso con adattamento al buio compromesso

#### Classe IV — Varianti AR (perdita di funzione biallelica)
Rare varianti che causano RP autosomica recessiva se in omozigosi:
- **Esempi**: alcuni nonsense in omozigosi in popolazioni consanguinee
- **Fenotipo**: simile a adRP ma richiede due alleli mutati

### Step 5: Mappa delle varianti sulla struttura TM

```
RHO (348 aa):

N-term   TM1   TM2   TM3   TM4   TM5   TM6   TM7   C-term
[N2,N15] |─────|─────|─────|─────|─────|─────|─────| [C322,C323,pS/T]
  ↑glc   34    70   105   147   199   246   285  309   330-348
                           ↑C110            K296↑      ↑VXPX
                           C110-C187 S-S    retinale   trafficking

Distribuzione varianti:
Classe I (misfolding):  ●●● [N-term+TM core]  →  P23H(23), T17M(17)...
Classe II (trafficking): ●● [C-term]           →  P347L(347), Q344ter...
Classe III (attivazione): ●  [TM7/K296]        →  K296E, K296M...
```

### Step 6: Report strutturato

#### A. Riepilogo statistico
```
Gene:             RHO (RP4 — autosomica dominante)
Varianti totali:  N
  Missense:       N (XX%) — principale causa adRP
  Nonsense:       N (XX%)
  Frameshift/del: N (XX%)
  VUS:            N (XX%)
```

#### B. Varianti per Classe Patogenetica
Tabella: posizione | variante | classe (I/II/III/IV) | meccanismo | frequenza nota | fenotipo atteso

#### C. Hot-spot adRP
- Cluster N-terminale (aa 1-33): varianti di Classe I
- C-terminus (aa 330-348): varianti di Classe II
- K296 e E113: Classe III

#### D. Varianti a Massima Priorità Clinica
P23H, T17M, P347L/S: le più frequenti in popolazione europea/nordamericana — richiedono counseling genetico adRP specifico.

#### E. Implicazioni per Terapia Genica
Nota importante per adRP: la terapia genica con sostituzione genica semplice (aggiunta di RHO WT) **non è sufficiente** se la proteina mutante esercita dominante negativo. Approcci terapeutici in studio:
- **Knockdown + replacement**: silenziamento allele-specifico (RNAi, ASO) + RHO WT codon-modified
- **Small molecules**: chaperoni farmacologici per Classe I (9-cis retinale, VRX-001)
- **Gene editing**: CRISPR allele-specifico per varianti dominanti comuni (P23H)

### Note cliniche adRP-RHO
- RHO è responsabile del ~25% di tutti i casi di adRP in Europa e Nord America
- P23H da sola rappresenta ~12% di adRP negli USA (fondatore europeo)
- Esordio tipico: prima-seconda decade; nictalopia (cecità notturna) come primo sintomo
- Progressione: rod → cone dystrophy; campo visivo a tunnel → cecità legale
- Per analisi PTM impattate: `/rho-ptm-analyze RHO`
- Per analisi cascata fototransduction: `/rho-ppi-compare RHO`
