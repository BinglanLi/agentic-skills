# AOP-DB Database Schema Reference

Complete schema documentation for AOP-DB v2 MySQL database.

## Table of Contents
- [AOP Tables](#aop-tables)
- [Gene Tables](#gene-tables)
- [Chemical/Stressor Tables](#chemicalstressor-tables)
- [Pathway Tables](#pathway-tables)
- [Disease Tables](#disease-tables)
- [Assay Tables](#assay-tables)
- [Event Tables](#event-tables)
- [Relationship Tables](#relationship-tables)

---

## AOP Tables

### aop_info
Primary table containing AOP metadata from AOP-Wiki.

| Column | Type | Description |
|--------|------|-------------|
| `AOP_id` | INT | AOP-Wiki identifier (primary key) |
| `AOP_name` | VARCHAR(500) | Full AOP name/title |
| `AOP_description` | TEXT | Detailed AOP description |
| `short_name` | VARCHAR(255) | Abbreviated name |
| `authors` | TEXT | Contributing authors |
| `status` | VARCHAR(50) | Development status (e.g., "Under Development", "Endorsed") |
| `oecd_status` | VARCHAR(50) | OECD endorsement status |
| `saaop_status` | VARCHAR(50) | SAAOP status |
| `abstract` | TEXT | AOP abstract/summary |
| `background` | TEXT | Scientific background |
| `created` | DATE | Creation date |
| `last_modified` | DATE | Last modification date |

**Example query:**
```sql
SELECT AOP_id, AOP_name, status FROM aop_info WHERE status = 'Endorsed';
```

### aop_ke
Links AOPs to their constituent Key Events.

| Column | Type | Description |
|--------|------|-------------|
| `AOP_id` | INT | Foreign key to aop_info |
| `event_id` | INT | Foreign key to event_info |
| `ke_order` | INT | Order of KE in the AOP |
| `ke_type` | VARCHAR(50) | Type: "MIE", "KE", or "AO" |

---

## Gene Tables

### gene_info
Central gene information table linking all gene-related data.

| Column | Type | Description |
|--------|------|-------------|
| `entrez` | INT | NCBI Entrez Gene ID (primary key) |
| `gene_symbol` | VARCHAR(50) | Official gene symbol |
| `gene_name` | VARCHAR(500) | Full gene name |
| `tax_id` | INT | NCBI Taxonomy ID (9606 = human) |
| `chromosome` | VARCHAR(10) | Chromosomal location |
| `gene_type` | VARCHAR(50) | Gene type (protein-coding, ncRNA, etc.) |
| `description` | TEXT | Gene description |
| `aliases` | TEXT | Alternative gene symbols |

**Example query:**
```sql
SELECT entrez, gene_symbol, gene_name FROM gene_info WHERE tax_id = 9606 LIMIT 10;
```

### aop_gene
Links AOPs to their associated genes.

| Column | Type | Description |
|--------|------|-------------|
| `AOP_id` | INT | Foreign key to aop_info |
| `entrez` | INT | Foreign key to gene_info |
| `gene_symbol` | VARCHAR(50) | Gene symbol |
| `source` | VARCHAR(100) | Data source for association |

### ke_gene
Links Key Events to genes.

| Column | Type | Description |
|--------|------|-------------|
| `event_id` | INT | Foreign key to event_info |
| `entrez` | INT | Foreign key to gene_info |
| `gene_symbol` | VARCHAR(50) | Gene symbol |
| `source` | VARCHAR(100) | Evidence source |

### gene_ortholog
Cross-species gene orthology from Homologene.

| Column | Type | Description |
|--------|------|-------------|
| `homologene_id` | INT | Homologene cluster ID |
| `entrez` | INT | Entrez Gene ID |
| `tax_id` | INT | Taxonomy ID |
| `gene_symbol` | VARCHAR(50) | Gene symbol |

---

## Chemical/Stressor Tables

### chemical_info
Chemical/compound information linked to DSSTox.

| Column | Type | Description |
|--------|------|-------------|
| `ChemicalID` | VARCHAR(20) | MeSH chemical ID |
| `DTX_id` | VARCHAR(20) | DSSTox Substance ID (DTXSID) |
| `chemical_name` | VARCHAR(500) | Chemical name |
| `CASRN` | VARCHAR(20) | CAS Registry Number |
| `InChIKey` | VARCHAR(50) | InChI Key |
| `SMILES` | TEXT | SMILES structure |
| `molecular_formula` | VARCHAR(100) | Molecular formula |
| `molecular_weight` | DECIMAL(10,4) | Molecular weight |

**Example query:**
```sql
SELECT DTX_id, chemical_name, CASRN FROM chemical_info WHERE chemical_name LIKE '%phenol%';
```

### stressor_info
Stressor information extracted from AOP-Wiki.

| Column | Type | Description |
|--------|------|-------------|
| `stressor_id` | INT | AOP-Wiki stressor ID (primary key) |
| `stressor_name` | VARCHAR(500) | Stressor name |
| `user_term` | VARCHAR(500) | User-defined term |
| `CASRN` | VARCHAR(20) | CAS Registry Number |
| `DTXSID` | VARCHAR(20) | DSSTox ID |
| `description` | TEXT | Stressor description |
| `stressor_type` | VARCHAR(50) | Type (chemical, biological, physical) |

### aop_stressor
Links AOPs to their associated stressors.

| Column | Type | Description |
|--------|------|-------------|
| `AOP_id` | INT | Foreign key to aop_info |
| `stressor_id` | INT | Foreign key to stressor_info |
| `DTXSID` | VARCHAR(20) | DSSTox ID (if mapped) |
| `evidence` | VARCHAR(100) | Evidence level |

### chemical_gene
Chemical-gene interactions from CTD.

| Column | Type | Description |
|--------|------|-------------|
| `ChemicalID` | VARCHAR(20) | MeSH chemical ID |
| `entrez` | INT | Entrez Gene ID |
| `gene_symbol` | VARCHAR(50) | Gene symbol |
| `interaction` | VARCHAR(500) | Interaction description |
| `interaction_type` | VARCHAR(100) | Type of interaction |
| `pubmed_ids` | TEXT | Supporting PubMed IDs |

---

## Pathway Tables

### pathway_gene
Gene-pathway associations from multiple sources.

| Column | Type | Description |
|--------|------|-------------|
| `path_id` | VARCHAR(50) | Pathway identifier |
| `path_name` | VARCHAR(500) | Pathway name |
| `entrez` | INT | Entrez Gene ID |
| `gene_symbol` | VARCHAR(50) | Gene symbol |
| `tax_id` | INT | Taxonomy ID |
| `ext_source` | VARCHAR(50) | Source database (KEGG, Reactome, ConsensusPathDB) |

**Example query (human pathways only):**
```sql
SELECT DISTINCT path_id, path_name, ext_source
FROM pathway_gene
WHERE tax_id = 9606
ORDER BY path_name;
```

**Cleaning pathway names:**
```sql
SELECT DISTINCT
    TRIM(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
        path_name, '<sub>', ''), '</sub>', ''),
        '<i>', ''), '</i>', ''),
        ' - Homo sapiens (human)', '')) as clean_name
FROM pathway_gene
WHERE tax_id = 9606;
```

### pathway_info
Pathway metadata (when available).

| Column | Type | Description |
|--------|------|-------------|
| `path_id` | VARCHAR(50) | Pathway identifier (primary key) |
| `path_name` | VARCHAR(500) | Pathway name |
| `source` | VARCHAR(50) | Source database |
| `description` | TEXT | Pathway description |
| `url` | VARCHAR(500) | External URL |

---

## Disease Tables

### disease_gene
Disease-gene associations from DisGeNET.

| Column | Type | Description |
|--------|------|-------------|
| `disease_id` | VARCHAR(20) | Disease identifier (UMLS CUI) |
| `disease_name` | VARCHAR(500) | Disease name |
| `entrez` | INT | Entrez Gene ID |
| `gene_symbol` | VARCHAR(50) | Gene symbol |
| `score` | DECIMAL(5,4) | Association score |
| `source` | VARCHAR(100) | Evidence source |
| `pmid` | TEXT | Supporting PubMed IDs |

### disease_info
Disease metadata.

| Column | Type | Description |
|--------|------|-------------|
| `disease_id` | VARCHAR(20) | Disease identifier (primary key) |
| `disease_name` | VARCHAR(500) | Disease name |
| `disease_class` | VARCHAR(100) | Disease classification |
| `semantic_type` | VARCHAR(100) | UMLS semantic type |

---

## Assay Tables

### assay_gene
ToxCast assay-gene target mappings.

| Column | Type | Description |
|--------|------|-------------|
| `assay_id` | VARCHAR(50) | ToxCast assay ID |
| `assay_name` | VARCHAR(500) | Assay name |
| `entrez` | INT | Target Entrez Gene ID |
| `gene_symbol` | VARCHAR(50) | Target gene symbol |
| `assay_endpoint` | VARCHAR(200) | Assay endpoint |
| `intended_target` | VARCHAR(200) | Intended molecular target |

### assay_info
ToxCast assay metadata.

| Column | Type | Description |
|--------|------|-------------|
| `assay_id` | VARCHAR(50) | Assay identifier (primary key) |
| `assay_name` | VARCHAR(500) | Full assay name |
| `assay_source` | VARCHAR(100) | Assay source |
| `assay_design` | VARCHAR(100) | Design type |
| `biological_process` | VARCHAR(200) | Target biological process |
| `cell_format` | VARCHAR(100) | Cell format |
| `tissue` | VARCHAR(100) | Target tissue |

---

## Event Tables

### event_info
Key Event (KE) information from AOP-Wiki.

| Column | Type | Description |
|--------|------|-------------|
| `event_id` | INT | KE identifier (primary key) |
| `event_name` | VARCHAR(500) | KE name/title |
| `short_name` | VARCHAR(255) | Short name |
| `ke_type` | VARCHAR(50) | Type: "MIE", "KE", or "AO" |
| `biological_level` | VARCHAR(100) | Level of biological organization |
| `description` | TEXT | Full description |
| `how_measured` | TEXT | Measurement methods |
| `evidence_supporting` | TEXT | Supporting evidence |

### ker_info
Key Event Relationships (KER) information.

| Column | Type | Description |
|--------|------|-------------|
| `ker_id` | INT | KER identifier (primary key) |
| `upstream_event_id` | INT | Upstream KE ID |
| `downstream_event_id` | INT | Downstream KE ID |
| `relationship_name` | VARCHAR(500) | Relationship description |
| `weight_of_evidence` | VARCHAR(100) | Evidence weight |
| `quantitative_understanding` | VARCHAR(100) | Quantitative understanding level |

---

## Relationship Tables

### aop_ke_relationship
Detailed AOP-KE relationships.

| Column | Type | Description |
|--------|------|-------------|
| `AOP_id` | INT | AOP identifier |
| `upstream_event_id` | INT | Upstream KE ID |
| `downstream_event_id` | INT | Downstream KE ID |
| `ker_id` | INT | KER identifier |
| `adjacency` | VARCHAR(50) | Adjacent or non-adjacent |

### gene_interaction
Gene-gene interaction data.

| Column | Type | Description |
|--------|------|-------------|
| `entrez_a` | INT | First gene Entrez ID |
| `entrez_b` | INT | Second gene Entrez ID |
| `gene_symbol_a` | VARCHAR(50) | First gene symbol |
| `gene_symbol_b` | VARCHAR(50) | Second gene symbol |
| `interaction_type` | VARCHAR(100) | Interaction type |
| `source` | VARCHAR(100) | Data source |
| `pubmed_ids` | TEXT | Supporting PubMed IDs |

---

## Common Query Patterns

### Get all genes for an AOP
```sql
SELECT DISTINCT g.entrez, g.gene_symbol, g.gene_name
FROM aop_gene ag
JOIN gene_info g ON ag.entrez = g.entrez
WHERE ag.AOP_id = 42;
```

### Get AOPs for a gene
```sql
SELECT a.AOP_id, a.AOP_name
FROM aop_info a
JOIN aop_gene ag ON a.AOP_id = ag.AOP_id
WHERE ag.gene_symbol = 'TP53';
```

### Get chemicals affecting an AOP
```sql
SELECT DISTINCT c.DTX_id, c.chemical_name, c.CASRN
FROM aop_stressor as_tbl
JOIN stressor_info si ON as_tbl.stressor_id = si.stressor_id
JOIN chemical_info c ON si.DTXSID = c.DTX_id
WHERE as_tbl.AOP_id = 42;
```

### Get pathways for a gene
```sql
SELECT DISTINCT path_id, path_name, ext_source
FROM pathway_gene
WHERE entrez = 7157 AND tax_id = 9606;
```

### Count records per table
```sql
SELECT 'aop_info' as table_name, COUNT(*) as row_count FROM aop_info
UNION ALL SELECT 'gene_info', COUNT(*) FROM gene_info
UNION ALL SELECT 'pathway_gene', COUNT(*) FROM pathway_gene
UNION ALL SELECT 'chemical_info', COUNT(*) FROM chemical_info
UNION ALL SELECT 'stressor_info', COUNT(*) FROM stressor_info;
```

---

## Notes

1. **Tax ID 9606** = Homo sapiens (human). Use this filter for human-specific queries.
2. **Pathway names** may contain HTML tags (`<sub>`, `<i>`) - clean with REPLACE.
3. **DTXSID** is the primary chemical identifier linking to EPA DSSTox.
4. **Entrez Gene ID** is the central gene identifier linking all gene tables.
5. Some tables may have additional columns in newer versions - use `DESCRIBE table_name` to verify.
