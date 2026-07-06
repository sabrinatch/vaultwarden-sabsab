# Vaultwarden DevSecOps — Brique 1 : Environnement local

Déploiement local de Vaultwarden derrière un reverse proxy Caddy (TLS),
comme socle du projet. Testé sur macOS Apple Silicon (M2) avec Docker Desktop.

## Prérequis

- Docker Desktop installé et lancé (Apple Silicon : les images utilisées
  (`vaultwarden/server`, `caddy:2-alpine`) fournissent nativement des
  variantes `arm64`, pas besoin d'émulation Rosetta).
- Docker Compose v2 (inclus dans Docker Desktop récent).

## Démarrage

1. Copier le fichier d'environnement modèle :
   ```bash
   cp .env.example .env
   ```

2. Générer le hash du token admin (ne jamais mettre un mot de passe en clair
   dans `ADMIN_TOKEN`) :
   ```bash
   docker run --rm -it vaultwarden/server /vaultwarden hash
   ```
   Copier le hash affiché (commence par `$argon2id$...`) dans `.env`,
   sur la ligne `ADMIN_TOKEN=`.

3. Lancer la stack :
   ```bash
   docker compose up -d
   ```

4. Vérifier que tout tourne :
   ```bash
   docker compose ps
   docker compose logs -f
   ```

5. Ouvrir https://localhost dans le navigateur.
   Le certificat est auto-signé (autorité locale générée par Caddy) : le
   navigateur affichera un avertissement, c'est normal en local. Tu peux
   l'accepter, ou faire confiance à la CA de Caddy pour l'enlever :
   ```bash
   docker compose exec caddy caddy trust
   ```

6. Interface d'administration Vaultwarden : https://localhost/admin
   (authentification avec le mot de passe en clair correspondant au hash
   généré à l'étape 2, jamais le hash lui-même).

## Arrêt

```bash
docker compose down
```
Les données persistent dans le volume Docker `vw-data` (pas supprimées par
`down`, seulement par `down -v`).

## Ce qu'on a mis en place ici

- **Aucun secret en dur dans le code** : `.env` est ignoré par Git, seul
  `.env.example` (sans valeur réelle) est versionné.
- **Surface d'attaque réduite** : seul Caddy expose des ports sur l'hôte ;
  Vaultwarden n'est joignable que via le réseau Docker interne.
- **Chiffrement en transit** : TLS géré automatiquement par Caddy.
- **Pas d'inscription libre** (`SIGNUPS_ALLOWED=false`) : seule la personne
  administrant l'instance peut créer des comptes.
- **Token admin sous forme de hash Argon2**, jamais en clair.

## Prochaine étape (brique 2)

Audit initial des secrets : scanner ce repo avec Gitleaks pour vérifier
qu'aucun secret n'est réellement exposé, et documenter la méthodologie
d'audit.
