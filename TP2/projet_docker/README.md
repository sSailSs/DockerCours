# Projet Docker - Architecture multi-services avec Traefik

## Prérequis
- Docker / Docker Desktop installé
- Les domaines `*.localhost` résolvent vers 127.0.0.1 automatiquement (pas besoin de modifier le fichier hosts sur la plupart des OS).

## Démarrage rapide
Depuis le dossier `projet_docker/` :

```sh
docker compose up -d
```

Arrêt :
```sh
docker compose down
```

## Services
- **Traefik** (reverse proxy) écoute sur le port 80 et route par host (config file provider : `traefik/dynamic.yml`)
- **Site1** : Next.js minimal, accessible sur http://site1.localhost
- **Home Assistant** : accessible sur http://ha.localhost, configuration persistée dans `ha/config`
- **Site3** : AutoPassion (site HTML/CSS sur les voitures), accessible sur http://site3.localhost
- **Site4** : Cosmos (site sur les planètes), accessible sur http://site4.localhost

### 🎬 BONUS : Stack Média "Netflix Maison"

Stack complète de gestion et diffusion multimédia :

- **Jellyfin** : Media server principal - http://jellyfin.localhost
- **Radarr** : Gestion des films - http://radarr.localhost
- **Sonarr** : Gestion des séries - http://sonarr.localhost
- **Lidarr** : Gestion de la musique - http://lidarr.localhost
- **Readarr** : Gestion des livres - http://readarr.localhost
- **Bazarr** : Gestion des sous-titres - http://bazarr.localhost
- **Prowlarr** : Gestion des indexeurs - http://prowlarr.localhost
- **qBittorrent** : Client de téléchargement - http://qbit.localhost

## Première utilisation de Home Assistant
1. Aller sur http://ha.localhost
2. Créer le premier utilisateur et terminer l’onboarding
3. Vérifier la persistance : arrêter/redémarrer le stack (`docker compose down` puis `up -d`) et constater que la configuration est conservée (volume `ha/config`).

**Proxy** : le `configuration.yaml` inclut `trusted_proxies` (`172.19.0.0/16`) et `use_x_forwarded_for` pour autoriser Traefik.

## Site3 - AutoPassion 🏎️

Site web statique sur l'automobile déployé avec Nginx :
- **Technologie** : HTML5 + CSS3 dans un conteneur Nginx Alpine
- **Contenu** : Présentation de voitures de sport, électriques, SUV et classiques
- **Design** : Interface responsive avec dégradé bleu, cards interactives et statistiques
- **Stockage** : Aucune donnée persistée (site statique uniquement)
- **Accessible sur** : http://site3.localhost

## Site4 - Cosmos 🪐

Site web statique sur les planètes déployé avec Nginx :
- **Technologie** : HTML5 + CSS3 dans un conteneur Nginx Alpine
- **Contenu** : Présentation des planètes + faits clés du système solaire
- **Design** : Thème spatial sombre, étoiles, badges et cartes planètes
- **Stockage** : Aucune donnée persistée (site statique uniquement)
- **Accessible sur** : http://site4.localhost

## Stack Média - Architecture et fonctionnement 🎬

### Vue d'ensemble

La stack média fonctionne comme un écosystème interconnecté :

1. **Prowlarr** : Point central pour configurer les indexeurs (sources de torrents)
2. **Radarr/Sonarr/Lidarr/Readarr** : Gèrent automatiquement la recherche et l'organisation du contenu
3. **qBittorrent** : Télécharge les fichiers demandés par les *arr
4. **Bazarr** : Récupère automatiquement les sous-titres pour les films et séries
5. **Jellyfin** : Lit et diffuse tous les médias téléchargés

### Réseaux

- **Réseau traefik** : Pour l'accès web via Traefik
- **Réseau media** : Réseau privé interne pour la communication entre services

### Volumes et stockage

```
media/
 ├── downloads/          # Dossier de téléchargement (qBittorrent)
 ├── movies/             # Films organisés (Radarr → Jellyfin)
 ├── shows/              # Séries organisées (Sonarr → Jellyfin)
 ├── music/              # Musique organisée (Lidarr → Jellyfin)
 ├── books/              # Livres (Readarr)
 └── configs/            # Configuration persistante de chaque service
      ├── jellyfin/
      ├── radarr/
      ├── sonarr/
      ├── lidarr/
      ├── readarr/
      ├── bazarr/
      ├── prowlarr/
      └── qbittorrent/
```

### Données persistées

Tous les services ont leur configuration sauvegardée dans `media/configs/`. Les médias téléchargés restent dans `media/movies`, `media/shows`, etc.

### Configuration initiale recommandée

1. **Prowlarr** : Configurer les indexeurs en premier
2. **qBittorrent** : Récupérer le mot de passe temporaire depuis les logs
3. **Radarr/Sonarr/Lidarr/Readarr** : Lier à Prowlarr et qBittorrent
4. **Bazarr** : Lier à Radarr et Sonarr
5. **Jellyfin** : Scanner les bibliothèques de médias

## Structure actuelle
```
projet_docker/
 ├── docker-compose.yml
 ├── README.md
 ├── traefik/
 │   └── dynamic.yml       # Configuration routing Traefik
 ├── ha/
 │   └── config/           # Volume pour Home Assistant
 ├── site1/
 │   ├── Dockerfile
 │   ├── next.config.js
 │   ├── package.json
 │   ├── pages/
 │   │   └── index.js
 │   └── .dockerignore
 ├── site3/
 │   ├── Dockerfile
 │   ├── index.html
 │   └── style.css
 ├── site4/
 │   ├── Dockerfile
 │   ├── index.html
 │   └── style.css
 └── media/                # BONUS: Stack média
     ├── downloads/
     ├── movies/
     ├── shows/
     ├── music/
     ├── books/
     └── configs/
         ├── jellyfin/
         ├── radarr/
         ├── sonarr/
         ├── lidarr/
         ├── readarr/
         ├── bazarr/
         ├── prowlarr/
         └── qbittorrent/
```

## Notes
- Aucun port n'est exposé directement pour les services applicatifs (seul Traefik expose le port 80)
- Tous les services sont accessibles uniquement via Traefik avec leur domaine respectif
- Les 3 sites de base sont déployés et fonctionnels ! 🚀
- **BONUS** : 8 services média additionnels pour un total de 11 services conteneurisés ! 🎬

## Commandes utiles

```sh
# Démarrer tous les services
docker compose up -d

# Voir les logs d'un service spécifique
docker compose logs -f jellyfin

# Arrêter tous les services
docker compose down

# Redémarrer un service
docker compose restart radarr

# Voir l'état des conteneurs
docker compose ps
```
