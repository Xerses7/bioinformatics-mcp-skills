# PTM Analyze — RHO (Rodopsina)

> **Skill ottimizzata per la rodopsina e le sue modificazioni post-traduzionali funzionali.**
> Interpreta le PTM nel contesto della struttura GPCR a 7 eliche transmembrana: N-glicosilazione intradiscale (N2/N15), ponte disolfuro (C110-C187), palmitoilazione C-terminale (C322/C323), fosforilazione GRK1-mediata (C-terminale), legame covalente con il retinale (K296).
> Per analisi PTM generica usa `/ptm-analyze GENE`.

Recupera e analizza tutte le PTM della rodopsina da UniProt, con mappa sulla struttura TM e interpretazione funzionale per ogni tipo di modifica nel ciclo visivo.

## Come usare
`/rho-ptm-analyze GENE_SYMBOL`

Esempi:
- `/rho-ptm-analyze RHO`
- `/rho-ptm-analyze GRK1`
- `/rho-ptm-analyze SAG`

## Istruzioni per Claude

Il gene da analizzare è: $ARGUMENTS

### Step 1: Recupera tutti i siti PTM da UniProt via SPARQL

Esegui su `https://sparql.uniprot.org/sparql`:

```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>

SELECT ?annotType ?comment ?begin ?end
WHERE {
  ?protein a up:Protein ;
           up:organism taxon:9606 ;
           up:encodedBy ?gene ;
           up:annotation ?annotation .
  ?gene skos:prefLabel ?geneName .
  ?annotation a ?annotType ;
              rdfs:comment ?comment ;
              up:range ?range .
  ?range faldo:begin ?beginPos ;
         faldo:end ?endPos .
  ?beginPos faldo:position ?begin .
  ?endPos faldo:position ?end .
  FILTER (?geneName = "GENE_SYMBOL")
  FILTER (?annotType IN (
    up:Glycosylation_Annotation,
    up:Modified_Residue_Annotation,
    up:Lipidation_Annotation,
    up:Disulfide_Bond_Annotation,
    up:Cross_Link_Annotation,
    up:PTM_Annotation
  ))
}
ORDER BY ?begin
```

### Step 2: Raggruppa per tipo e confronta con PTM attese per RHO

Per il gene RHO, le PTM attese dalla letteratura sono:

| PTM | Posizione | Tipo UniProt | Funzione |
|-----|-----------|-------------|---------|
| N-glicosilazione | N2 | Glycosylation_Annotation | Folding nel RE, stabilità intradiscale |
| N-glicosilazione | N15 | Glycosylation_Annotation | Folding nel RE, stabilità intradiscale |
| Ponte disolfuro | C110-C187 | Disulfide_Bond_Annotation | Stabilizza loop extracellulare 2 (ECL2) |
| Palmitoilazione | C322 | Lipidation_Annotation | Ancoraggio C-terminale alla membrana discale |
| Palmitoilazione | C323 | Lipidation_Annotation | Ancoraggio C-terminale alla membrana discale |
| Retinal (Schiff base) | K296 | Cross_Link_Annotation | Legame covalente con 11-cis retinale (cromoforo) |
| Fosforilazione | S334, S338, S343 | Modified_Residue_Annotation | GRK1-mediata, necessaria per binding arrestina |
| Fosforilazione | T336, T340 | Modified_Residue_Annotation | GRK1-mediata (siti aggiuntivi) |

Per altri geni del pathway (GRK1, SAG, GNAT1…), recupera le PTM da UniProt senza aspettative predefinite.

### Step 3: Mappa sulla struttura di RHO

Costruisci la mappa della proteina con i domini TM e le PTM:

```
RHO (348 aa) — struttura GPCR 7-TM:

Intradiscale (N-terminus):
[N2-glc][N15-glc]──────────────────── aa 1-33
                                              ↕ C110-C187 (S-S)
TM1    TM2    TM3    TM4    TM5    TM6    TM7
|──────|──────|──────|──────|──────|──────|──────|
34     70    105    147    199    246    285    309

                            K296 ← retinale (TM7)

C-terminus (citoplasmatico):
─────── [pS334][pT336][pS338][pT340][pS343] ─── [C322-palm][C323-palm]
        ↑ fosforilazione GRK1                    ↑ palmitoilazione
        necessaria per SAG binding
```

### Step 4: Analisi biologica delle PTM di RHO

Spiega ciascun tipo di PTM nel contesto del ciclo visivo:

1. **N-glicosilazione a N2 e N15** (intradiscale, RE-mediata):
   - Avviene nel RE durante la biosintesi
   - Necessaria per il corretto folding e la stabilità della proteina
   - Varianti P23H e T17M (adRP comuni) alterano queste regioni → mancato folding → ritenzione nel RE → UPR → morte cellulare
   - Tipo: N-linked (sequon N-X-T: N2-A-T4 e N15-I-T17)

2. **Ponte disolfuro C110-C187** (loop extracellulare 2):
   - Conservato in tutti i GPCR di classe A
   - Stabilizza ECL2, che copre il sito di legame del retinale
   - Varianti che rompono questo ponte → instabilità strutturale → adRP

3. **Palmitoilazione C322/C323** (C-terminus):
   - Ancora il C-terminale alla membrana discale → crea un 4° loop citoplasmatico
   - Influenza l'accoppiamento con la transducina
   - Palmitoilazione dinamica: può essere rimossa e riformata

4. **Legame con 11-cis retinale a K296** (TM7):
   - Legame Schiff base covalente (reversibile dalla luce)
   - 11-cis retinale → rodopsina (stato scuro, inattivo)
   - Fotoisomerizzazione → tutto-trans retinale → metarodopsina II (RHO*, attivo)
   - K296M (variante adRP) → rhodopsina costitutivamente attiva → tossicità

5. **Fosforilazione C-terminale** (S334, T336, S338, T340, S343):
   - Catalizzata da GRK1 (rhodopsin kinase) solo su RHO* attivata
   - Multisito: 1-2 fosforilazioni → parziale inibizione; 6-7 → blocco completo
   - Binding ad alta affinità di SAG (arrestina) richiede ≥3 fosforilazioni
   - Fosforilazione → internalizzazione → riciclo o degradazione

### Step 5: Considerazioni patologiche adRP

- **Varianti che colpiscono N-glicosilazione** (N2/N15, P23H, T17M): mancato folding → ER stress → UPR → apoptosi. Meccanismo dominante negativo: proteina mutante sequestra HSP90/calnexina depauperando le risorse di folding per RHO WT
- **Varianti che colpiscono ECL2/C110-C187**: destabilizzazione strutturale
- **Varianti C-terminali** (P347L/S, vicino C322/C323): alterano trafficking verso i segmenti esterni
- **Varianti in K296**: costitutive (K296E/M) → segnalazione al buio → rumore di fondo → degenerazione

Proponi: "Per analisi impatto varianti sui siti PTM: `/ptm-variant-impact RHO`"
Per le varianti specifiche adRP: `/rho-variant-annotate RHO`"

### Note tecniche
- UniProt P08100 (RHO_HUMAN) è una entry Swiss-Prot reviewed con annotazioni estensive
- Il legame retinale (K296) è classificato come `Cross_Link_Annotation` in UniProt
- La palmitoilazione è dinamica: UniProt riporta il sito, non lo stato di palmitoilazione in vivo
- Per la fosforilazione GRK1-mediata: è attività-dipendente (solo RHO* attivata viene fosforilata)
