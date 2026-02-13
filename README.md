# Datasets-for-AdaHD
Datasets and Preprocessing Steps for “Sparsity-Aware Temporal Knowledge Graph Reasoning via Adaptive Time-Encoding and Historical Distillation”
# Dataset Description

## 1. Dataset Sources

This project uses six widely adopted temporal knowledge graph benchmarks:

- **ICEWS14**: Derived from the Integrated Crisis Early Warning System, archiving global political events
- **ICEWS18**: Also derived from the Integrated Crisis Early Warning System, covering global political events across a different time span
- **ICEWS05-15**: Derived from the Integrated Crisis Early Warning System (Boschee et al., 2015), covering global political events from 2005 to 2015
- **WIKI**: Derived from Wikipedia, containing temporal facts with validity times (Leblay and Chekol, 2018)
- **YAGO**: Derived from multilingual Wikipedias, providing a knowledge base with temporal information (Mahdisoltani et al., 2014)
- **GDELT**: Derived from the Global Database of Events, Language, and Tone project (Leetaru and Schrodt, 2013), covering events from January 1 to 15, 2018

## 2. Dataset Details

### Dataset Statistics

| Dataset       | Entities | Relations | Training    | Validation | Test      | Time Granularity | Snapshots |
|---------------|----------|-----------|-------------|------------|-----------|------------------|-----------|
| ICEWS14       | 7,128    | 230       | 74,845      | 8,514      | 7,371     | 24 hours         | 365       |
| ICEWS18       | 23,033   | 256       | 373,018     | 45,995     | 49,545    | 24 hours         | 365       |
| ICEWS05-15    | 10,488   | 251       | 368,868     | 46,302     | 46,159    | 24 hours         | 4017      |
| WIKI          | 12,554   | 24        | 539,286     | 67,538     | 63,110    | 1 year           | --        |
| YAGO          | 10,623   | 10        | 161,540     | 19,523     | 20,026    | 1 year           | --        |
| GDELT         | 7,691    | 240       | 1,734,399   | 238,765    | 305,241   | 15 minutes       | 2975      |

### Data Format

All datasets follow the same format:

- **Training/Validation/Test Files**: Each line contains a quadruple `(head entity ID, relation ID, tail entity ID, timestamp)`
- **Entity and Relation Mappings**: Each dataset includes `entity2id.txt` and `relation2id.txt` files, mapping entities and relations to unique IDs
- **Statistics**: Each dataset includes a `stat.txt` file, recording the total number of entities and relations

## 3. Preprocessing Steps

### 3.1 Entity-Word Mapping Preprocessing (`ICEWS05-15/ent2word.py`)

This script processes entity names and builds mapping relationships between entities and words:

1. **Load entity and relation mappings**: Load mappings from `entity2id.txt` and `relation2id.txt` files
2. **Process entity names**:
   - Extract words from entity names containing parentheses (e.g., "China(Country)" is extracted into "China" and "Country")
   - Count the number of processed entities
3. **Build word vocabulary**: Create a set of all extracted words
4. **Generate word ID mappings**: Map each word to a unique ID and save to `word2id.txt`
5. **Build entity-word relationship graph**:
   - Create relationship triples between each entity and its words
   - Relationship types include:
     - 0: isA relationship (entity and its type)
     - 1: affiliation relationship (entity and its category)
     - 2: direct mapping (entity name itself)
   - Generate `e-w-graph.txt` file containing entity-word relationships

### 3.2 Historical Subgraph Sampling Preprocessing (`get_his_subg.py`)

This script samples subgraphs from historical temporal data to provide historical context information for the model:

1. **Load data**: Use the `load_quadruples` function to load training data and timestamps
2. **Get statistics**: Use the `get_total_number` function to get the total number of entities and relations
3. **Split data by time**: Use the `split_by_time` function to split data into snapshot sequences by timestamp
4. **Initialize dictionaries and directories**:
   - Create `sr_to_sro` and `s_to_sro` dictionaries to store context information of entities and relations
   - Create directory structure for saving subgraphs
