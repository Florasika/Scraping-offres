#  Day 05 — Scraping d'Offres d'Emploi avec Python

> **Série : 30 Days of Python for Data** · Jour 5/30  
> Publié sur [LinkedIn]() 

---

##  Ce que tu vas apprendre

| Concept | Fonction | Utilité |
|---------|----------|---------|
| Requêtes HTTP | `requests.get()` | Récupérer le contenu d'une page web |
| Parsing HTML | `BeautifulSoup` | Naviguer et extraire des données HTML |
| Lecture / écriture | `json.dump()` / `json.load()` | Sauvegarder et relire des données JSON |
| Analyse | `pd.DataFrame()` / `groupby()` | Transformer et analyser les données |

---

##  Structure du projet

```
day-05-scraping-offres/
│
├── scraping_offres.py       ← Script principal
├── offres_emploi.json       ← Dataset généré automatiquement
├── dashboard_offres.png     ← Dashboard exporté
└── README.md
```

---

##  Installation & Lancement

```bash
pip install requests beautifulsoup4 pandas matplotlib
python scraping_offres.py
```

---

##  Explication détaillée du code

###  Les imports

```python
import requests
from bs4 import BeautifulSoup
import json
import pandas as pd
import matplotlib.pyplot as plt
```

**`requests`** — La bibliothèque qui permet à Python de naviguer sur internet. Elle envoie des requêtes HTTP exactement comme ton navigateur, sauf que c'est du code. Sans elle, Python ne sait pas parler à internet.

**`from bs4 import BeautifulSoup`** — On importe uniquement `BeautifulSoup` depuis la bibliothèque `bs4`. C'est le parser HTML — il prend du code HTML brut et le rend navigable. Le nom vient d'une expression anglaise pour désigner une soupe de balises HTML mal écrites.

