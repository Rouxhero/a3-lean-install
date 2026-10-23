# A3 Lean

**Éditeur web de rapports A3 (méthode Lean) et de rapports techniques**, à installer chez soi avec Docker.
Une affiche A3 pour la synthèse, un rapport texte de plusieurs pages pour le détail, reliés par un lien et un QR code.
Aide à la rédaction par une IA locale, sans envoyer vos textes ailleurs.

*Web editor for Lean A3 reports and linked technical reports, self-hosted with Docker. The interface is in French.*

![Une affiche A3 dans l'éditeur](images/editeur-affiche.png)

## Ce que fait A3 Lean

- **Affiches A3** : une feuille A3 paysage en sections, remplie de blocs — texte, listes, tableaux, schémas Mermaid, images, indicateurs, étapes, et blocs Lean (Ishikawa, 5 pourquoi, Gantt, plan d'actions). Aperçu en direct, mise en page à la souris, flèches entre sections, palettes et styles.
- **Rapports texte** : un document A4 en Markdown, avec page de garde, sommaire et titres numérotés, relié à son affiche. L'affiche peut imprimer un **QR code** vers le rapport.
- **Exports** : PDF (A3 ou A4 avec sommaire paginé), image PNG, PowerPoint, Word, et un PDF « dossier » affiche + rapport.
- **Partage** : lien de lecture, bibliothèque publique protégée par un mot de passe (avec journal des accès).
- **IA locale** (Ollama, facultative) : reformuler un texte, transformer des notes en affiche, traduire, relecture critique façon coach Lean.
- **Travail à plusieurs** : historique des versions, présence, conflits d'enregistrement sans perte, notes de relecture.
- **Vos données restent des fichiers** : une affiche = un fichier `.yaml`, un rapport = un fichier `.md`, dans un dossier que vous sauvegardez comme vous voulez. Pas de base de données, fonctionne hors ligne.

| Rapport texte (A4) | Portail des projets |
|---|---|
| ![Un rapport texte](images/editeur-rapport.png) | ![Le portail](images/portail.png) |

## Installer

Il faut [Docker](https://docs.docker.com/get-docker/) avec Compose (PC Linux, Windows ou Mac ; processeur Intel/AMD ou ARM, dont Raspberry Pi 4/5 en 64 bits et Mac M*). Prévoir ~4 Go de RAM libre avec l'IA, ~1 Go sans.

```bash
mkdir a3-lean && cd a3-lean
curl -fsSLO https://raw.githubusercontent.com/Rouxhero/a3-lean-install/main/docker-compose.yml
docker compose up -d
```

Puis ouvrir **http://localhost:8000**. Deux exemples reliés vous attendent : l'affiche *Prise en main d'A3 Lean* et son *Guide technique*.

Au premier lancement, le modèle d'IA (`qwen2.5:3b`, ~2 Go) se télécharge en arrière-plan : l'éditeur marche tout de suite, l'aide à la rédaction dès la fin du téléchargement (`docker compose logs -f ollama-pull`).

## Régler

Copier [`.env.example`](.env.example) en `.env` à côté de `docker-compose.yml`, décommenter ce qu'il faut changer, puis `docker compose up -d` :

| Réglage | Exemple |
|---|---|
| Port | `A3_PORT=8080` |
| Nom de l'auteur (versions, notes) | `A3_AUTEUR=Camille` |
| Modèle Ollama plus gros (meilleurs textes, plus de RAM) | `A3_LLM_MODEL=qwen2.5:7b` |
| Autre serveur compatible OpenAI (OpenAI, LM Studio…) | `A3_LLM_URL`, `A3_LLM_MODEL`, `A3_LLM_API_KEY` |
| Version fixe | `A3_VERSION=1.1.0` |

**Sans IA** : supprimer les services `ollama` et `ollama-pull` (et la ligne `depends_on`) dans `docker-compose.yml`. Le reste de l'application fonctionne normalement.

## Mettre à jour, sauvegarder

```bash
docker compose pull && docker compose up -d     # dernière version
```

Tout est dans le dossier `data/` (rapports, images, versions) : c'est lui qu'il faut sauvegarder. La version installée s'affiche dans ⚙ *Réglages*. Les versions publiées : [ghcr.io/rouxhero/a3-lean](https://github.com/users/Rouxhero/packages/container/package/a3-lean).

## ⚠ Sécurité : pas de connexion intégrée

A3 Lean n'a pas de comptes ni de mot de passe : toute personne qui atteint le port peut lire et modifier les rapports. C'est pourquoi le `docker-compose.yml` ne l'expose **que sur la machine elle-même** (`127.0.0.1`).

Pour l'utiliser à plusieurs ou à distance, placez-la derrière un **reverse proxy qui demande une connexion** : Authentik, Authelia, Caddy ou Nginx avec authentification, Tailscale… Si le proxy transmet le nom de la personne dans les en-têtes `X-authentik-username` et `X-authentik-name`, chaque version et chaque note est signée de son nom.

Seule la bibliothèque publique (préfixe `/p/`) est faite pour être ouverte sans compte : elle a son propre mot de passe, ne permet que la lecture des rapports que vous y mettez, et note chaque visite.

## Licence

[MIT](LICENSE). Ce dépôt ne contient que de quoi installer l'application : l'image Docker est construite et publiée automatiquement à chaque version.
