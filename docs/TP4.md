# TP4 : GitOps avec Argo CD

**Durée : environ 3 h** (+ QCM) · Jour 4, après-midi

## Objectif

Déployer l'application avec le GitOps : on ne touche plus jamais au cluster à la main, on modifie un dépôt Git
et Argo CD applique le changement. À la fin, un simple `git tag` déploie une nouvelle version dans le cluster.

> 📸 Ce TP contient 10 captures pour le dossier (TP4-1 à TP4-10).

## Avant de commencer

- [x] Docker Desktop lancé, avec **au moins 4 Go de mémoire** (Settings → Resources)
- [x] Votre image `ghcr.io/<compte>/tasks-api:0.1.0` existe et son package est **public**
- [x] `kubectl version --client` et `kind version` répondent

### Les commandes qui changent selon votre système

| Action | macOS / Linux | Windows (PowerShell) |
|---|---|---|
| Mot de passe Argo CD | `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" \| base64 -d` | `[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String((kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}")))` |
| Appeler l'API | `curl http://localhost:8081/version` | `curl.exe http://localhost:8081/version` |

---

## Partie 1 : Cluster et Argo CD (35 min)

```bash
kind create cluster --name gitops        # inutile si déjà fait hier
kubectl get nodes
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s
```

Ouvrez l'interface : dans un terminal **qui reste ouvert**, lancez `kubectl port-forward svc/argocd-server -n argocd 8080:443`,
puis allez sur https://localhost:8080 (acceptez l'avertissement du certificat). Utilisateur : `admin`, mot de passe : voir le tableau ci-dessus.

> 📸 **TP4-1 :** `kubectl get nodes` (un nœud `Ready`).
> 📸 **TP4-2 :** `kubectl get pods -n argocd` (tous `Running`) et la page de connexion d'Argo CD.

**Questions :** C'est quoi un pod ? Un deployment ? Pourquoi un `namespace` ?

> **Réponses :**
> - **Un Pod** est la plus petite unité déployable dans Kubernetes : il encapsule un ou plusieurs conteneurs (ici notre conteneur `tasks-api`) qui partagent la même adresse IP et le même espace réseau/stockage.
> - **Un Deployment** est un contrôleur Kubernetes qui gère le cycle de vie des pods : il s'assure que le nombre souhaité d'instances (`replicas`) tourne en permanence, recrée automatiquement un pod s'il plante, et gère les mises à jour sans coupure (*rolling updates*).
> - **Un `namespace`** permet de découper et d'isoler logiquement les ressources au sein d'un même cluster (par exemple isoler les composants système d'Argo CD dans le namespace `argocd` et ceux de notre application métier dans le namespace `tasks`).

---

## Partie 2 : Le dépôt de configuration (30 min)

1. Créez un **nouveau dépôt GitHub public** nommé `tasks-config`.
2. Copiez-y le contenu du dossier `config-repo/` de votre projet (le dossier `k8s/` et le `README.md`).
3. Dans `k8s/deployment.yaml`, remplacez `<votre-compte-github>` par votre identifiant (minuscules).
4. Commitez et poussez sur `main`.
5. Dans votre projet applicatif, ouvrez `argocd/application.yaml`, remplacez aussi `<votre-compte-github>`, puis :

```bash
kubectl apply -f argocd/application.yaml
kubectl get pods -n tasks -w     # Ctrl + C pour quitter
```

6. Observez l'application dans l'interface Argo CD : elle doit devenir **Synced** et **Healthy**.
7. Accédez à l'API : `kubectl port-forward svc/tasks-api -n tasks 8081:80` (terminal ouvert), puis appelez `/version`.

> 📸 **TP4-3 :** l'application dans Argo CD, état Synced et Healthy, avec ses 2 pods.
> 📸 **TP4-4 :** la réponse de `/version` obtenue dans le cluster.

**Question :** Pourquoi sépare-t-on le dépôt de l'application et le dépôt de configuration ?

> **Réponse :**
> - **Découplage CI / CD :** modifier la configuration Kubernetes (par exemple changer le nombre de `replicas`) ne relance pas inutilement les tests Python et la construction de l'image Docker du dépôt applicatif.
> - **Sécurité et séparation des accès :** les développeurs peuvent travailler sur `tasks-api` tandis que seuls le pipeline de release et l'équipe Ops/Infra (ou Argo CD en lecture) accèdent à `tasks-config`.
> - **Historique d'audit propre :** l'historique Git de `tasks-config` ne contient que les changements d'état du cluster (versions déployées, réplicas), ce qui rend la lecture et les retours arrière (`git revert`) immédiats.

---

## Partie 3 : Le GitOps en action (25 min)

**A. Changer par Git.** Dans `tasks-config`, passez `replicas` de 2 à 3, commitez, poussez. Cliquez sur **Refresh** dans Argo CD
(ou attendez environ 3 minutes) : un 3ᵉ pod apparaît.

**B. Dérive.** Faites une modification **à la main** : `kubectl scale deployment tasks-api -n tasks --replicas=5`.
Observez : Argo CD détecte l'écart et **remet 3** (c'est `selfHeal`).

**C. Retour arrière.** Dans `tasks-config`, faites `git revert HEAD` (annule le dernier commit), poussez : le nombre de pods revient à 2.

> 📸 **TP4-5 :** le commit sur `replicas: 3` et les 3 pods dans Argo CD.
> 📸 **TP4-6 :** les pods juste après le `kubectl scale` à 5, puis après la correction automatique (deux captures).
> 📸 **TP4-7 :** l'historique des commits de `tasks-config` avec le `revert`, et le retour à 2 pods.