5. **Historical subgraph sampling**:
   - For each time step, get historical data
   - Use the `update_dict` function to update context dictionaries
   - Use the `get_sample_from_history_graph` function to sample historical subgraphs:
     - Generate inverse triples to handle reverse relationships
     - Calculate triple frequencies
     - Perform second-order neighbor sampling
     - Save forward and inverse subgraphs
6. **Save results**:
   - Save sampled subgraphs to `his_graph_for/` and `his_graph_inv/` directories
   - Save context dictionaries to `his_dict/` directory

### 3.3 Data Splitting Strategy

To ensure fair comparison with baseline models, the data splitting strategy adopted in previous studies is followed:
- ICEWS14 and ICEWS05-15 are divided into training, validation, and test sets with a proportion of 80%, 10%, and 10% by timestamps
- Other datasets are divided into training, validation, and test sets with a quantity ratio of 8:1:1

## 4. Directory Structure

```
data/
├── ICEWS14/
│   ├── train.txt
│   ├── valid.txt
│   ├── test.txt
│   ├── entity2id.txt
│   ├── relation2id.txt
│   ├── stat.txt
│   ├── his_graph_for/  # Sampled forward historical subgraphs
│   ├── his_graph_inv/  # Sampled inverse historical subgraphs
│   └── his_dict/       # Context dictionaries
├── ICEWS18/
│   ├── [Same file structure as ICEWS14]
├── ICEWS05-15/
│   ├── [Same file structure as ICEWS14]
│   ├── ent2word.py     # Entity-word mapping preprocessing script
│   ├── word2id.txt     # Word ID mappings
│   └── e-w-graph.txt   # Entity-word relationship graph
├── WIKI/
│   ├── train.txt
│   ├── valid.txt
│   ├── test.txt
│   ├── entity2id.txt
│   ├── relation2id.txt
│   ├── stat.txt
│   ├── his_graph_for/  # Sampled forward historical subgraphs
│   ├── his_graph_inv/  # Sampled inverse historical subgraphs
│   └── his_dict/       # Context dictionaries
├── YAGO/
│   ├── train.txt
│   ├── valid.txt
│   ├── test.txt
│   ├── entity2id.txt
│   ├── relation2id.txt
│   ├── stat.txt
│   ├── his_graph_for/  # Sampled forward historical subgraphs
│   ├── his_graph_inv/  # Sampled inverse historical subgraphs
│   └── his_dict/       # Context dictionaries
├── GDELT/
│   ├── [Same file structure as ICEWS14]
└── get_his_subg.py     # Historical subgraph sampling preprocessing script
```

## 5. Usage Instructions

### Running Preprocessing Scripts

1. **Entity-word mapping preprocessing**:
   ```bash
   cd data/ICEWS05-15
   python ent2word.py
   ```

2. **Historical subgraph sampling preprocessing**:
   ```bash
   cd data
   python get_his_subg.py
   ```
   > Note: By default, it processes WIKI and YAGO datasets. To process other datasets, please modify the `dataset_list` variable

## 6. References

- Boschee, E., et al. (2015). The ICEWS dataset of political events, 1995-2015.
- Leetaru, K., & Schrodt, P. A. (2013). GDELT: Global database of events, language, and tone, 1979-2012.
- Leblay, J., & Chekol, M. W. (2018). Deriving validity time in knowledge graph. In Companion Proceedings of the The Web Conference 2018.
- Mahdisoltani, F., Biega, J., & Suchanek, F. (2014). Yago3: A knowledge base from multilingual wikipedias. In 7th biennial conference on innovative data systems research. CIDR Conference.
- Chen, W., Wan, H., Wu, Y., Zhao, S., Cheng, J., Li, Y., & Lin, Y. (2023). Local-Global History-aware Contrastive Learning for Temporal Knowledge Graph Reasoning. arXiv preprint arXiv:2312.01601.
