#  TP 03 - Python : Scanner de port

| Propriétés | Valeurs |
|---|---|
| **Tags** | B3, Python |
| **Bloc** | Bloc 3 |
| **Compétence du référentiel** | B3.5.4 - SLAM - Prévenir les attaques |

---

##  Objectifs

Un **scanner de port** est utilisé pour vérifier quels ports sont ouverts et prêts à
recevoir des connexions sur un hôte distant.

Dans ce TP, on réalise un scanner de ports en Python en découvrant trois notions :

- le module **`socket`** : établir des connexions réseau sur les ports ;
- le module **`threading`** : créer et gérer des threads (processus légers) qui
  balayent les ports de manière parallèle ;
- la classe **`Queue`** (module `queue`) : gérer une file d'attente qui contient
  les ports à scanner.

##  Avertissement

> L'utilisation de scanners de ports sur des systèmes distants **sans autorisation
> préalable** peut être contraire à la loi. Vous devez toujours obtenir une
> **autorisation explicite** avant de scanner des ports sur un réseau qui ne vous
> appartient pas.
>
> La tentative d'accès frauduleux à un système informatique est expressément
> incriminée par l'**article 323-7 du Code pénal français** : la tentative est
> punie des mêmes peines que l'infraction consommée.

##  Contenu du projet

| Fichier | Rôle |
|---|---|
| `scanner.py` | Le script du scanner de ports |
| `README.md` | Documentation du TP |

##  Prérequis

- **Python 3** installé (aucune bibliothèque externe : `socket`, `threading` et
  `queue` sont des modules de la bibliothèque standard).

##  Fonctionnement du script

### 1. Les imports

```python
import socket
import threading
from queue import Queue
```

- **`socket`** : utilisé pour effectuer des opérations réseau, notamment pour
  établir des connexions sur les ports.
- **`threading`** : permet de créer et de gérer des threads qui effectueront le
  balayage des ports de manière parallèle.
- **`Queue`** : gère une file d'attente qui contiendra les ports à scanner.

### 2. Déclaration des variables

```python
# Demande à l'utilisateur
target = input("Entrer l'adresse IP de la cible : ")

# Création de la queue
queue = Queue()

# Liste des ports ouverts
open_ports = []
```

- `target` : l'adresse IP de la cible saisie par l'utilisateur, qui sera scannée
  pour déterminer quels ports sont ouverts.
- `queue` : la file d'attente qui stocke les numéros de port à scanner.
- `open_ports` : la liste vide qui stockera les numéros de port ouverts.

### 3. La fonction `port_scan`

```python
def port_scan(port):
    try:
        # Configuration de socket
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        # Connexion à la cible sur le port passé en paramètre
        sock.connect((target, port))
        return True
    except:
        return False
```

- Prend en paramètre un numéro de port.
- Tente une connexion au port passé en paramètre.
- Renvoie `True` en cas de réussite.
- Renvoie `False` en cas d'échec (port fermé ou inaccessible).

### 4. La fonction `fill_queue`

```python
def fill_queue(port_list):
    for port in port_list:
        queue.put(port)
```

- Prend en paramètre une liste de ports.
- Parcourt la liste et ajoute chaque port à la queue.

### 5. La fonction `executor`

```python
def executor():
    while not queue.empty():
        port = queue.get()
        if port_scan(port):
            print("Le port {} est ouvert".format(port))
            open_ports.append(port)
```

- Parcourt la queue jusqu'au bout (`empty`).
- Pour chaque élément de la queue : récupération du port, puis appel de
  `port_scan` avec ce port.
- Si `port_scan` renvoie `True` : affichage `Le port XX est ouvert` et ajout du
  port dans la liste `open_ports`.

### 6. Lancement des threads et affichage des ports

```python
# Liste des ports de 1 à 1024
port_list = range(1, 1024)

# Appel de la fonction fill_queue
fill_queue(port_list)

# Stockage des threads dans une liste
thread_list = []

for t in range(500):
    # Définition de la fonction exécutée par le thread
    thread = threading.Thread(target=executor)
    # Ajout du thread à thread_list
    thread_list.append(thread)

for thread in thread_list:
    # Lancement du thread
    thread.start()

for thread in thread_list:
    # Attend que le thread soit terminé
    thread.join()

print("Les ports ouverts sont : ", open_ports)
```

- Les numéros de port de la liste `port_list` sont ajoutés à la file d'attente
  avec `fill_queue(port_list)`.
- Un total de **500 threads** est créé, chacun exécutant la fonction `executor()`.
- Chaque thread est démarré avec `thread.start()`.
- `thread.join()` attend que tous les threads aient terminé leur exécution.
- Une fois tous les threads terminés, le programme affiche la liste des ports
  ouverts stockée dans `open_ports`.

##  Utilisation

```bash
python3 scanner.py
```

Le script demande l'adresse IP de la cible :

```
Entrer l'adresse IP de la cible : 
```

## 🖥️ Résultats sur ma propre machine (172.16.191.254)

Exécution du script ciblant `172.16.191.254` :

```
Entrer l'adresse IP de la cible : 172.16.191.254
Les ports ouverts sont :  []
```

**Quels sont les résultats qui apparaissent ?**

Le script affiche les ports 80 et 22 ouvert sur la machine cible.

##  Test sur la VM d'un camarade

> *À compléter après le test en salle :*
>
> - IP de la VM scannée : `172.16.200.254`
> - Ports ouverts trouvés : `22 et 80`