**`json`** — Module natif Python (pas besoin de l'installer). Il sert à lire et écrire des fichiers JSON — le format universel des APIs.

**`random` et `datetime`** — Pour générer des données réalistes. `random.seed(42)` garantit que les mêmes données sont générées à chaque exécution — important pour la reproductibilité.

---

###  Partie 1 — requests

```python
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)...'
}

response = requests.get(url, headers=headers, timeout=10)
response.raise_for_status()
```

**`headers`** — Un dictionnaire qui simule un vrai navigateur. Sans ça, beaucoup de sites détectent que c'est un bot et bloquent la requête avec une erreur 403. Le `User-Agent` dit au serveur *"je suis un navigateur Chrome sur Windows"*.

**`requests.get(url, headers=headers, timeout=10)`** — Trois paramètres importants :
- `url` : l'adresse à appeler
- `headers` : les informations qu'on envoie au serveur
- `timeout=10` : si le serveur ne répond pas en 10 secondes, on arrête. Sans ça, le script peut rester bloqué indéfiniment.

**`response.raise_for_status()`** — Vérifie que la requête a réussi. Si le serveur retourne une erreur (404, 500, 403...), cette ligne lève automatiquement une exception au lieu de continuer avec une réponse vide. C'est une bonne pratique systématique.

**`len(response.text)`** — `response.text` c'est le HTML brut de la page, sous forme de chaîne de caractères. On affiche sa taille pour vérifier qu'on a bien reçu quelque chose.

**`try / except`** — Bloc de gestion d'erreur. Si la connexion échoue (pas internet, site bloqué, timeout), le `except` attrape l'erreur et continue proprement au lieu de planter le script entier.

---

###  Partie 2 — BeautifulSoup

```python
soup = BeautifulSoup(response.text, 'html.parser')
titres = soup.find_all('h3')
```

**`BeautifulSoup(response.text, 'html.parser')`** — On passe deux choses : le HTML brut et le moteur de parsing. `html.parser` est le parser natif Python. Il existe aussi `lxml` (plus rapide) et `html5lib` (plus tolérant).

**`soup.find_all('h3')`** — Cherche toutes les balises `<h3>` dans la page et les retourne sous forme de liste. Tu peux chercher n'importe quelle balise HTML : `find_all('div')`, `find_all('a')`, `find_all('span')`.

```python
soup_demo.find_all('div', class_='offre')
```

**`class_='offre'`** — Filtre par classe CSS. Le `_` après `class` est obligatoire parce que `class` est un mot réservé en Python. C'est comme faire un CTRL+F sur le HTML mais en ciblant exactement la bonne structure.

```python
o.find('h2').get_text()
o.find('span', class_='lieu').get_text()
```

**`.find()`** — Retourne le *premier* élément qui correspond (contrairement à `find_all` qui retourne *tous*).

**`.get_text()`** — Extrait uniquement le texte visible, sans les balises HTML. `<h2>Data Analyst</h2>` → `"Data Analyst"`. On peut ajouter `strip=True` pour supprimer les espaces autour.

---

###  Partie 3 — Générer les données JSON

```python
random.seed(42)
```

**`seed(42)`** — Initialise le générateur aléatoire avec une valeur fixe. Résultat : à chaque exécution, les mêmes nombres sont générés dans le même ordre. Sans seed, tu obtiens des données différentes à chaque fois — impraticable pour un tutoriel reproductible.

```python
salaires = {
    'Junior (0-2 ans)': (32000, 42000),
    'Mid (2-5 ans)':    (42000, 58000),
    'Senior (5+ ans)':  (58000, 80000),
}
```

**Dictionnaire de tuples** — On associe chaque niveau à une fourchette `(min, max)`. On l'utilise ensuite avec `salaires[niveau][0]` pour le min et `salaires[niveau][1]` pour le max. C'est plus propre qu'une série de `if/elif`.

```python
random.randint(sal_min, sal_max)
random.choice(postes)
random.sample(competences, random.randint(2, 4))
```

**`randint(a, b)`** — Entier aléatoire entre a et b inclus.

**`choice(liste)`** — Pioche un élément aléatoire dans une liste.

**`sample(liste, k)`** — Pioche k éléments sans remise (pas de doublon). Ici entre 2 et 4 compétences par offre.

```python
(date_base - timedelta(days=random.randint(0, 30))).strftime('%Y-%m-%d')
```

**`timedelta(days=n)`** — Soustrait n jours à une date. Ça génère des dates réparties sur les 30 derniers jours.

**`strftime('%Y-%m-%d')`** — Formate la date en chaîne de caractères. `%Y` = année 4 chiffres, `%m` = mois 2 chiffres, `%d` = jour 2 chiffres. Résultat : `"2024-05-10"`.

---

###  Partie 4 — Sauvegarder et relire en JSON

```python
with open('offres_emploi.json', 'w', encoding='utf-8') as f:
    json.dump(offres, f, ensure_ascii=False, indent=2)
```

**`with open(...) as f`** — Ouvre le fichier et le ferme automatiquement à la fin du bloc, même si une erreur survient. C'est la bonne façon de manipuler des fichiers en Python.

**`'w'`** — Mode écriture (write). Écrase le fichier s'il existe déjà. Pour ajouter à la suite, on utiliserait `'a'` (append).

**`ensure_ascii=False`** — Autorise les caractères accentués (é, è, ê...). Sans ça, `"Île-de-France"` deviendrait `"\u00cele-de-France"`.

**`indent=2`** — Formate le JSON avec une indentation de 2 espaces. Sans ça, tout serait sur une seule ligne illisible.

```python
with open('offres_emploi.json', encoding='utf-8') as f:
    data = json.load(f)

df = pd.DataFrame(data)
```

**`json.load(f)`** — Lit le fichier JSON et le convertit en objet Python (ici une liste de dictionnaires).

**`pd.DataFrame(data)`** — Convertit une liste de dictionnaires en DataFrame. Chaque dictionnaire devient une ligne, chaque clé devient une colonne. C'est la conversion la plus fréquente quand tu travailles avec des APIs.

---

###  Partie 5 — L'analyse pandas

```python
df['poste'].value_counts()
```

**`value_counts()`** — Compte les occurrences de chaque valeur unique dans une colonne et trie par ordre décroissant. C'est le réflexe numéro 1 pour comprendre une colonne catégorielle.

```python
df.groupby('poste')['salaire'].mean().sort_values(ascending=False).round(0)
```

Chaîne de 4 méthodes enchaînées — c'est le style pandas qu'on appelle *method chaining* :

| Méthode | Rôle |
|---------|------|
| `groupby('poste')` | Groupe les lignes par poste |
| `['salaire']` | Sélectionne uniquement la colonne salaire |
| `.mean()` | Calcule la moyenne par groupe |
| `.sort_values(ascending=False)` | Trie du plus élevé au plus bas |
| `.round(0)` | Arrondit à l'entier |

---

###  Partie 6 — La visualisation

```python
ax3.xaxis.set_major_formatter(plt.FuncFormatter(lambda v, _: f'{v/1000:.0f}k€'))
```

**`FuncFormatter`** — Applique une fonction personnalisée à chaque label de l'axe. Ici on divise la valeur par 1000 et on ajoute `k€`. Le `_` est le second paramètre (position) qu'on ignore. Résultat : `47000` devient `47k€` sur l'axe.

---

##  Résumé en une phrase

> *"Le scraping c'est trois étapes : `requests.get()` pour récupérer le HTML, `BeautifulSoup` pour le parser, `get_text()` pour extraire le texte. Et quand tu travailles avec une API, c'est encore plus simple — `requests` retourne du JSON, `json.load()` le transforme en dictionnaire, `pd.DataFrame()` en tableau. De là, tu analyses comme d'habitude."*

---

##  Prochains projets

| Jour | Projet | Concepts |
|------|--------|----------|
| **Jour 6** | Analyse de Texte & WordCloud | `re`, `Counter`, `WordCloud` |
| **Jour 7** | Régression Linéaire | `scikit-learn`, `LinearRegression` |
| **Jour 8** | Classification Spam/Ham | `TfidfVectorizer`, `LogisticRegression` |

---


---

⭐ **Si ce projet t'aide, mets une étoile sur le repo !**

<img width="2084" height="1330" alt="dashboard_offres_emploi" src="https://github.com/user-attachments/assets/0a54a8bb-156b-4839-87a4-162a5b28e8d6" />