**Questions :** Que signifie `selfHeal` ? Pourquoi le retour arrière est-il simple en GitOps ?

> **Réponses :**
> - **`selfHeal` (auto-réparation) :** lorsque `selfHeal: true` est activé, Argo CD compare en continu l'état réel du cluster Kubernetes avec l'état souhaité dans Git. Si quelqu'un modifie le cluster à la main (ex. `kubectl scale --replicas=5`), Argo CD détecte immédiatement cette dérive (*drift*) et réapplique l'état défini dans Git (`replicas: 3`).
> - **Simplicité du retour arrière (*rollback*) :** en GitOps, le dépôt Git est l'unique source de vérité (*Single Source of Truth*). Pour annuler un déploiement ou un changement de configuration, on n'a pas besoin de chercher quelles commandes `kubectl` lancer : un simple `git revert HEAD` suivi d'un `git push` dans `tasks-config` suffit pour qu'Argo CD rétablisse l'état précédent.

---

## Partie 4 : La chaîne complète (60 min)

Objectif : un tag `v0.2.0` sur le dépôt applicatif doit aboutir à la nouvelle version dans le cluster, sans aucune commande manuelle.

1. **Jeton d'accès.** GitHub → Settings → Developer settings → **Fine-grained personal access tokens** → *Generate new token* :
   accès limité au **seul dépôt `tasks-config`**, permission **Contents : Read and write**. Copiez le jeton.
2. Dans le dépôt **applicatif** : Settings → Secrets and variables → Actions → nouveau secret `CONFIG_REPO_TOKEN` (collez le jeton).
3. Le dossier `scripts/` de votre projet contient déjà `set-image-tag.sh` : il remplace la version dans le manifeste, vous n'avez rien à écrire.
4. Ajoutez à `docker.yml` un job `update-config` (après `deploy-production`, uniquement sur tag) qui :
   - récupère le dépôt applicatif **et** le dépôt `tasks-config` (`actions/checkout` avec `repository`, `token: ${{ secrets.CONFIG_REPO_TOKEN }}` et `path: config`) ;
   - lance `bash scripts/set-image-tag.sh config/k8s/deployment.yaml "$VERSION"` ;
   - commite et pousse dans `tasks-config`.
   (Passez la version par une variable d'environnement `VERSION`, **jamais** en écrivant `${{ … }}` directement dans la commande.)
5. Créez le tag `v0.2.0`, approuvez la production, et suivez : pipeline → commit automatique dans `tasks-config` → Argo CD → nouveaux pods.
6. Vérifiez que `/version` répond `0.2.0`.

> 📸 **TP4-8 :** le run complet avec tous les jobs au vert, `update-config` compris.
> 📸 **TP4-9 :** le commit automatique dans `tasks-config` et Argo CD synchronisé sur `0.2.0`.
> 📸 **TP4-10 :** `/version` qui répond `0.2.0`.

**Questions :** Résumez en 5 lignes le chemin d'un `git tag` jusqu'au pod. Qui pousse (push) et qui tire (pull) ? Quelle différence avec un déploiement CI/CD « classique » ?

> **Réponses :**
> - **Chemin d'un `git tag` jusqu'au pod (en 5 lignes) :**
>   1. Le développeur pousse le tag `v0.2.0` sur le dépôt applicatif `tasks-api`, ce qui déclenche le workflow `docker.yml`.
>   2. Les jobs `test` et `build` valident le code puis construisent et publient l'image `ghcr.io/maximemlly/tasks-api:0.2.0` sur GHCR.
>   3. Après le test de fumée sur `deploy-staging` et l'approbation manuelle sur `deploy-production`, le job `update-config` s'exécute.
>   4. `update-config` clone `tasks-config`, met à jour le tag `0.2.0` dans `k8s/deployment.yaml` via `set-image-tag.sh`, puis commite et pousse sur `main`.
>   5. Argo CD détecte le nouveau commit sur `tasks-config`, synchronise le `Deployment` dans le cluster Kubernetes, et Kubernetes remplace les pods avec l'image `0.2.0`.
> - **Qui pousse (push) et qui tire (pull) ?**
>   - **Push :** le développeur pousse le tag Git ; le pipeline GitHub Actions pousse l'image Docker vers GHCR et pousse le commit de mise à jour vers `tasks-config`.
>   - **Pull :** Argo CD (à l'intérieur du cluster) tire (*pull*) les manifestes depuis `tasks-config`, et le nœud Kubernetes tire (*pull*) l'image Docker depuis GHCR.
> - **Différence avec un déploiement CI/CD « classique » :**
>   - Dans un pipeline CI/CD « classique » (*push-based*), c'est le runner GitHub Actions qui exécute `kubectl apply` vers le cluster : il faut donc ouvrir l'API du cluster sur Internet et stocker des identifiants administrateur Kubernetes dans les secrets GitHub. En GitOps (*pull-based*), le cluster ne donne aucun accès entrant à la CI : c'est Argo CD, installé dans le cluster, qui vient lire Git et appliquer les changements.

---

## QCM final (30 min)

Le formateur vous donne un lien : QCM individuel, sans documents. C'est la moitié de la note.

## Dossier à rendre

Complétez votre dossier (TP1 à TP4 + bilan) : à rendre le vendredi 6 novembre à 17 h. Voir `docs/DOSSIER.md`.

## Nettoyage (facultatif)

`kind delete cluster --name gitops`
