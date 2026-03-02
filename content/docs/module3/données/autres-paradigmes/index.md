---
title: "Autres paradigmes importants"
weight: 30
---

# Autres paradigmes importants

## Logging

Le *logging* (journalisation) consiste à enregistrer de façon systématique les
événements qui se produisent dans un système&nbsp;: erreurs, avertissements, actions
utilisateur, métriques de performance, etc. Dans un contexte de gestion des
données, les logs constituent eux-mêmes une source de données précieuse. Le
module `logging` de Python offre un système structuré avec des niveaux de
sévérité (`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`), des *handlers*
configurables et des formats personnalisables. À petite échelle, on peut écrire
les logs dans des fichiers locaux, mais à l'échelle d'une infrastructure
distribuée, le volume de logs générés devient rapidement massif et difficile à
exploiter. C'est là qu'interviennent des outils comme
[Elasticsearch](https://www.elastic.co/elasticsearch), un moteur de recherche et
d'analytique distribué souvent déployé au sein de la pile « ELK »
(Elasticsearch, Logstash, Kibana). Logstash collecte et transforme les logs
provenant de sources multiples, Elasticsearch les indexe pour permettre des
recherches *full-text* quasi instantanées, et Kibana fournit des tableaux de
bord de visualisation. Cette approche transforme des logs bruts — qui seraient
autrement difficiles à consulter — en une base de données interrogeable en temps
réel, permettant le débogage, la surveillance d'anomalies et l'analyse de
tendances.

{{< pyodide >}}
import logging

# Configuration d'un logger avec deux handlers
logger = logging.getLogger("mon_application")
logger.setLevel(logging.DEBUG)

# Handler console (INFO et plus)
console = logging.StreamHandler()
console.setLevel(logging.INFO)
console.setFormatter(logging.Formatter("%(levelname)s - %(message)s"))
logger.addHandler(console)

# Simulation d'événements applicatifs
logger.info("Démarrage de l'application")
logger.debug("Connexion à la base de données (détail ignoré en console)")
logger.info("Utilisateur alice s'est connecté")
logger.warning("Temps de réponse élevé : 2.3s pour /api/données")
logger.error("Échec de la requête vers le service externe (timeout)")
logger.info("Utilisateur alice a exporté le rapport mensuel")
logger.critical("Espace disque insuffisant sur /var/log")
{{< /pyodide >}}

## Messaging

Le *messaging* (messagerie asynchrone) est un paradigme dans lequel des
composants logiciels communiquent en s'échangeant des messages via un
intermédiaire (*broker*), plutôt qu'en s'appelant directement. Ce découplage est
fondamental pour les architectures distribuées&nbsp;: le producteur d'un message n'a
pas besoin de savoir qui le consommera, ni même si le consommateur est
disponible au moment de l'envoi. Le broker se charge de stocker les messages
dans des files d'attente (*queues*) et de les acheminer vers les consommateurs
selon différents patrons — point à point, publication/abonnement (*pub/sub*), ou
routage par sujet (*topic*). Dans un contexte de gestion des données, le
messaging permet d'orchestrer des pipelines de traitement&nbsp;: par exemple, un
événement « nouvelle commande » peut déclencher simultanément la mise à jour de
l'inventaire, l'envoi d'un courriel de confirmation et l'alimentation d'un
entrepôt analytique, chaque tâche étant gérée par un consommateur indépendant.
[RabbitMQ](https://www.rabbitmq.com/) et [Apache
Kafka](https://kafka.apache.org/) sont parmi les brokers les plus utilisés.
Python offre la bibliothèque `queue` dans sa librairie standard, qui implémente
les mêmes concepts fondamentaux (producteur, consommateur, file d'attente) dans
un contexte local multi-thread.

{{< pyodide >}}
from collections import deque
import random

# --- File d'attente (simule un broker de messages) ---

class FileDAttente:
    """Simule une file de messages (queue) comme dans RabbitMQ ou Kafka."""

    def __init__(self, nom):
        self.nom = nom
        self.messages = deque()
        self.abonnes = []  # liste de consommateurs (pub/sub)

    def publier(self, message):
        """Un producteur dépose un message dans la file."""
        self.messages.append(message)

    def consommer(self):
        """Un consommateur retire le prochain message (point à point)."""
        if self.messages:
            return self.messages.popleft()
        return None

    def diffuser(self, message):
        """Publie un message à tous les abonnés (pub/sub)."""
        for callback in self.abonnes:
            callback(message)

    def abonner(self, callback):
        """Enregistre un abonné qui recevra tous les messages diffusés."""
        self.abonnes.append(callback)

# --- Producteurs ---

def producteur(nom, file, n_messages):
    """Simule un service qui génère des événements."""
    produits = ["Livre", "Clavier", "Écran", "Câble USB", "Souris"]
    for i in range(n_messages):
        commande = {
            "id": f"{nom}-{i+1:03d}",
            "produit": random.choice(produits),
            "quantité": random.randint(1, 5),
        }
        file.publier(commande)
        print(f"[{nom}] Publié : {commande['id']} → {commande['quantité']}x {commande['produit']}")
    print(f"[{nom}] Terminé.")

# --- Consommateurs ---

def consommateur(nom, file):
    """Simule un service qui traite les messages de la file."""
    traites = 0
    while True:
        message = file.consommer()
        if message is None:
            break
        print(f"  [{nom}] Traitement de {message['id']} : {message['quantité']}x {message['produit']}")
        traites += 1
    print(f"  [{nom}] {traites} message(s) traité(s).")

# === Patron 1 : Point à point (chaque message traité par UN seul consommateur) ===

print("=== Point à point (queue) ===\n")

file_commandes = FileDAttente("commandes")

# Deux producteurs publient des commandes
producteur("Web", file_commandes, 3)
producteur("Mobile", file_commandes, 3)

print(f"\n--- {len(file_commandes.messages)} messages en file ---\n")

# Deux consommateurs se partagent les messages
consommateur("Inventaire", file_commandes)
consommateur("Courriel", file_commandes)

# === Patron 2 : Publication / abonnement (chaque message reçu par TOUS les abonnés) ===

print(f"\n=== Publication / abonnement (pub/sub) ===\n")

file_alertes = FileDAttente("alertes")

# Enregistrer des abonnés
journal = []
file_alertes.abonner(lambda msg: print(f"  [Monitoring] Alerte reçue : {msg}"))
file_alertes.abonner(lambda msg: print(f"  [Courriel]   Notification : {msg}"))
file_alertes.abonner(lambda msg: journal.append(msg))

# Publier des alertes (chaque abonné les reçoit toutes)
for alerte in ["CPU > 90%", "Disque presque plein", "Latence élevée"]:
    print(f"[Système] Diffusion : {alerte}")
    file_alertes.diffuser(alerte)

print(f"\n  Journal : {journal}")
{{< /pyodide >}}

## Streaming

Le *streaming* (traitement en flux) est un paradigme où les données sont
traitées de manière continue, au fur et à mesure de leur arrivée, plutôt que
d'être accumulées puis traitées en lot (*batch*). Cette distinction est cruciale
en gestion des données&nbsp;: un système batch calcule un rapport quotidien des
ventes à partir d'une base de données, tandis qu'un système en streaming met à
jour ce rapport en temps réel à chaque nouvelle transaction. Le streaming repose
sur la notion de flux (*stream*) — une séquence potentiellement infinie
d'événements ordonnés dans le temps. [Apache Kafka](https://kafka.apache.org/)
(qui est aussi un broker de messaging, cf. section précédente) et [Apache
Flink](https://flink.apache.org/) sont les plateformes de référence pour le
streaming distribué à grande échelle. Kafka modélise les flux comme des *topics*
partitionnés et persistants, permettant à plusieurs consommateurs de lire les
données à leur propre rythme. Flink ajoute des capacités de traitement
complexe&nbsp;: fenêtres temporelles (*windowing*), agrégations continues et
jointures entre flux. En Python, les générateurs et le mot-clé `yield` incarnent
naturellement le concept de flux&nbsp;: ils produisent des valeurs une à la
fois, sans charger l'ensemble des données en mémoire, ce qui est exactement le
principe fondamental du streaming.

{{< pyodide >}}
import random
import time
from collections import defaultdict

# --- Générateur : simule un flux infini d'événements ---

def flux_transactions(n):
    """Simule un flux de transactions en temps réel."""
    magasins = ["Montréal", "Québec", "Sherbrooke", "Gatineau"]
    produits = ["Café", "Thé", "Jus", "Eau", "Limonade"]
    for i in range(n):
        yield {
            "id": i + 1,
            "magasin": random.choice(magasins),
            "produit": random.choice(produits),
            "montant": round(random.uniform(2.0, 15.0), 2),
        }

# --- Traitement en fenêtre glissante (window) ---

def fenetre_glissante(flux, taille_fenetre):
    """Agrège les ventes par magasin sur une fenêtre de N événements."""
    fenetre = []
    for evenement in flux:
        fenetre.append(evenement)
        if len(fenetre) == taille_fenetre:
            # Calculer les agrégats pour cette fenêtre
            ventes_par_magasin = defaultdict(float)
            for e in fenetre:
                ventes_par_magasin[e["magasin"]] += e["montant"]
            yield {
                "ids": f"{fenetre[0]['id']}-{fenetre[-1]['id']}",
                "ventes": dict(ventes_par_magasin),
            }
            fenetre = []  # Réinitialiser la fenêtre

# --- Pipeline de streaming ---

print("=== Streaming : ventes en temps réel ===\n")

flux = flux_transactions(20)

for rapport in fenetre_glissante(flux, taille_fenetre=5):
    print(f"Fenêtre [{rapport['ids']}]")
    for magasin, total in sorted(rapport["ventes"].items()):
        print(f"  {magasin:12s} : {total:6.2f} $")
    print()
{{< /pyodide >}}

## Caching

Le *caching* (mise en cache) consiste à stocker temporairement des données
fréquemment accédées dans un espace mémoire rapide, afin d'éviter de les
recalculer ou de les récupérer depuis une source plus lente (base de données,
API externe, système de fichiers). Dans un contexte de gestion des données, le
cache est omniprésent&nbsp;: il réduit la latence des requêtes, diminue la charge sur
les bases de données et améliore la capacité d'un système à supporter un grand
nombre d'utilisateurs simultanés. [Redis](https://redis.io/) est le système de
cache distribué le plus populaire — c'est un magasin clé-valeur en mémoire
(*in-memory*) qui offre des temps de réponse de l'ordre de la microseconde et
supporte des structures de données riches (chaînes, listes, ensembles,
hachages). [Memcached](https://memcached.org/) est une alternative plus simple
et légère. Le défi principal du caching est l'**invalidation**&nbsp;: s'assurer que
le cache reste cohérent avec la source de vérité. Les stratégies courantes
incluent le TTL (*time-to-live*, expiration après un délai fixe), le
*write-through* (mise à jour simultanée du cache et de la source) et le
*cache-aside* (le code vérifie d'abord le cache, puis interroge la source en cas
d'absence). Python offre `functools.lru_cache`, un décorateur qui implémente un
cache LRU (*Least Recently Used*) directement sur les appels de fonction.

{{< pyodide >}}
import functools
import time

# --- Cache LRU intégré à Python ---

@functools.lru_cache(maxsize=128)
def requete_couteuse(utilisateur_id):
    """Simule une requête lente vers une base de données."""
    print(f"  [BD] Requête pour utilisateur {utilisateur_id}...")
    time.sleep(0.01)  # Simule la latence
    return {"id": utilisateur_id, "nom": f"Utilisateur_{utilisateur_id}", "score": utilisateur_id * 17 % 100}

# --- Démonstration : avec et sans cache ---

print("=== Premier appel (cache vide — miss) ===\n")
for uid in [42, 7, 42, 13, 7, 42]:
    t0 = time.time()
    resultat = requete_couteuse(uid)
    duree = (time.time() - t0) * 1000
    source = "MISS → BD" if duree > 5 else "HIT  → cache"
    print(f"  utilisateur {uid:2d} → {resultat['nom']:16s} (score={resultat['score']:2d})  [{source}]")

print(f"\n=== Statistiques du cache ===\n")
info = requete_couteuse.cache_info()
print(f"  Hits   : {info.hits}")
print(f"  Misses : {info.misses}")
print(f"  Taille : {info.currsize}/{info.maxsize}")
print(f"  Taux   : {info.hits / (info.hits + info.misses) * 100:.0f}%")

# --- Invalidation manuelle ---

print(f"\n=== Invalidation du cache ===\n")
requete_couteuse.cache_clear()
print(f"  Cache vidé. Nouvelle taille : {requete_couteuse.cache_info().currsize}")
{{< /pyodide >}}

## Searching and indexing

La *recherche et l'indexation* (*searching and indexing*) désignent l'ensemble
des techniques permettant de retrouver rapidement de l'information dans de
grands volumes de données, en particulier des données textuelles non
structurées. Contrairement à une requête SQL qui filtre des colonnes bien
définies, la recherche *full-text* doit gérer la langue naturelle&nbsp;: synonymes,
flexions grammaticales, fautes de frappe, pertinence relative des résultats. Le
mécanisme fondamental est l'**index inversé** (*inverted index*)&nbsp;: plutôt que de
parcourir chaque document pour y chercher un mot, on construit à l'avance une
table qui associe chaque terme à la liste des documents qui le contiennent —
exactement comme l'index à la fin d'un livre.
[Elasticsearch](https://www.elastic.co/elasticsearch) (déjà mentionné dans la
section sur le logging) et [Apache Solr](https://solr.apache.org/), tous deux
construits sur la bibliothèque [Apache Lucene](https://lucene.apache.org/), sont
les moteurs de recherche les plus utilisés en entreprise. Ils offrent la
tokenisation (découpage du texte en termes), la racinisation (*stemming*&nbsp;: «
chercher », « cherche », « cherché » → « cherch »), le classement par pertinence
(TF-IDF, BM25) et la recherche par facettes. En Python, on peut illustrer le
concept d'index inversé avec un simple dictionnaire, ce qui permet de comprendre
le principe avant de passer à des systèmes distribués.

{{< pyodide >}}
import re
from collections import defaultdict

# --- Construction d'un index inversé ---

documents = {
    1: "Le langage Python est utilis�� en science des données et en intelligence artificielle",
    2: "Les bases de données relationnelles utilisent le langage SQL pour les requêtes",
    3: "Python permet de se connecter à des bases de données avec des bibliothèques comme SQLAlchemy",
    4: "L'intelligence artificielle utilise des modèles entraînés sur de grandes quantités de données",
    5: "La science des données combine statistiques, programmation Python et visualisation",
}

def tokeniser(texte):
    """Découpe un texte en tokens normalisés (minuscules, sans ponctuation)."""
    return re.findall(r"\b[a-zàâéèêëïîôùûüç]+\b", texte.lower())

# Construire l'index inversé
index = defaultdict(set)
for doc_id, texte in documents.items():
    for token in tokeniser(texte):
        index[token].add(doc_id)

print(f"=== Index inversé ({len(index)} termes) ===\n")
for terme in ["python", "données", "intelligence", "sql"]:
    print(f'  "{terme}" → documents {sorted(index[terme])}')

# --- Recherche avec classement par pertinence (TF simple) ---

def rechercher(requete, index, documents):
    """Recherche les documents correspondant à la requête, classés par pertinence."""
    termes = tokeniser(requete)
    scores = defaultdict(int)
    for terme in termes:
        for doc_id in index.get(terme, set()):
            scores[doc_id] += 1  # Score = nombre de termes trouvés
    resultats = sorted(scores.items(), key=lambda x: x[1], reverse=True)
    return resultats

print(f"\n=== Recherche : 'Python données science' ===\n")
resultats = rechercher("Python données science", index, documents)
for doc_id, score in resultats:
    print(f"  [{score} terme(s)] Doc {doc_id}: {documents[doc_id][:70]}...")

print(f"\n=== Recherche : 'intelligence artificielle' ===\n")
resultats = rechercher("intelligence artificielle", index, documents)
for doc_id, score in resultats:
    print(f"  [{score} terme(s)] Doc {doc_id}: {documents[doc_id][:70]}...")
{{< /pyodide >}}

## Object storage

Le *stockage objet* (*object storage*) est un paradigme de stockage conçu pour
gérer de très grands volumes de données non structurées — images, vidéos,
fichiers audio, sauvegardes, jeux de données scientifiques, archives de logs —
sous forme d'objets identifiés par une clé unique dans un espace de noms plat
(*flat namespace*), sans la hiérarchie de répertoires d'un système de fichiers
traditionnel. Chaque objet contient les données brutes (*blob*), des métadonnées
descriptives (type MIME, date de création, tags personnalisés) et sa clé
d'identification. Ce modèle est idéal pour le stockage à l'échelle du pétaoctet&nbsp;: il se distribue et se réplique facilement, offre une haute disponibilité et
s'intègre naturellement aux architectures infonuagiques (*cloud*). [Amazon
S3](https://aws.amazon.com/s3/) est le service de référence qui a popularisé ce
paradigme ; son API est devenue un standard *de facto* implémenté par de
nombreuses alternatives comme [MinIO](https://min.io/) (open source,
auto-hébergeable) et [Google Cloud Storage](https://cloud.google.com/storage).
Dans un contexte de gestion des données, le stockage objet sert souvent de « lac
de données » (*data lake*) brut à partir duquel des pipelines de traitement
extraient, transforment et chargent les données vers des bases structurées. En
Python, on peut illustrer les concepts fondamentaux du stockage objet — espace
de noms plat, métadonnées, opérations CRUD par clé — avec un simple
dictionnaire.

{{< pyodide >}}
import datetime
import hashlib
import json

class StockageObjet:
    """Simulacre minimaliste d'un service de stockage objet (style S3)."""

    def __init__(self, nom_bucket):
        self.nom = nom_bucket
        self.objets = {}  # clé → {"donnees": bytes, "meta": dict}

    def put(self, cle, donnees, **meta):
        """Dépose un objet dans le bucket."""
        contenu = donnees.encode() if isinstance(donnees, str) else donnees
        self.objets[cle] = {
            "donnees": contenu,
            "meta": {
                "taille": len(contenu),
                "md5": hashlib.md5(contenu).hexdigest(),
                "date": datetime.datetime.now().isoformat(),
                **meta,
            },
        }
        print(f"  PUT  {cle}  ({len(contenu)} octets)")

    def get(self, cle):
        """Récupère un objet par sa clé."""
        obj = self.objets.get(cle)
        if obj is None:
            print(f"  GET  {cle}  → NON TROUVÉ")
            return None
        print(f"  GET  {cle}  → {obj['meta']['taille']} octets")
        return obj["donnees"]

    def head(self, cle):
        """Retourne uniquement les métadonnées (sans télécharger le contenu)."""
        obj = self.objets.get(cle)
        if obj is None:
            return None
        return obj["meta"]

    def ls(self, prefixe=""):
        """Liste les clés avec un préfixe donné (simule le flat namespace)."""
        return [c for c in sorted(self.objets) if c.startswith(prefixe)]

    def delete(self, cle):
        """Supprime un objet."""
        if cle in self.objets:
            del self.objets[cle]
            print(f"  DEL  {cle}")

# --- Démonstration ---

bucket = StockageObjet("mon-data-lake")

print("=== Dépôt d'objets ===\n")
bucket.put("rapports/2025/ventes-q1.csv", "produit,montant\nCafé,1200\nThé,800", type="text/csv")
bucket.put("rapports/2025/ventes-q2.csv", "produit,montant\nCafé,1350\nThé,920", type="text/csv")
bucket.put("images/logo.png", "<<données binaires PNG simulées>>", type="image/png")
bucket.put("modeles/classification-v2.pkl", "<<modèle sérialisé>>", type="application/octet-stream")

print(f"\n=== Liste par préfixe ===\n")
for prefixe in ["rapports/", "images/", ""]:
    cles = bucket.ls(prefixe)
    label = prefixe if prefixe else "(tout)"
    print(f'  "{label}" → {cles}')

print(f"\n=== Métadonnées (HEAD) ===\n")
meta = bucket.head("rapports/2025/ventes-q1.csv")
for k, v in meta.items():
    print(f"  {k:8s} : {v}")

print(f"\n=== Lecture et suppression ===\n")
contenu = bucket.get("rapports/2025/ventes-q1.csv")
print(f"  Contenu : {contenu.decode()[:50]}...")
bucket.delete("modeles/classification-v2.pkl")
print(f"  Objets restants : {bucket.ls()}")
{{< /pyodide >}}

## Analytics / OLAP

L'*analytique* et l'*OLAP* (*Online Analytical Processing*) désignent un
paradigme de gestion des données optimisé pour l'analyse de grands volumes
plutôt que pour le traitement transactionnel. Les bases de données
transactionnelles classiques (OLTP — *Online Transaction Processing*) sont
conçues pour insérer, modifier et lire des enregistrements individuels
rapidement (une commande, un virement), tandis que les systèmes OLAP sont
optimisés pour des requêtes analytiques qui agrègent des millions de lignes&nbsp;: «
quel est le chiffre d'affaires par région et par trimestre ? », « quelle est la
tendance des ventes sur 3 ans ? ». La distinction architecturale clé est le
**stockage en colonnes** (*columnar storage*)&nbsp;: plutôt que de stocker les
données ligne par ligne (comme PostgreSQL ou MySQL), les bases OLAP stockent
chaque colonne de manière contiguë, ce qui permet de lire uniquement les
colonnes nécessaires à une agrégation et de les compresser très efficacement.
[Apache Parquet](https://parquet.apache.org/) est le format de fichier columnar
de référence dans l'écosystème big data, tandis que
[ClickHouse](https://clickhouse.com/), [DuckDB](https://duckdb.org/) et [Google
BigQuery](https://cloud.google.com/bigquery) sont des moteurs OLAP populaires.
Le concept d'**entrepôt de données** (*data warehouse*) repose sur ce paradigme&nbsp;: les données transactionnelles sont périodiquement extraites, transformées et
chargées (ETL) dans un schéma en étoile (*star schema*) optimisé pour l'analyse
dimensionnelle. En Python, on peut illustrer la différence entre stockage en
lignes et en colonnes, ainsi que l'avantage du columnar pour les agrégations.

{{< pyodide >}}
import random
import time

# --- Génération de données de ventes ---

random.seed(42)
regions = ["Montréal", "Québec", "Sherbrooke", "Gatineau"]
produits = ["Café", "Thé", "Jus", "Eau"]
trimestres = ["T1", "T2", "T3", "T4"]

N = 10_000  # nombre de transactions

# Stockage en LIGNES (style OLTP) : chaque ligne est un dict
lignes = [
    {
        "region": random.choice(regions),
        "produit": random.choice(produits),
        "trimestre": random.choice(trimestres),
        "montant": round(random.uniform(5.0, 50.0), 2),
    }
    for _ in range(N)
]

# Stockage en COLONNES (style OLAP) : chaque colonne est une liste
colonnes = {
    "region":     [r["region"] for r in lignes],
    "produit":    [r["produit"] for r in lignes],
    "trimestre":  [r["trimestre"] for r in lignes],
    "montant":    [r["montant"] for r in lignes],
}

print(f"=== {N} transactions générées ===\n")

# --- Requête analytique : ventes par région ---
# En mode colonnes, on ne touche que les colonnes nécessaires

def ventes_par_region_lignes(lignes):
    """Agrégation en parcourant les lignes (OLTP-style)."""
    totaux = {}
    for ligne in lignes:
        r = ligne["region"]        # accède à TOUTE la ligne
        totaux[r] = totaux.get(r, 0) + ligne["montant"]
    return totaux

def ventes_par_region_colonnes(colonnes):
    """Agrégation en parcourant les colonnes (OLAP-style)."""
    totaux = {}
    regions = colonnes["region"]   # ne lit QUE cette colonne
    montants = colonnes["montant"] # et celle-ci
    for i in range(len(regions)):
        r = regions[i]
        totaux[r] = totaux.get(r, 0) + montants[i]
    return totaux

# Comparer les résultats
res_lignes = ventes_par_region_lignes(lignes)
res_colonnes = ventes_par_region_colonnes(colonnes)

print("Ventes totales par région :\n")
for region in sorted(res_colonnes):
    print(f"  {region:12s} : {res_colonnes[region]:10,.2f} $")

print(f"\n  Total       : {sum(res_colonnes.values()):10,.2f} $")

# --- Requête multidimensionnelle (cube OLAP simplifié) ---

print(f"\n=== Cube OLAP : ventes par région × trimestre ===\n")

cube = {}
for i in range(N):
    cle = (colonnes["region"][i], colonnes["trimestre"][i])
    cube[cle] = cube.get(cle, 0) + colonnes["montant"][i]

# Affichage en tableau croisé
print(f"  {'':12s}", end="")
for t in trimestres:
    print(f"  {t:>10s}", end="")
print()

for region in sorted(regions):
    print(f"  {region:12s}", end="")
    for t in trimestres:
        val = cube.get((region, t), 0)
        print(f"  {val:10,.2f}", end="")
    print()
{{< /pyodide >}}

## Time series

Les *séries temporelles* (*time series*) sont des séquences de points de données
indexés par le temps&nbsp;: température toutes les minutes, cours boursier toutes les
secondes, nombre de requêtes par heure sur un serveur, fréquence cardiaque d'un
patient. Ce type de données est omniprésent dans la surveillance
d'infrastructure (monitoring), l'Internet des objets (IoT), la finance et les
sciences. Les bases de données relationnelles classiques peuvent stocker des
séries temporelles, mais elles ne sont pas optimisées pour leurs particularités&nbsp;: les données arrivent principalement en mode *append* (on ajoute sans
modifier), les requêtes portent presque toujours sur des intervalles de temps,
et le volume peut être considérable (des millions de points par capteur par
jour). Les bases de données spécialisées en séries temporelles exploitent ces
propriétés pour offrir une compression agressive, des fonctions d'agrégation
temporelle natives (*downsample*, *rollup*) et des politiques de rétention
automatiques (supprimer les données de plus de 90 jours, par exemple).
[InfluxDB](https://www.influxdata.com/) et
[TimescaleDB](https://www.timescale.com/) (extension PostgreSQL) sont les
solutions les plus populaires, tandis que [Prometheus](https://prometheus.io/)
domine le monitoring d'infrastructure. En Python, on peut illustrer les
opérations fondamentales sur les séries temporelles — rééchantillonnage, moyenne
mobile, détection d'anomalies — avec les structures de données de base.

{{< pyodide >}}
import random
import math
from datetime import datetime, timedelta

# --- Génération d'une série temporelle : température d'un capteur IoT ---

random.seed(42)
debut = datetime(2025, 6, 1, 0, 0)
n_points = 288  # 1 point toutes les 5 min × 24h = 288

serie = []
for i in range(n_points):
    t = debut + timedelta(minutes=5 * i)
    heure = t.hour + t.minute / 60
    # Température sinusoïdale (cycle jour/nuit) + bruit
    base = 20 + 8 * math.sin((heure - 6) * math.pi / 12)
    bruit = random.gauss(0, 1.5)
    # Injection d'une anomalie à 14h30
    anomalie = 15 if (t.hour == 14 and t.minute == 30) else 0
    serie.append({"temps": t, "valeur": round(base + bruit + anomalie, 2)})

print(f"=== Série temporelle : {n_points} points (5 min) sur 24h ===\n")
print(f"  Début : {serie[0]['temps']}")
print(f"  Fin   : {serie[-1]['temps']}")
print(f"  Min   : {min(s['valeur'] for s in serie):.1f} °C")
print(f"  Max   : {max(s['valeur'] for s in serie):.1f} °C")

# --- Rééchantillonnage (downsample) : moyenne horaire ---

print(f"\n=== Downsample : moyenne par heure ===\n")
par_heure = {}
for s in serie:
    h = s["temps"].hour
    par_heure.setdefault(h, []).append(s["valeur"])

for h in sorted(par_heure):
    vals = par_heure[h]
    moy = sum(vals) / len(vals)
    print(f"  {h:02d}h : {moy:5.1f} °C  ({len(vals)} points)")

# --- Moyenne mobile (rolling average) ---

def moyenne_mobile(serie, fenetre):
    """Calcule la moyenne mobile sur une fenêtre de N points."""
    resultats = []
    for i in range(len(serie)):
        debut_f = max(0, i - fenetre + 1)
        vals = [serie[j]["valeur"] for j in range(debut_f, i + 1)]
        resultats.append({
            "temps": serie[i]["temps"],
            "brut": serie[i]["valeur"],
            "lissé": round(sum(vals) / len(vals), 2),
        })
    return resultats

lisse = moyenne_mobile(serie, fenetre=12)  # fenêtre d'1 heure

# --- Détection d'anomalies (seuil sur l'écart à la moyenne mobile) ---

print(f"\n=== Détection d'anomalies (écart > 10 °C vs moyenne mobile) ===\n")
anomalies = []
for pt in lisse:
    ecart = abs(pt["brut"] - pt["lissé"])
    if ecart > 10:
        anomalies.append(pt)
        print(f"  ⚠ {pt['temps'].strftime('%H:%M')} : brut={pt['brut']:.1f} °C, "
              f"lissé={pt['lissé']:.1f} °C, écart={ecart:.1f} °C")

if not anomalies:
    print("  Aucune anomalie détectée.")
else:
    print(f"\n  {len(anomalies)} anomalie(s) détectée(s).")
{{< /pyodide >}}

## Configuration / coordination

La *configuration et coordination distribuée* est un paradigme qui consiste à
centraliser les paramètres de configuration, la découverte de services et la
synchronisation d'état dans un magasin de données hautement disponible et
cohérent, partagé par tous les composants d'une infrastructure distribuée. Dans
un système monolithique, la configuration peut résider dans un simple fichier ;
mais dans une architecture à microservices — avec des dizaines ou des centaines
de processus répartis sur plusieurs machines — il faut un mécanisme fiable pour
que chaque service sache où trouver les autres, quelle est la version courante
d'un paramètre, et qui détient un verrou ou un rôle de leader. [Apache
ZooKeeper](https://zookeeper.apache.org/) a été le pionnier de ce domaine,
utilisé notamment par Kafka et Hadoop pour l'élection de leader et la gestion de
la topologie de cluster. [etcd](https://etcd.io/) est le magasin clé-valeur
distribué qui sous-tend Kubernetes&nbsp;: il stocke l'état complet du cluster (pods,
services, secrets) avec des garanties de consensus fort (algorithme Raft).
[HashiCorp Consul](https://www.consul.io/) combine la configuration clé-valeur
avec la découverte de services et la vérification de santé (*health checks*).
Ces systèmes partagent des primitives communes&nbsp;: le stockage clé-valeur
hiérarchique, la surveillance de changements (*watch*), les baux temporaires
(*leases*) et l'élection de leader. En Python, on peut illustrer ces mécanismes
fondamentaux avec un registre de configuration en mémoire qui supporte les
notifications de changement et la découverte de services.

{{< pyodide >}}
import time
from datetime import datetime

class RegistreConfiguration:
    """Simulacre d'un magasin de configuration distribué (style etcd/Consul)."""

    def __init__(self):
        self.donnees = {}          # clé → {"valeur": ..., "version": int}
        self.observateurs = {}     # clé → [callbacks]
        self.services = {}         # nom_service → [{"adresse": ..., "port": ..., "sain": bool}]
        self.version_globale = 0

    # --- Stockage clé-valeur ---

    def put(self, cle, valeur):
        """Écrit une clé avec versionnement automatique."""
        self.version_globale += 1
        ancienne = self.donnees.get(cle)
        self.donnees[cle] = {"valeur": valeur, "version": self.version_globale}
        print(f"  PUT {cle} = {valeur!r}  (v{self.version_globale})")
        # Notifier les observateurs (watch)
        for cb in self.observateurs.get(cle, []):
            cb(cle, ancienne, self.donnees[cle])

    def get(self, cle):
        """Lit une clé avec sa version."""
        return self.donnees.get(cle)

    def get_prefix(self, prefixe):
        """Retourne toutes les clés sous un préfixe (arborescence hiérarchique)."""
        return {c: v for c, v in self.donnees.items() if c.startswith(prefixe)}

    # --- Watch (surveillance de changements) ---

    def watch(self, cle, callback):
        """Enregistre un observateur notifié à chaque modification de la clé."""
        self.observateurs.setdefault(cle, []).append(callback)
        print(f"  WATCH {cle}  (observateur enregistré)")

    # --- Découverte de services ---

    def enregistrer_service(self, nom, adresse, port):
        """Enregistre une instance de service."""
        instance = {"adresse": adresse, "port": port, "sain": True}
        self.services.setdefault(nom, []).append(instance)
        print(f"  REGISTER {nom} → {adresse}:{port}")

    def decouvrir(self, nom):
        """Retourne les instances saines d'un service."""
        return [s for s in self.services.get(nom, []) if s["sain"]]

    def marquer_malsain(self, nom, adresse):
        """Simule un health check échoué."""
        for s in self.services.get(nom, []):
            if s["adresse"] == adresse:
                s["sain"] = False
                print(f"  UNHEALTHY {nom} @ {adresse}")

# --- Démonstration ---

registre = RegistreConfiguration()

print("=== Configuration clé-valeur ===\n")
registre.put("/app/database/host", "db-primary.internal")
registre.put("/app/database/port", 5432)
registre.put("/app/database/max_connections", 100)
registre.put("/app/cache/host", "redis-01.internal")
registre.put("/app/feature_flags/dark_mode", True)

print(f"\n=== Lecture par préfixe ===\n")
for cle, info in registre.get_prefix("/app/database/").items():
    print(f"  {cle} = {info['valeur']}  (v{info['version']})")

print(f"\n=== Watch (notification de changement) ===\n")

def on_change(cle, ancien, nouveau):
    anc = ancien['valeur'] if ancien else 'N/A'
    print(f"  ⚡ CHANGEMENT {cle} : {anc} → {nouveau['valeur']}")

registre.watch("/app/database/max_connections", on_change)
registre.put("/app/database/max_connections", 200)  # Déclenche la notification

print(f"\n=== Découverte de services ===\n")
registre.enregistrer_service("api-gateway", "10.0.1.10", 8080)
registre.enregistrer_service("api-gateway", "10.0.1.11", 8080)
registre.enregistrer_service("api-gateway", "10.0.1.12", 8080)

instances = registre.decouvrir("api-gateway")
print(f"\n  Instances saines de 'api-gateway' : {len(instances)}")
for s in instances:
    print(f"    → {s['adresse']}:{s['port']}")

print(f"\n=== Health check échoué ===\n")
registre.marquer_malsain("api-gateway", "10.0.1.11")
instances = registre.decouvrir("api-gateway")
print(f"  Instances saines restantes : {len(instances)}")
for s in instances:
    print(f"    → {s['adresse']}:{s['port']}")
{{< /pyodide >}}

## Collaborative/replicated data (CRDTs & offline-first)

Les *données collaboratives et répliquées* désignent un paradigme où plusieurs
utilisateurs ou appareils peuvent modifier simultanément les mêmes données —
même sans connexion réseau — avec la garantie que toutes les copies convergeront
automatiquement vers un état cohérent lorsqu'elles se synchroniseront. C'est le
défi fondamental de l'édition collaborative en temps réel (Google Docs, Figma)
et des applications *offline-first* (qui fonctionnent hors ligne et se
synchronisent plus tard). La solution classique repose sur les **CRDT**
(*Conflict-free Replicated Data Types*), des structures de données
mathématiquement conçues pour que la fusion de modifications concurrentes soit
toujours possible sans conflit, quel que soit l'ordre dans lequel les opérations
arrivent. Les CRDTs existent en deux familles&nbsp;: les CRDTs basés sur l'état
(*state-based* ou CvRDT), où l'on fusionne les états complets via une fonction
de *merge*, et les CRDTs basés sur les opérations (*operation-based* ou CmRDT),
où l'on diffuse et rejoue les opérations individuelles. Parmi les CRDTs les plus
courants, on trouve le compteur G-Counter (croissant seulement), le compteur
PN-Counter (incréments et décréments), l'ensemble G-Set (ajout seulement) et
l'ensemble OR-Set (*Observed-Remove Set*, ajout et suppression).
[Yjs](https://yjs.dev/) et [Automerge](https://automerge.org/) sont les
bibliothèques de référence pour l'édition collaborative, tandis que des bases de
données comme [Riak](https://riak.com/) utilisent les CRDTs nativement pour la
réplication multi-datacenter. En Python, on peut implémenter quelques CRDTs
fondamentaux pour illustrer comment la convergence automatique fonctionne.

{{< pyodide >}}
# --- G-Counter : compteur distribué (croissant seulement) ---

class GCounter:
    """
    Grow-only Counter — chaque nœud maintient son propre compteur.
    La valeur globale est la somme de tous les compteurs.
    La fusion prend le max de chaque compteur (idempotente et commutative).
    """

    def __init__(self, noeud_id, compteurs=None):
        self.noeud_id = noeud_id
        self.compteurs = dict(compteurs) if compteurs else {}

    def incrementer(self, n=1):
        self.compteurs[self.noeud_id] = self.compteurs.get(self.noeud_id, 0) + n

    def valeur(self):
        return sum(self.compteurs.values())

    def merge(self, autre):
        """Fusionne avec un autre G-Counter (max par nœud)."""
        tous_noeuds = set(self.compteurs) | set(autre.compteurs)
        fusionne = {}
        for n in tous_noeuds:
            fusionne[n] = max(self.compteurs.get(n, 0), autre.compteurs.get(n, 0))
        return GCounter(self.noeud_id, fusionne)

    def __repr__(self):
        return f"GCounter({self.compteurs}) = {self.valeur()}"

# Simulation : 3 serveurs comptent des visites indépendamment

print("=== G-Counter : compteur de visites distribué ===\n")

serveur_a = GCounter("A")
serveur_b = GCounter("B")
serveur_c = GCounter("C")

# Chaque serveur reçoit des visites indépendamment
serveur_a.incrementer(5)
serveur_b.incrementer(3)
serveur_c.incrementer(7)
serveur_a.incrementer(2)  # A reçoit encore des visites

print(f"  Serveur A (local) : {serveur_a}")
print(f"  Serveur B (local) : {serveur_b}")
print(f"  Serveur C (local) : {serveur_c}")

# Synchronisation : fusion des états
total = serveur_a.merge(serveur_b).merge(serveur_c)
print(f"\n  Après fusion      : {total}")

# --- OR-Set : ensemble avec ajout ET suppression ---

print(f"\n=== OR-Set : panier d'achat collaboratif ===\n")

class ORSet:
    """
    Observed-Remove Set — supporte ajout et suppression sans conflit.
    Chaque ajout reçoit un tag unique ; la suppression retire les tags observés.
    """

    def __init__(self, noeud_id):
        self.noeud_id = noeud_id
        self.elements = {}  # element → set(tags)
        self.compteur = 0

    def _tag(self):
        self.compteur += 1
        return f"{self.noeud_id}:{self.compteur}"

    def ajouter(self, element):
        tag = self._tag()
        self.elements.setdefault(element, set()).add(tag)
        print(f"  [{self.noeud_id}] + {element}  (tag={tag})")

    def supprimer(self, element):
        if element in self.elements:
            tags = self.elements.pop(element)
            print(f"  [{self.noeud_id}] - {element}  (tags supprimés: {tags})")

    def contenu(self):
        return {e for e, tags in self.elements.items() if tags}

    def merge(self, autre):
        """Fusionne : union des tags pour chaque élément."""
        resultat = ORSet(self.noeud_id)
        resultat.compteur = max(self.compteur, autre.compteur)
        tous = set(self.elements) | set(autre.elements)
        for e in tous:
            tags = self.elements.get(e, set()) | autre.elements.get(e, set())
            if tags:
                resultat.elements[e] = set(tags)
        return resultat

    def __repr__(self):
        return f"ORSet({sorted(self.contenu())})"

# Alice et Bob modifient le même panier hors ligne

alice = ORSet("Alice")
bob = ORSet("Bob")

alice.ajouter("Lait")
alice.ajouter("Pain")
alice.ajouter("Beurre")

print()
bob.ajouter("Lait")
bob.ajouter("Œufs")
bob.supprimer("Lait")  # Bob retire le lait

print(f"\n  Panier Alice : {alice}")
print(f"  Panier Bob   : {bob}")

# Synchronisation
fusionne = alice.merge(bob)
print(f"\n  Après fusion  : {fusionne}")
print(f"  → Le lait est présent car Alice l'a ajouté avec un tag")
print(f"    que Bob n'avait pas observé lors de sa suppression")
{{< /pyodide >}}

## Ledgers / blockchain

Un *ledger* (grand livre) est un registre chronologique et immuable de
transactions — exactement comme le grand livre comptable utilisé depuis des
siècles, où chaque écriture est datée, signée et ne peut être ni effacée ni
modifiée après coup. Dans un contexte de gestion des données, l'immuabilité est
une propriété puissante&nbsp;: plutôt que de mettre à jour un solde en écrasant
l'ancienne valeur (comme dans une base de données classique), un ledger conserve
l'historique complet de toutes les opérations, et l'état courant se déduit en
rejouant la séquence des transactions. Cette approche est à la base de la
comptabilité en partie double, du *event sourcing* en architecture logicielle,
et bien sûr de la **blockchain**. Une blockchain est un ledger distribué où les
transactions sont regroupées en blocs, chaque bloc contenant le *hash*
cryptographique du bloc précédent, formant ainsi une chaîne dont toute
modification rétroactive serait immédiatement détectable.
[Bitcoin](https://bitcoin.org/) et [Ethereum](https://ethereum.org/) sont les
blockchains publiques les plus connues, tandis que [Hyperledger
Fabric](https://www.hyperledger.org/projects/fabric) cible les consortiums
d'entreprises. [Amazon QLDB](https://aws.amazon.com/qldb/) offre un ledger
centralisé (sans consensus distribué) avec vérification cryptographique, utile
quand on veut l'immuabilité sans la complexité d'une blockchain. En Python, on
peut implémenter un mini-ledger avec chaînage cryptographique pour illustrer les
principes fondamentaux.

{{< pyodide >}}
import hashlib
import json
from datetime import datetime

# --- Bloc : unité de base de la chaîne ---

class Bloc:
    def __init__(self, index, transactions, hash_precedent):
        self.index = index
        self.horodatage = datetime.now().isoformat()
        self.transactions = transactions
        self.hash_precedent = hash_precedent
        self.hash = self.calculer_hash()

    def calculer_hash(self):
        """Hash SHA-256 du contenu du bloc (immuabilité cryptographique)."""
        contenu = json.dumps({
            "index": self.index,
            "horodatage": self.horodatage,
            "transactions": self.transactions,
            "hash_precedent": self.hash_precedent,
        }, sort_keys=True)
        return hashlib.sha256(contenu.encode()).hexdigest()

# --- Ledger : chaîne de blocs ---

class Ledger:
    def __init__(self):
        # Bloc genesis (premier bloc, sans prédécesseur)
        genesis = Bloc(0, [{"type": "GENESIS", "description": "Création du ledger"}], "0" * 64)
        self.chaine = [genesis]
        self.transactions_en_attente = []

    def ajouter_transaction(self, transaction):
        """Ajoute une transaction au pool en attente."""
        transaction["horodatage"] = datetime.now().isoformat()
        self.transactions_en_attente.append(transaction)

    def miner_bloc(self):
        """Regroupe les transactions en attente dans un nouveau bloc."""
        if not self.transactions_en_attente:
            print("  Aucune transaction en attente.")
            return
        nouveau = Bloc(
            index=len(self.chaine),
            transactions=self.transactions_en_attente,
            hash_precedent=self.chaine[-1].hash,
        )
        self.chaine.append(nouveau)
        n = len(self.transactions_en_attente)
        self.transactions_en_attente = []
        print(f"  Bloc #{nouveau.index} miné ({n} transaction(s)), hash: {nouveau.hash[:16]}...")

    def verifier_integrite(self):
        """Vérifie que la chaîne n'a pas été altérée."""
        for i in range(1, len(self.chaine)):
            bloc = self.chaine[i]
            precedent = self.chaine[i - 1]
            # Vérifier le chaînage
            if bloc.hash_precedent != precedent.hash:
                return False, f"Bloc #{i} : hash_precedent ne correspond pas"
            # Vérifier l'intégrité du bloc lui-même
            if bloc.hash != bloc.calculer_hash():
                return False, f"Bloc #{i} : hash invalide (données altérées)"
        return True, "Chaîne intègre"

    def solde(self, compte):
        """Calcule le solde d'un compte en rejouant toutes les transactions."""
        total = 0.0
        for bloc in self.chaine:
            for tx in bloc.transactions:
                if tx.get("de") == compte:
                    total -= tx.get("montant", 0)
                if tx.get("a") == compte:
                    total += tx.get("montant", 0)
        return total

# --- Démonstration ---

ledger = Ledger()

print("=== Ajout de transactions ===\n")

# Dépôts initiaux
ledger.ajouter_transaction({"type": "DEPOT", "a": "Alice", "montant": 1000, "description": "Dépôt initial"})
ledger.ajouter_transaction({"type": "DEPOT", "a": "Bob", "montant": 500, "description": "Dépôt initial"})
ledger.miner_bloc()

# Transferts
ledger.ajouter_transaction({"type": "TRANSFERT", "de": "Alice", "a": "Bob", "montant": 150, "description": "Paiement facture"})
ledger.ajouter_transaction({"type": "TRANSFERT", "de": "Bob", "a": "Charlie", "montant": 75, "description": "Remboursement"})
ledger.miner_bloc()

ledger.ajouter_transaction({"type": "TRANSFERT", "de": "Alice", "a": "Charlie", "montant": 200, "description": "Achat service"})
ledger.miner_bloc()

print(f"\n=== État de la chaîne ({len(ledger.chaine)} blocs) ===\n")
for bloc in ledger.chaine:
    print(f"  Bloc #{bloc.index}  hash: {bloc.hash[:16]}...  prev: {bloc.hash_precedent[:16]}...")
    for tx in bloc.transactions:
        desc = tx.get("description", "")
        montant = tx.get("montant", "")
        print(f"    → {tx['type']:10s} {desc}  {f'{montant} $' if montant else ''}")

print(f"\n=== Soldes (calculés par replay) ===\n")
for compte in ["Alice", "Bob", "Charlie"]:
    print(f"  {compte:10s} : {ledger.solde(compte):8.2f} $")

print(f"\n=== Vérification d'intégrité ===\n")
valide, message = ledger.verifier_integrite()
print(f"  {message}")

# Tentative de falsification
print(f"\n=== Tentative de falsification ===\n")
ledger.chaine[1].transactions[0]["montant"] = 999999
print(f"  Bloc #1 modifié : montant changé à 999999 $")
valide, message = ledger.verifier_integrite()
print(f"  Vérification : {message}")
{{< /pyodide >}}

## Vector databases

Les *bases de données vectorielles* (*vector databases*) sont un paradigme de
stockage et de recherche conçu pour manipuler des **vecteurs d'embedding** — des
représentations numériques de haute dimension (typiquement 256 à 4096
dimensions) produites par des modèles d'apprentissage automatique pour encoder
le « sens » d'un texte, d'une image, d'un son ou de toute autre donnée.
L'opération fondamentale est la **recherche par similarité**&nbsp;: plutôt que de
chercher une correspondance exacte (comme en SQL avec `WHERE nom = 'Alice'`), on
cherche les vecteurs les plus *proches* d'un vecteur requête selon une mesure de
distance (cosinus, euclidienne, produit scalaire). C'est cette capacité qui rend
possible la recherche sémantique (« trouver des documents qui *parlent* de la
même chose, même avec des mots différents »), les systèmes de recommandation et
le *RAG* (*Retrieval-Augmented Generation*) qui alimente les assistants IA
modernes. [Pinecone](https://www.pinecone.io/),
[Weaviate](https://weaviate.io/), [Milvus](https://milvus.io/) et
[Qdrant](https://qdrant.tech/) sont des bases vectorielles spécialisées, tandis
que [pgvector](https://github.com/pgvector/pgvector) ajoute cette capacité à
PostgreSQL. Le défi technique principal est la recherche approximative des plus
proches voisins (*ANN — Approximate Nearest Neighbors*), car une recherche
exacte en haute dimension est prohibitivement lente ; des algorithmes comme HNSW
(*Hierarchical Navigable Small World*) permettent des recherches quasi
instantanées même sur des millions de vecteurs. En Python, on peut illustrer les
concepts fondamentaux — embedding, distance cosinus, recherche par similarité —
avec des vecteurs simplifiés.

{{< pyodide >}}
import math
import random

# --- Similarité cosinus ---

def similarite_cosinus(a, b):
    """Mesure la similarité entre deux vecteurs (1 = identiques, 0 = orthogonaux)."""
    produit = sum(x * y for x, y in zip(a, b))
    norme_a = math.sqrt(sum(x * x for x in a))
    norme_b = math.sqrt(sum(x * x for x in b))
    if norme_a == 0 or norme_b == 0:
        return 0.0
    return produit / (norme_a * norme_b)

# --- Simulacre d'embedding : vecteur thématique simplifié ---
# Dimensions : [informatique, cuisine, sport, musique, science]

def embedding(themes):
    """Crée un vecteur d'embedding simplifié à partir de scores thématiques."""
    random.seed(hash(tuple(themes)) % 2**32)
    # Ajouter du bruit pour simuler un vrai modèle
    return [v + random.gauss(0, 0.1) for v in themes]

# --- Base de données vectorielle minimale ---

class BaseVectorielle:
    def __init__(self):
        self.documents = []  # [{"id": ..., "texte": ..., "vecteur": [...]}]

    def inserer(self, id_doc, texte, vecteur):
        self.documents.append({"id": id_doc, "texte": texte, "vecteur": vecteur})

    def rechercher(self, vecteur_requete, k=3):
        """Recherche les k documents les plus similaires (force brute)."""
        scores = []
        for doc in self.documents:
            sim = similarite_cosinus(vecteur_requete, doc["vecteur"])
            scores.append((doc, sim))
        scores.sort(key=lambda x: x[1], reverse=True)
        return scores[:k]

# --- Construction de la base ---

db = BaseVectorielle()

corpus = [
    ("doc1", "Introduction à Python et aux algorithmes",         [0.9, 0.0, 0.0, 0.0, 0.3]),
    ("doc2", "Recette de gâteau au chocolat",                    [0.0, 0.9, 0.0, 0.0, 0.0]),
    ("doc3", "Les réseaux de neurones en intelligence artificielle", [0.8, 0.0, 0.0, 0.0, 0.9]),
    ("doc4", "Entraînement de marathon et nutrition sportive",   [0.0, 0.3, 0.9, 0.0, 0.1]),
    ("doc5", "Théorie musicale et composition assistée par IA",  [0.4, 0.0, 0.0, 0.9, 0.2]),
    ("doc6", "Analyse de données avec pandas et matplotlib",     [0.8, 0.0, 0.0, 0.0, 0.5]),
    ("doc7", "Cuisine moléculaire et science des aliments",      [0.0, 0.8, 0.0, 0.0, 0.7]),
    ("doc8", "Statistiques sportives et apprentissage automatique", [0.6, 0.0, 0.7, 0.0, 0.5]),
]

print("=== Insertion des documents ===\n")
for id_doc, texte, themes in corpus:
    vecteur = embedding(themes)
    db.inserer(id_doc, texte, vecteur)
    print(f"  {id_doc}: {texte[:50]}...")

# --- Recherches par similarité ---

requetes = [
    ("Programmation et science",     [0.8, 0.0, 0.0, 0.0, 0.7]),
    ("Recettes et alimentation",     [0.0, 0.9, 0.0, 0.0, 0.1]),
    ("Sport et données",             [0.4, 0.0, 0.8, 0.0, 0.3]),
]

for description, themes_requete in requetes:
    vecteur_q = embedding(themes_requete)
    resultats = db.rechercher(vecteur_q, k=3)

    print(f"\n=== Recherche : '{description}' ===\n")
    for doc, score in resultats:
        print(f"  {score:.3f}  {doc['id']}: {doc['texte'][:55]}...")
{{< /pyodide >}}
