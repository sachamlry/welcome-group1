# Welcome – Group1

Page d'information de Group1, fournisseur de partage de fichiers (`welcome.group1.umons`) : service proposé, obtention d'un compte, conditions d'utilisation, état des services, contact.

Le site est **statique** : nginx sert directement les fichiers du dossier `site/`. Aucune application ne tourne derrière (plus de Flask ni de Gunicorn) : rien à mettre à jour, surveiller ni défendre en dehors de nginx.

## Structure

| Chemin | Rôle |
|---|---|
| `site/index.html` | Page d'accueil |
| `site/404.html` | Page d'erreur |
| `site/static/css/welcome.css` | Style (couleurs en variables au début, thème clair puis sombre) |
| `site/static/img/favicon.svg` | Icône |
| `nginx/welcome.conf` | Copie de référence de la config nginx de la VM |

Seul `site/` est servi. Le reste du dépôt (`.git`, ce README, `nginx/`) n'est pas accessible depuis le navigateur.

## Modifier la page

Les passages à mettre à jour sont repérés par `=====` dans `site/index.html` :

- **État du service** (2 endroits) : `data-state="bientot"` et son texte. Valeurs : `bientot` (Bientôt disponible), `en_ligne` (En ligne), `maintenance` (Maintenance), `incident` (Incident en cours). Quand le service est en ligne, décommenter le bouton « Ouvrir le service ».
- **Message d'état** : décommenter le paragraphe `note` sous la liste d'état.
- **Contact** : remplacer « sera publiée très prochainement » par l'adresse réelle (exemple en commentaire).
- **HTTPS** : passer `http://files.group1.umons` en `https://` dans le bouton.

Contraintes imposées par la politique de sécurité (CSP) :

- pas de `<script>`, pas d'attribut `style="..."` ;
- pas de ressource externe (police, image en ligne). Une image va dans `site/static/img/` et s'appelle par `/static/img/nom.png`.

Aperçu local : ouvrir `site/index.html` dans un navigateur, ou `python -m http.server 8000 --directory site` puis http://127.0.0.1:8000.

## Déploiement sur la VM

### Première installation

```bash
sudo apt install -y git
sudo git clone https://github.com/<compte>/<depot>.git /opt/welcome
sudo cp /opt/welcome/nginx/welcome.conf /etc/nginx/sites-available/welcome
sudo ln -sfn /etc/nginx/sites-available/welcome /etc/nginx/sites-enabled/welcome
sudo nginx -t && sudo systemctl reload nginx
curl -i http://127.0.0.1/
```

### Mettre à jour une page

```bash
cd /opt/welcome
sudo git pull --ff-only
```

Les pages sont servies immédiatement, sans recharger nginx.

### Mettre à jour la config nginx

Après un changement de `nginx/welcome.conf` :

```bash
cd /opt/welcome && sudo git pull --ff-only
sudo cp nginx/welcome.conf /etc/nginx/sites-available/welcome
sudo nginx -t && sudo systemctl reload nginx
```

Si `nginx -t` échoue, l'ancienne config reste en service tant qu'on ne recharge pas nginx : corriger puis recommencer.

### Annoncer une maintenance ou un incident

Modifier l'état et le message dans `site/index.html`, commit, push, puis `git pull` sur la VM. En cas d'urgence sans accès à GitHub, modifier directement `/opt/welcome/site/index.html` sur la VM, puis reporter la modification dans le dépôt.

### Vérification

```bash
curl http://127.0.0.1/healthz                                        # {"status":"ok"}
curl -s -o /dev/null -w "%{http_code}\n" http://127.0.0.1/xyz        # 404
curl -s -o /dev/null -w "%{http_code}\n" -X POST http://127.0.0.1/   # 405
curl -s -o /dev/null -w "%{http_code}\n" http://127.0.0.1/.git/HEAD  # 404
```

## Limites connues

- L'état des services est déclaratif : il est mis à jour à la main, pas mesuré automatiquement.
- Le site est en HTTP tant que le certificat signé par la CA du cours n'est pas obtenu.
