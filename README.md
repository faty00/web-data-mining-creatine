# Knowledge Graph — Créatine & Suppléments alimentaires

Projet web datamining  - A4 DIA4 - Marya LATIF, Faty LO - 2025/2026

Construction d'un Knowledge Graph sur la créatine à partir d'articles web français,
avec alignement Wikidata, raisonnement SWRL, embeddings KGE et pipeline RAG.

---

## Structure du projet

```
project-root/
├── notebooks/
│   ├── lab1.ipynb           # Crawling + NER
│   ├── lab4.ipynb           # KB Construction + Alignment + Expansion
│   ├── Lab5.ipynb           # SWRL + KGE
│   └── lab6_rag.ipynb       # RAG pipeline
├── kg_artifacts/
│   ├── graph_knowledge.nt   # KB privée initiale
│   ├── expanded_kb.nt       # KB expansée finale (~67k triplets)
│   ├── alignment.ttl        # Alignement entités + ontologie
│   └── predicate_alignment.ttl
├── kge/
│   ├── train.txt            # 80% — 52 681 triplets
│   ├── valid.txt            # 10% — 4 786 triplets
│   └── test.txt             # 10% — 4 777 triplets
├── data/
│   ├── crawler_output.jsonl
│   ├── extracted_knowledge.csv
│   └── mapping_table.csv
├── reports/
│   └── final_report.pdf
├── README.md
├── requirements.txt
└── .gitignore
```

---

## Installation

```bash
pip install -r requirements.txt
python -m spacy download fr_core_news_sm
```

### Java (requis pour le raisonnement SWRL — Lab 5)

Télécharger Java 25 : https://www.oracle.com/java/technologies/downloads/#java25-windows

Sur Windows, après installation :
```
setx JAVA_HOME "C:\Program Files\Java\jdk-25.0.2" /M
```

### Ollama (requis pour le RAG — Lab 6)

1. Télécharger Ollama : https://ollama.com/download/windows
2. Après installation, Ollama démarre automatiquement en arrière-plan
3. Télécharger le modèle :
```
ollama pull llama3.2:1b
```
4. Vérifier que le service tourne : ouvrir http://localhost:11434 dans un navigateur

---

## Comment exécuter chaque module

### Lab 1 — Crawling + NER
Environnement : Google Colab
```
notebooks/lab1.ipynb
```
Génère : `crawler_output.jsonl`, `extracted_knowledge.csv`

### Lab 4 — KB Construction, Alignment & Expansion
Environnement : Google Colab (GPU recommandé)
```
notebooks/lab4.ipynb
```
Génère : `graph_knowledge.nt`, `alignment.ttl`, `mapping_table.csv`, `expanded_kb.nt`

Note : le Step 4 (expansion SPARQL) peut prendre 10-20 minutes.

### Lab 5 — SWRL + KGE
Environnement : Google Colab avec GPU (T4)
```
notebooks/Lab5.ipynb
```
Fichiers requis : `family.owl`, `expanded_kb.nt`
Génère : `train.txt`, `valid.txt`, `test.txt`, `tsne_embeddings.png`

Note : Java 25 est téléchargé automatiquement dans la cellule 11.

### Lab 6 — RAG Pipeline
Environnement : **Local uniquement** (Ollama ne fonctionne pas sur Colab)
```
jupyter notebook notebooks/lab6_rag.ipynb
```
Fichiers requis : `expanded_kb.nt` dans le même dossier que le notebook

Pour lancer la démo CLI interactive :
```
python lab6_rag.py
```

---

## Lancer la démo RAG

1. Vérifier qu'Ollama tourne (http://localhost:11434)
2. Placer `expanded_kb.nt` dans le dossier `notebooks/`
3. Ouvrir `lab6_rag.ipynb` dans Jupyter
4. Exécuter toutes les cellules dans l'ordre

---

## Hardware requis

| Module | Environnement | GPU requis |
|---|---|---|
| Lab 1 (Crawling + NER) | Google Colab | Non |
| Lab 4 (KB + Expansion) | Google Colab | Non |
| Lab 5 (KGE) | Google Colab | Oui (T4 recommandé) |
| Lab 6 (RAG) | Local | Non |

---

## Statistiques du Knowledge Graph final

| Dimension | Valeur | Objectif |
|---|---|---|
| Triplets | 67 234 | 50k – 200k |
| Entités | ~25 000 | 5k – 30k |
| Relations | 200 | 50 – 200 |

---

## Screenshot

![RAG Pipeline](reports/screenshot_rag.png)

---

## Fichiers volumineux

`expanded_kb.nt` (~67k triplets) est inclus dans `kg_artifacts/`.
Si le fichier dépasse la limite GitHub (100MB), le télécharger depuis la release v1.0-final.
