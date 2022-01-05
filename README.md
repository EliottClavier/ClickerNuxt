# DeployNuxtB3
Utilisation de Firebase Hosting et GitHub Actions sur un projet NuxtJS par défault

Commandes utilisées:

```console
firebase init
```
... pour initialiser le hosting de firebase

```console
firebase deploy
```
... pour tester le déploiement

```console
npm run ci && npm run build
```
... qui build l'application

```console
npm run test
```
... qui redirige vers le testing jest

Avec Ubuntu du Microsoft Store
```console
base64 .env > env.b64
```
... pour générer le .env en base64 et l'enregistrer dans les secrets de GitHub (clé utilisée dans le worflow de déploiement)
