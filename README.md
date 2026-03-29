# Projet Web Data Mining — Knowledge Graph sur la créatine

================================================================
README — Instructions de reproduction des Labs
Cours : Web Datamining & Semantics
Domaine : Supplémentation en créatine
================================================================

## Vue d’ensemble
Ce projet consiste à extraire des connaissances à partir de données web sur la créatine, construire une base de connaissances (Knowledge Graph), l’aligner avec Wikidata, l’enrichir via des requêtes SPARQL, puis l’exploiter dans un système de question-réponse basé sur un Knowledge Graph

Le projet est structuré en plusieurs labs :

- Lab 1 : Collecte de données et extraction d’entités
- Lab 4 : Construction, alignement et expansion du Knowledge Graph
- Lab 5 : Raisonnement (SWRL) et embeddings (KGE)
- Lab 6 : Système RAG avec RDF/SPARQL et LLM local

----------------------------------------------------------------
LAB 1 — From the Unstructured Web to Structured Entities
----------------------------------------------------------------

Environnement : Google Colab (Python 3.12)

Dépendances à installer :
    pip install trafilatura spacy pandas httpx
    python -m spacy download fr_core_news_sm

Fichiers générés :
    - crawler_output.jsonl   : textes extraits des 4 URLs
    - extracted_knowledge.csv : entités NER filtrées (PER, ORG, LOC)

Instructions :
1. Ouvrir lab1.ipynb sur Google Colab
2. Monter le Drive si nécessaire (cellule commentée en haut)
3. Exécuter toutes les cellules dans l'ordre (Run All)
4. Les fichiers crawler_output.jsonl et extracted_knowledge.csv
   sont générés dans le répertoire courant

Notes :
- Le modèle utilisé est fr_core_news_sm (français) car les articles
  sources sont en français. Le TP recommande en_core_web_trf mais
  ce modèle ne convient pas à un corpus francophone.
- Un URL est bloqué par robots.txt (protegez-vous.ca), ce qui est
  le comportement attendu du crawler éthique.
- La méthode nsubj/dobj retourne peu de résultats en français,
  ce qui est expliqué dans le commentaire de la dernière cellule.


----------------------------------------------------------------
LAB 4 — KB Construction, Alignment, and Expansion
----------------------------------------------------------------

Environnement : Google Colab (Python 3.12) avec GPU recommandé

Dépendances à installer :
    pip install trafilatura spacy pandas httpx rdflib sparqlwrapper

Fichiers requis :
    - Aucun fichier externe requis (tout est généré depuis les URLs)

Fichiers générés :
    - graph_knowledge.nt     : KB privée initiale (116 triplets)
    - alignment.ttl          : alignement entités + ontologie
    - mapping_table.csv      : table de mapping Wikidata
    - predicate_alignment.ttl: alignement prédicats
    - expanded_kb.nt         : KB finale expansée (~67 000 triplets)

Instructions :
1. Ouvrir lab4.ipynb sur Google Colab
2. Monter le Drive (cellule 1 — décommenter si nécessaire)
3. Exécuter toutes les cellules dans l'ordre (Run All)
4. Le Step 4 (expansion SPARQL) peut prendre 10-20 minutes
   car il fait de nombreuses requêtes sur l'endpoint Wikidata

Notes :
- Les requêtes Wikidata peuvent échouer avec HTTP 429 (rate limit).
  En cas d'erreur, attendre quelques secondes et relancer la cellule.
- Le Step 4 génère environ 67 000 triplets après nettoyage et
  filtrage des 200 relations les plus fréquentes.
- Les statistiques finales attendues :
    Triplets  : ~67 000  (objectif : 50k-200k)
    Entités   : ~25 000  (objectif : 5k-30k)
    Relations : 200      (objectif : 50-200)


----------------------------------------------------------------
LAB 5 — Knowledge Reasoning with SWRL and KGE
----------------------------------------------------------------

Environnement : Google Colab avec GPU

Dépendances à installer :
    pip install owlready2 rdflib pykeen

Fichiers requis :
    - family.owl      : ontologie familiale (à placer dans le Drive)
    - expanded_kb.nt  : généré au lab 4 (à placer dans le Drive)

Instructions :
1. Activer le GPU : Exécution > Modifier le type d'exécution > GPU
2. Ouvrir Lab5.ipynb sur Google Colab
3. Monter le Drive (cellule commentée en haut)
4. Vérifier que family.owl et expanded_kb.nt sont dans le bon dossier
5. Exécuter toutes les cellules dans l'ordre

Notes importantes :

PART 1 (SWRL) :
- Java 25 est téléchargé automatiquement depuis oracle.com
  (cellule 11 — wget ~180MB, prend 1-2 minutes)
- Pellet est le reasoner utilisé (pas HermiT) car seul Pellet
  supporte les règles SWRL dans owlready2
- onto.individuals() retourne vide à cause du import Protege —
  utiliser onto.search(type=onto.Person) à la place (cellule 13)
- Résultat attendu : Peter (70) et Marie (69) classifiés oldPerson

PART 2 (KGE) :
- L'entraînement de TransE et DistMult prend 5-15 minutes avec GPU
- Sans GPU, peut prendre 1-2 heures — fortement déconseillé
- Les métriques attendues (realistic, both) :
    TransE  : MRR ~0.075, Hits@10 ~0.18
    DistMult: MRR ~0.266, Hits@10 ~0.42
- Le t-SNE est calculé sur 2000 entités (sous-échantillon)
  pour des raisons de performance
- Section 8 (SWRL vs embedding) : les relations p106 et p166
  sont introuvables dans le modèle entraîné car absentes du
  graphe expansé — ce résultat est documenté et expliqué


----------------------------------------------------------------
LAB 6 — RAG with RDF/SPARQL and a Local LLM
----------------------------------------------------------------

Environnement : LOCAL uniquement (Windows, Jupyter)
Ce lab NE PEUT PAS tourner sur Google Colab car il requiert
un serveur Ollama local sur http://localhost:11434

Prérequis :
1. Installer Ollama : https://ollama.com/download/windows
2. Ollama démarre automatiquement en service après installation
3. Télécharger le modèle :
       ollama pull llama3.2:1b
4. Installer les dépendances Python :
       pip install rdflib requests

Fichiers requis :
    - expanded_kb.nt : généré au lab 4
      (à placer dans le même dossier que le notebook)

Instructions :
1. Vérifier qu'Ollama tourne : ouvrir http://localhost:11434
   dans un navigateur (doit afficher "Ollama is running")
2. Ouvrir lab6_rag.ipynb dans Jupyter (pas Colab)
3. Exécuter toutes les cellules dans l'ordre
4. La cellule de comparaison 5 questions (cellule 15)
   prend environ 15-30 minutes sur CPU

Notes :
- Le modèle gemma:2b (recommandé dans le TP) refuse les tâches
  de génération SPARQL. llama3.2:1b est utilisé à la place —
  il est explicitement listé dans les alternatives du TP.
- Le RAG retourne 0 résultats sur toutes les questions car
  llama3.2:1b hallucine des préfixes inexistants (brick:, skos:).
  Ce comportement est documenté et analysé dans la Discussion.
- La boucle de self-repair fonctionne (Réparé ? True sur 4/5
  questions) mais ne peut pas corriger les IRIs inventés.
- La demo CLI (run_cli_demo) est commentée dans le notebook —
  pour la lancer en interactif : python lab6_rag.py
- Un modèle plus grand (7B+) améliorerait significativement
  la qualité de génération SPARQL.

================================================================
