# Animap Tracker — Application mobile Flutter

> Projet de fin d'étude — BTS CIEL (Cybersécurité, Informatique et réseaux, ELectronique)
> Janvier 2025 — Juin 2025

Application mobile de suivi GPS d'animaux en temps réel, développée avec Flutter. Elle constitue la partie interface utilisateur principale du projet Animap Tracker.

## Fonctionnalités

- Affichage en temps réel des positions GPS des animaux sur une carte OpenStreetMap
- Historique des déplacements par animal
- Système d'alertes
- Gestion de compte utilisateur (inscription / connexion JWT)
- Disponible en **français**, **anglais** et **breton**

## Prérequis

- Flutter 3.0+
- Dart 3.0+
- Un émulateur Android/iOS ou un appareil physique connecté

## Installation et lancement

```bash
# Installer les dépendances
flutter pub get

# Lancer en mode développement
flutter run

# Compiler un APK Android (release)
flutter build apk --release

# Compiler pour iOS (macOS requis)
flutter build ios --release
```

## Structure

```
lib/
├── main.dart          # Point d'entrée + navigation principale
├── pages/             # Écrans de l'application
│   ├── home.dart
│   ├── map.dart       # Carte interactive (flutter_map + OpenStreetMap)
│   ├── list.dart      # Liste des animaux suivis
│   ├── alert.dart     # Alertes
│   ├── account.dart   # Gestion du compte
│   └── login.dart     # Authentification
└── l10n/              # Fichiers de traduction (FR, EN, Breton)
```

## Dépendances principales

| Package | Rôle |
|---|---|
| `flutter_map` | Affichage carte OpenStreetMap |
| `location` | Accès GPS de l'appareil |
| `provider` | Gestion d'état |
| `http` | Requêtes vers l'API Django |
| `webview_flutter` | Affichage web embarqué |
