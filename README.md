# Animap Tracker — Système de suivi GPS d'animaux

> Projet de fin d'étude — BTS CIEL (Cybersécurité, Informatique et réseaux, ELectronique)

Projet collaboratif de suivi GPS d'animaux en temps réel, développé par 4 étudiants. Le système combine du matériel embarqué (collier GPS), une infrastructure cloud IoT (LoRaWAN / TTN), des backends REST, et plusieurs interfaces utilisateurs (application mobile Flutter et dashboard web Vue.js).

---

## Table des matières

- [Architecture générale](#architecture-générale)
- [Structure du dépôt](#structure-du-dépôt)
- [Flux de données](#flux-de-données)
- [Technologies utilisées](#technologies-utilisées)
- [Prérequis](#prérequis)
- [Installation et lancement](#installation-et-lancement)
  - [1. Firmware embarqué (Fabien)](#1-firmware-embarqué-fabien)
  - [2. Backend Django (Matts)](#2-backend-django-matts)
  - [3. Frontend Vue.js (Matts)](#3-frontend-vuejs-matts)
  - [4. Serveur Express / Carte (Mateo)](#4-serveur-express--carte-mateo)
  - [5. Application mobile Flutter (Simon)](#5-application-mobile-flutter-simon)
- [API REST](#api-rest)
- [Schéma de base de données](#schéma-de-base-de-données)
- [Configuration](#configuration)

---

## Architecture générale

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      GPS ANIMAL TRACKING SYSTEM                         │
│                                                                         │
│  ┌──────────────┐    LoRaWAN     ┌──────────────────┐                  │
│  │ Collier GPS  │ ─────────────► │ The Things       │                  │
│  │ CubeCell GPS │                │ Network (TTN)    │                  │
│  │ Air530Z      │                │ MQTT Broker      │                  │
│  └──────────────┘                └────────┬─────────┘                  │
│                                           │ MQTT                        │
│                                  ┌────────▼─────────┐                  │
│                                  │  Django REST API  │                  │
│                                  │  + MQTT Client    │                  │
│                                  │  PostgreSQL+GIS   │                  │
│                                  └────────┬─────────┘                  │
│                                           │ HTTP                        │
│                        ┌──────────────────┴──────────────────┐         │
│                        │                                      │         │
│               ┌────────▼────────┐                ┌───────────▼──────┐  │
│               │  App Flutter    │                │  Dashboard Vue   │  │
│               │  (iOS/Android)  │                │  + Express.js    │  │
│               └─────────────────┘                └──────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Structure du dépôt

```
Projet_2024/
├── Fabien/                                  # Firmware embarqué
│   ├── Animal-location-sending-GPS/         # Firmware principal (PlatformIO)
│   │   ├── src/
│   │   │   ├── main.cpp                     # Boucle principale : lecture GPS + envoi LoRaWAN
│   │   │   └── flagSettings.cpp             # Feature flags de transmission
│   │   └── platformio.ini                   # Config board Heltec CubeCell GPS
│   ├── Animal-Location-RawData-GPS/         # Variante : journalisation brute
│   └── fnTests/                             # Fonctions de test
│
├── Matts/                                   # Backend Django & frontend Vue
│   ├── Panimal/                             # Projet Django
│   │   ├── manage.py
│   │   ├── Panimal/
│   │   │   ├── settings.py                  # Config Django (BDD, JWT, CORS)
│   │   │   └── urls.py                      # Routage URL
│   │   └── mqtt_app/                        # Application principale
│   │       ├── models.py                    # Modèles : Compte, Objet, Donnee, Affectation
│   │       ├── views.py                     # Endpoints REST API
│   │       ├── serializers.py               # Sérialisation des données
│   │       └── mqtt_client.py               # Client MQTT (connexion TTN)
│   └── panimalvue/                          # Frontend Vue.js
│       ├── src/
│       │   ├── App.vue                      # Navbar + layout global
│       │   ├── router/                      # Vue Router
│       │   ├── views/                       # Pages (Home, Alert, List, Maps, Settings)
│       │   └── components/                  # Composants réutilisables
│       └── package.json
│
├── Mateo/                                   # Visualisation carte & serveur Node
│   ├── sans django/                         # Projet actif
│   │   ├── server.js                        # Serveur Express (port 8000)
│   │   ├── index.html                       # Interface carte Leaflet
│   │   ├── map.js                           # Logique carte (stats, export GeoJSON)
│   │   ├── style.css
│   │   └── package.json
│   └── Projet_3/                            # Implémentation alternative
│
└── Simon/                                   # Application mobile Flutter
    ├── animap_tracker/                      # App principale
    │   ├── lib/
    │   │   ├── main.dart                    # Point d'entrée + navigation
    │   │   ├── pages/                       # Écrans (home, map, list, alert, account, login)
    │   │   └── l10n/                        # Traductions (FR, EN, Breton)
    │   └── pubspec.yaml                     # Dépendances Flutter
    └── demo_test/                           # Projet de test/démo
```

---

## Flux de données

1. **Collecte** — Le collier GPS (Heltec CubeCell + Air530Z) lit la position toutes les 5 minutes et encode les données (cardID, latitude, longitude, altitude) en un tableau de 12 octets.
2. **Transmission** — Les données sont envoyées par radio **LoRaWAN** vers le réseau **The Things Network (TTN)**.
3. **Réception** — Le client **MQTT** du backend Django est abonné aux topics TTN. Il décode le payload LoRaWAN et stocke le point GPS dans **PostgreSQL + PostGIS**.
4. **Exposition** — L'**API REST Django** expose les données (filtrage par date, derniers N points, authentification JWT).
5. **Affichage** — L'app **Flutter** et le dashboard **Vue.js** interrogent l'API pour afficher les positions sur une carte **OpenStreetMap / Leaflet**.

---

## Technologies utilisées

| Couche | Technologie |
|---|---|
| Firmware | C++ / Arduino, PlatformIO, Heltec CubeCell GPS, Air530Z |
| Réseau IoT | LoRaWAN, The Things Network, MQTT |
| Backend | Django 5.1, Django REST Framework, Simple JWT |
| Base de données | PostgreSQL + PostGIS |
| Serveur carte | Express.js (Node.js) |
| Frontend web | Vue.js 3, Vue Router, Axios, Leaflet.js |
| Application mobile | Flutter / Dart, flutter_map, provider |
| Cartographie | OpenStreetMap (tuiles gratuites) |

---

## Prérequis

| Outil | Version minimale |
|---|---|
| Python | 3.10+ |
| Node.js | 18+ |
| Flutter | 3.0+ |
| PostgreSQL | 14+ avec extension PostGIS |
| PlatformIO | Dernière version (CLI ou VSCode) |

---

## Installation et lancement

### 1. Firmware embarqué (Fabien)

> Nécessite une carte **Heltec CubeCell GPS** et PlatformIO installé.

```bash
cd Fabien/Animal-location-sending-GPS

# Compiler le firmware
pio run

# Flasher la carte
pio run -t upload

# Surveiller la sortie série
pio device monitor
```

Le firmware envoie les données GPS toutes les **5 minutes** via LoRaWAN. L'identifiant de la carte (`cardID`) permet de distinguer plusieurs colliers.

---

### 2. Backend Django (Matts)

#### Installation des dépendances

```bash
cd Matts/Panimal

pip install django==5.1.5
pip install djangorestframework
pip install djangorestframework-simplejwt
pip install django-cors-headers
pip install psycopg2-binary
pip install paho-mqtt
pip install django.contrib.gis  # Inclus dans Django, nécessite GDAL/GEOS installés
```

> Sur Ubuntu/Debian, installez les librairies système pour PostGIS :
> ```bash
> sudo apt install postgresql postgis gdal-bin libgdal-dev python3-gdal
> ```

#### Configuration de la base de données

```bash
# Créer l'utilisateur et la base
sudo -u postgres psql -c "CREATE USER felix WITH PASSWORD 'felix22';"
sudo -u postgres psql -c "CREATE DATABASE projet2024test OWNER felix;"
sudo -u postgres psql -d projet2024test -c "CREATE EXTENSION postgis;"
```

#### Migrations et démarrage

```bash
python manage.py migrate
python manage.py runserver 0.0.0.0:8000
```

Le serveur Django écoute sur `http://0.0.0.0:8000`. Le client MQTT se connecte automatiquement à TTN au démarrage.

---

### 3. Frontend Vue.js (Matts)

```bash
cd Matts/panimalvue

npm install
npm run serve
```

Accessible sur `http://localhost:8080`.

---

### 4. Serveur Express / Carte (Mateo)

> Ce serveur fournit des données GPS simulées et sert l'interface Leaflet indépendamment de Django.

```bash
cd "Mateo/sans django"

npm install
npm start
```

Accessible sur `http://127.0.0.1:8000`.

Fonctionnalités disponibles :
- Ajout de points GPS par clic sur la carte
- Statistiques : distance totale, vitesse moyenne, durée
- Export au format GeoJSON
- Mode mesure

---

### 5. Application mobile Flutter (Simon)

```bash
cd Simon/animap_tracker

flutter pub get

# Lancer sur un émulateur ou appareil connecté
flutter run

# Compiler un APK Android
flutter build apk --release

# Compiler pour iOS (macOS uniquement)
flutter build ios --release
```

L'application est disponible en **français**, **anglais** et **breton** (géré via `flutter_localizations`).

---

## API REST

Base URL : `http://<host>:8000/api/`

| Méthode | Endpoint | Description | Auth |
|---|---|---|---|
| `POST` | `/register` | Créer un compte utilisateur | Non |
| `POST` | `/login` | Connexion, retourne un token JWT | Non |
| `POST` | `/token/` | Obtenir un token JWT | Non |
| `POST` | `/token/refresh/` | Rafraîchir le token JWT | Non |
| `GET` | `/gps-data/` | Toutes les données GPS (filtrables par date) | Oui |
| `GET` | `/gps-data/<n>` | Les N derniers points GPS | Oui |

### Exemple de requête filtrée par date

```
GET /api/gps-data/?start=2024-01-01&end=2024-12-31
Authorization: Bearer <jwt_token>
```

---

## Schéma de base de données

```
Compte          Objet           Donnee2                 Affectation
──────────      ──────────      ────────────────────    ────────────────
id_user   ←──  id_objet  ←──  id_data                  utilisateur_id
nom             nom_objet       date                     objet_id
mail                            location (POINT/PostGIS)
passwd                          hdop
                                speed
                                course
                                sats
                                id_objet_id
```

---

## Configuration

### The Things Network (MQTT)

Défini dans `Matts/Panimal/mqtt_app/mqtt_client.py` :

| Paramètre | Valeur |
|---|---|
| Broker | `eu1.cloud.thethings.network:8883` |
| Application | `projet-animal-connecte@ttn` |
| Device | `animal-device-prototype-05` |

### Base de données (Django)

Défini dans `Matts/Panimal/Panimal/settings.py` :

| Paramètre | Valeur |
|---|---|
| Nom de la BDD | `projet2024test` |
| Utilisateur | `felix` |
| Mot de passe | `felix22` |
| Hôte | `*` (toutes interfaces) |

### Centre de la carte par défaut

**Lannion, France** — `48.7324°N, 3.4373°W`

---

## Équipe

| Membre | Contribution |
|---|---|
| **Simon** | Application mobile Flutter (iOS/Android), localisation multilingue |
| **Mateo** | Visualisation carte Leaflet, serveur Express.js |
| **Matts / Felix** | Backend Django, API REST, client MQTT, base PostGIS, frontend Vue.js |
| **Fabien** | Firmware embarqué Arduino/PlatformIO, intégration GPS + LoRaWAN |
