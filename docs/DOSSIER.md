# Dossier de rendu : GitOps & CI/CD

## L'évaluation en bref

| Élément                                                             | Poids                                                            |
| ------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Dossier de TP (ce document)                                         | **10 points sur 20** (50 %) : dossier complet et correct = 10/10 |
| QCM (20 questions, 30 minutes, jeudi 5 novembre, en fin de journée) | **10 points sur 20** (50 %)                                      |

Le dossier rassemble **tous les TP de la semaine**, avec des captures d'écran et vos explications.
Vous le construisez **au fil des jours** : ne gardez pas tout pour le jeudi soir !

- **À rendre :** le **vendredi 6 novembre à 17 h**, dernier délai · envoi du lien de votre dépôt GitHub dans le tableur Excel fourni par le formateur (voir ci-dessous)
- **Qui rend quoi :** le dossier est **individuel** : vous pouvez travailler à deux pendant les TP, mais chacun rend son propre dossier, avec ses propres captures et ses propres mots.
- **Format :** un seul fichier **PDF**, nommé `NOM_Prenom_GitOps-CICD.pdf`.
  Rédigez-le dans l'outil de votre choix (Word, Google Docs, LibreOffice…) puis exportez en PDF.
- **Lien :** indiquez en première page l'adresse de votre dépôt GitHub (il doit être **public**).
- **Dépôt et tableur :** publiez votre projet `tasks-api` sur GitHub (dépôt **public**, avec le code, les workflows et vos modifications), puis remplissez le **tableur fourni par le formateur** : nom, prénom, adresse e-mail, lien du dépôt. Un dépôt absent, privé ou incorrect fait perdre des points au dossier.

## Structure du dossier

1. **Page de garde** : nom(s), prénom(s), date, lien vers votre dépôt GitHub.
2. **TP1** : première CI avec GitHub Actions.
3. **TP2** : une CI solide.
4. **TP3** : Docker et livraison continue.
5. **TP4** : GitOps.
6. **Bilan** : 10 lignes maximum : ce que vous avez retenu, ce qui vous a posé problème, ce que vous referiez autrement.

Pour **chaque TP**, trois choses :

- **Objectif** : 2 ou 3 phrases avec vos mots (pas une copie de l'énoncé).
- **Captures d'écran** numérotées, chacune avec une **légende** d'une phrase qui explique ce que l'on voit.
- **Réponses** aux questions posées dans l'énoncé du TP, et **difficultés rencontrées** (ce qui a bloqué, comment vous avez trouvé la solution).

## TP1 : captures et questions

**Objectif :** Mettre en place un premier pipeline d'intégration continue (CI) avec GitHub Actions afin d'exécuter automatiquement les tests unitaires (`pytest`) à chaque `git push`. Apprendre à lire les journaux d'exécution (logs) d'un workflow, identifier rapidement une régression lorsque la CI passe au rouge, et sécuriser la fusion d'une Pull Request grâce aux vérifications automatiques.

| N°    | Capture à fournir                                             |
| ----- | ------------------------------------------------------------- |
| TP1-1 | Le run « Hello » réussi, étape « branche / auteur » dépliée   |
| TP1-2 | Le run `ci.yml` réussi, avec la liste des étapes              |
| TP1-3 | Le run rouge, étape `pytest` dépliée avec le message d'erreur |
| TP1-4 | Le run vert après la correction                               |
| TP1-5 | La pull request avec les vérifications vertes, avant fusion   |
| TP1-6 | La pull request fusionnée                                     |
| TP1-7 | Le `README.md` sur GitHub avec le badge affiché               |

Questions à traiter :

1. Combien de temps a duré le run « Hello » ? Où voit-on les logs de chaque étape ?
   <br><i>Le run a duré 5 secondes. On voit les logs de chaque étape en cliquant sur le run puis sur chaque étape.</i>

2. Quel test a échoué dans la partie 3, et à quelle ligne ? Qu'est-ce qui a permis de le trouver rapidement ?
   <br><i>Le test qui a échoué dans la partie 3 est test_title_too_long à la ligne 12. On a trouvé le problème grâce aux logs du run.</i>

3. Pourquoi attend-on que la CI soit verte avant de fusionner une pull request ?
   <br><i>On attend que la CI soit verte avant de fusionner une pull request car elle permet de s'assurer que le code est de bonne qualité et qu'il respecte les conventions établies. De plus, cela évite d'introduire des régressions dans le code.</i>

**Difficultés rencontrées :**
<br><i>Comprendre que le runner GitHub Actions démarre sur une machine virtuelle vierge à chaque exécution et qu'il faut donc systématiquement récupérer le code (`actions/checkout`), installer Python et installer les dépendances avant de lancer `pytest`.</i>

## TP2 : captures et questions

**Objectif :** Renforcer le pipeline de CI en séparant la vérification de la qualité du code (`flake8`, `black`) et l'exécution des tests avec exigence de couverture (`pytest --cov`). Tester l'application sur plusieurs versions de Python en parallèle via une `matrix`, accélérer les runs grâce au cache `pip`, protéger la branche `main` et manipuler les secrets et conflits Git en équipe.

| N°     | Capture à fournir                                                                                                 |
| ------ | ----------------------------------------------------------------------------------------------------------------- |
| TP2-1  | un run vert montrant les deux jobs `lint` et `test`.                                                              |
| TP2-2  | le job `lint` rouge, avec le message de flake8 lisible.                                                           |
| TP2-3  | la liste des 3 jobs de la matrice (`test (3.11)`, `test (3.12)`, `test (3.13)`).                                  |
| TP2-4  | l'étape setup-python du 2ᵉ run, avec la mention du cache restauré, et la section « Artifacts » de la page du run. |
| TP2-5  | la page de la règle de protection avec les vérifications obligatoires.                                            |
| TP2-6  | une PR dont la fusion est bloquée par une vérification en échec.                                                  |
| TP2-7  | la PR de B avec le message de conflit.                                                                            |
| TP2-8  | le fichier `CHANGELOG.md` final sur `main` contenant les deux lignes.                                             |
| TP2-9  | les logs où la valeur du secret apparaît masquée par `***`.                                                       |
| TP2-10 | le run en échec de chaque pipeline (une capture par pipeline, nommées TP2-10A, TP2-10B, TP2-10C).                 |
| TP2-11 | la liste des runs montrant vos trois pipelines réparés au vert.                                                   |

Questions à traiter :

1. Quelle différence entre un linter (`flake8`) et un formateur (`black`) ? Pourquoi séparer `lint` et `test` en deux jobs ?
   <br><i>Un linter (`flake8`) analyse le code pour détecter des erreurs de syntaxe, des variables ou imports inutilisés et des problèmes de qualité, tandis qu'un formateur (`black`) impose un style de mise en forme uniforme (indentation, guillemets, longueur des lignes). Séparer `lint` et `test` en deux jobs permet de les exécuter en parallèle et de voir immédiatement au premier coup d'œil si un échec provient du style/qualité du code ou d'un test cassé.</i>

2. Pourquoi tester sur plusieurs versions de Python ? Que gagne-t-on avec le cache ? Un artefact, c'est quoi ?
   <br><i>Tester sur plusieurs versions de Python (3.11, 3.12, 3.13) garantit la compatibilité de l'application sur différents environnements et anticipe les dépréciations. Le cache `pip` évite de retélécharger les paquets à chaque run, ce qui réduit le temps d'exécution du pipeline. Un artefact est un fichier généré pendant l'exécution d'un workflow (ici le rapport `coverage.xml`) que GitHub conserve après la fin du run pour qu'on puisse le télécharger ou le réutiliser dans un autre job.</i>

3. Pourquoi interdire de pousser directement sur `main` ?
   <br><i>Interdire le push direct sur `main` oblige chaque modification à passer par une Pull Request et par les vérifications automatiques de la CI (`lint` et `test`). Cela garantit que la branche `main` reste toujours stable et déployable.</i>

4. Pourquoi Git ne peut-il pas résoudre ce conflit seul ? Qu'est-ce qui a vérifié le code après la résolution ?
   <br><i>Git ne peut pas résoudre ce conflit seul car les deux branches ont modifié exactement la même ligne du fichier `CHANGELOG.md` de manière différente : seul un humain peut décider s'il faut garder l'une, l'autre ou les deux modifications. Après la résolution et le push, c'est la CI GitHub Actions qui s'est relancée automatiquement sur la Pull Request pour vérifier le résultat.</i>

5. Pourquoi GitHub masque-t-il le secret ? Que se passe-t-il si on le commite dans le dépôt ? Secret ou variable : comment choisir ?
   <br><i>GitHub masque les secrets par `***` dans les logs pour empêcher toute fuite accidentelle d'informations sensibles (jetons d'API, mots de passe), surtout sur un dépôt public où les logs sont lisibles par tous. Si on commite un secret dans le dépôt, il reste inscrit dans l'historique Git et est compromis : il faut immédiatement le révoquer et en régénérer un nouveau. On utilise un **secret** pour toute donnée sensible (token, clé privée, mot de passe) et une **variable** (`vars`) pour une configuration non sensible (nom d'environnement `staging`, URL publique, options).</i>

6. Pour chaque pipeline (A, B, C) : quel était le bug, comment l'avez-vous trouvé, comment l'éviter à l'avenir ?
   <br><i>- **Pipeline A :** Bug d'indentation YAML à la ligne 12 (`- run: pip install ...` décalé d'un espace en trop), qui empêchait le workflow de se lancer (erreur de syntaxe YAML indiquée dès l'onglet Actions). Pour l'éviter : utiliser un éditeur avec validation YAML ou `actionlint`.
   <br>- **Pipeline B :** Oubli de l'étape `uses: actions/checkout@v6` au début du job, trouvé dans les logs de `pip install` (`ERROR: Could not open requirements file: [Errno 2] No such file or directory: 'requirements-dev.txt'`). Pour l'éviter : toujours commencer un job qui manipule les fichiers du dépôt par `actions/checkout`.
   <br>- **Pipeline C :** Les versions dans la matrice (`[3.10, 3.11, 3.12]`) n'étaient pas entourées de guillemets : YAML a interprété `3.10` comme le nombre flottant `3.1`, faisant échouer `setup-python` qui cherchait Python 3.1. Pour l'éviter : toujours écrire les numéros de version entre guillemets (`["3.10", "3.11", "3.12"]`).</i>

**Difficultés rencontrées :**
<br><i>Lors de la mise en place de la matrice Python, il fallait penser à mettre les versions entre guillemets et à conditionner l'upload de l'artefact (`if: matrix.python-version == '3.12'`) pour éviter que les 3 jobs n'écrasent le même nom d'artefact.</i>

## TP3 : captures et questions

**Objectif :** Conteneuriser l'application Flask à l'aide d'un `Dockerfile` sécurisé (utilisateur non-root) et d'un fichier `docker-compose.yml`. Automatiser la construction et la publication de l'image sur GitHub Container Registry (GHCR) via GitHub Actions, gérer le versionnement par tags Git (`v0.1.0`), et mettre en place un pipeline de livraison continue avec tests de fumée et approbation manuelle avant la mise en production.

| N°    | Capture à fournir                                                                            |
| ----- | -------------------------------------------------------------------------------------------- |
| TP3-1 | le `docker build` réussi, puis `curl` sur `/health` qui répond `{"status":"ok"}`.            |
| TP3-2 | `docker compose up` en cours et le `curl` sur `/version`.                                    |
| TP3-3 | le run avec `test` puis `build` au vert.                                                     |
| TP3-4 | la page du package `tasks-api` avec son tag `sha-…`.                                         |
| TP3-5 | la liste des tags du package (`0.1.0` et `sha-…`).                                           |
| TP3-6 | `docker run` de l'image **téléchargée depuis GHCR** et `curl /version` qui répond `0.1.0`.   |
| TP3-7 | le run en attente d'approbation (« Waiting » / « Review deployments »), puis le run terminé. |
| TP3-8 | la sortie de la commande `id` dans le conteneur.                                             |

Questions à traiter :

1. Différence entre une image et un conteneur ? À quoi sert le `.dockerignore` ? Pourquoi `requirements.txt` est-il copié avant le code ?
   <br><i>Une **image** est un modèle immuable (en lecture seule) contenant le système, les dépendances et le code de l'application, tandis qu'un **conteneur** est une instance en cours d'exécution de cette image. Le fichier `.dockerignore` permet d'exclure les fichiers inutiles ou lourds (`.venv`, `.git`, `.pytest_cache`) du contexte envoyé au démon Docker, ce qui accélère le build et allège l'image. On copie `requirements.txt` et on exécute `pip install` **avant** de copier `app/` pour profiter du cache de couches (layers) de Docker : tant que les dépendances ne changent pas, modifier le code source ne réexécute pas `pip install`.</i>

2. Que gagne-t-on à décrire le lancement dans un fichier (`docker-compose.yml`) plutôt que dans une longue commande ?
   <br><i>Le fichier `docker-compose.yml` documente et versionne toute la configuration d'exécution (ports, variables d'environnement, options de build) dans Git. Toute l'équipe peut lancer l'application de manière reproductible avec une simple commande `docker compose up --build` sans risque d'oublier un paramètre.</i>

3. Pourquoi `build` attend-il `test` (`needs: test`) ? Pourquoi éviter le tag `latest` en production ?
   <br><i>`build` attend `test` pour ne jamais construire ni publier sur le registre une image contenant du code qui ne passe pas les tests unitaires. En production, on évite le tag `latest` car il est mouvant et ambigu : on ne sait pas quelle version exacte tourne réellement, et il devient difficile de revenir précisément à la version précédente (rollback) en cas de bug.</i>

4. Quel est l'avantage d'une image versionnée par rapport à « le code de ma machine » ?
   <br><i>Une image versionnée (ex. `ghcr.io/maximemlly/tasks-api:0.1.0`) embarque exactement le code et ses dépendances tels qu'ils ont été testés par la CI. Elle garantit qu'elle s'exécutera de façon identique sur n'importe quel serveur ou cluster, éliminant le problème du « ça marche sur ma machine ».</i>

5. À quoi sert un test de fumée (smoke test) ? Pourquoi demander une approbation humaine avant la production ?
   <br><i>Un test de fumée vérifie rapidement que le conteneur démarre réellement et répond correctement sur ses routes essentielles (`/health` et `/version`) avant d'aller plus loin. L'approbation humaine (`Required reviewers` sur l'environnement `production`) ajoute un contrôle métier et organisationnel avant d'impacter les utilisateurs finaux.</i>

6. Pourquoi ne pas exécuter une application en root dans un conteneur ?
   <br><i>Si un attaquant exploite une faille dans l'application alors qu'elle tourne en `root` (uid 0), il dispose des privilèges administrateur dans le conteneur et, en cas d'évasion de conteneur, d'un accès privilégié sur la machine hôte. Utiliser un utilisateur dédié non-privilégié (`appuser`, uid 1000) applique le principe du moindre privilège.</i>

**Difficultés rencontrées :**
<br><i>Le registre GHCR impose que le nom du propriétaire de l'image soit entièrement en minuscules (`${GITHUB_REPOSITORY_OWNER,,}`) et il fallait bien passer `APP_VERSION` en `build-args` dans `docker/build-push-action` pour que la route `/version` renvoie le numéro du tag Git.</i>

## TP4 : captures et questions

**Objectif :** Mettre en œuvre le déploiement continu selon l'approche GitOps avec Kubernetes (`kind`) et Argo CD. Séparer le dépôt applicatif (`tasks-api`) et le dépôt d'infrastructure/configuration (`tasks-config`), observer la synchronisation automatique et l'auto-réparation (`selfHeal`) d'Argo CD, et automatiser toute la chaîne de bout en bout pour qu'un simple tag Git `v0.2.0` déploie la nouvelle version dans le cluster.

| N°     | Capture à fournir                                                                                  |
| ------ | -------------------------------------------------------------------------------------------------- |
| TP4-1  | `kubectl get nodes` (un nœud `Ready`).                                                             |
| TP4-2  | `kubectl get pods -n argocd` (tous `Running`) et la page de connexion d'Argo CD.                   |
| TP4-3  | l'application dans Argo CD, état Synced et Healthy, avec ses 2 pods.                               |
| TP4-4  | la réponse de `/version` obtenue dans le cluster.                                                  |
| TP4-5  | le commit sur `replicas: 3` et les 3 pods dans Argo CD.                                            |
| TP4-6  | les pods juste après le `kubectl scale` à 5, puis après la correction automatique (deux captures). |
| TP4-7  | l'historique des commits de `tasks-config` avec le `revert`, et le retour à 2 pods.                |
| TP4-8  | le run complet avec tous les jobs au vert, `update-config` compris.                                |
| TP4-9  | le commit automatique dans `tasks-config` et Argo CD synchronisé sur `0.2.0`.                      |
| TP4-10 | `/version` qui répond `0.2.0`.                                                                     |

Questions à traiter :

1. C'est quoi un pod ? Un deployment ? Pourquoi un `namespace` ?
   <br><i>Un **Pod** est la plus petite unité d'exécution dans Kubernetes, qui encapsule un ou plusieurs conteneurs partageant le même réseau et stockage. Un **Deployment** est un contrôleur qui gère le cycle de vie des pods : il maintient le nombre de réplicas souhaité (`replicas: 2`), les recrée en cas de panne et gère les mises à jour progressives (rolling updates). Un **Namespace** permet d'isoler logiquement les ressources au sein d'un même cluster (par exemple séparer les composants d'administration dans `argocd` et notre application métier dans `tasks`).</i>

2. Pourquoi sépare-t-on le dépôt de l'application (`tasks-api`) et le dépôt de configuration (`tasks-config`) ?
   <br><i>Séparer les deux dépôts permet de découpler le cycle de vie du code applicatif de celui de l'infrastructure Kubernetes : un commit dans la configuration ne relance pas inutilement les tests Python et le build Docker, les droits d'accès peuvent être distincts (équipes Dev vs Ops), et l'historique Git de `tasks-config` constitue un journal d'audit clair de tous les déploiements.</i>

3. Que signifie `selfHeal` ? Pourquoi le retour arrière est-il simple en GitOps ?
   <br><i>`selfHeal: true` signifie qu'Argo CD surveille en permanence l'état réel du cluster et corrige automatiquement toute dérive manuelle (comme un `kubectl scale` fait à la main) pour ramener le cluster à l'état déclaré dans Git. Le retour arrière (rollback) est très simple en GitOps car Git est l'unique source de vérité : il suffit d'un `git revert HEAD` et d'un `git push` sur le dépôt de configuration pour qu'Argo CD rétablisse immédiatement l'état précédent.</i>

4. Résumez en 5 lignes le chemin d'un `git tag` jusqu'au pod. Qui pousse (push) et qui tire (pull) ? Quelle différence avec un déploiement CI/CD « classique » ?
   <br><i>1. Le développeur pousse un tag `v0.2.0` sur le dépôt applicatif `tasks-api`.
   <br>2. GitHub Actions exécute les tests (`test`), construit et pousse l'image `ghcr.io/maximemlly/tasks-api:0.2.0` sur GHCR (`build`), puis valide `deploy-staging`.
   <br>3. Après approbation humaine sur `deploy-production`, le job `update-config` modifie `k8s/deployment.yaml` dans `tasks-config` avec la version `0.2.0` et pousse un commit.
   <br>4. Argo CD (qui tourne dans le cluster) détecte le nouveau commit dans `tasks-config` et tire (pull) les manifestes mis à jour.
   <br>5. Kubernetes télécharge la nouvelle image `0.2.0` depuis GHCR et remplace progressivement les pods.
   <br>En résumé : le pipeline CI **pousse (push)** l'image vers GHCR et le commit vers Git, tandis qu'Argo CD **tire (pull)** la configuration depuis Git vers le cluster. Contrairement à un déploiement CI/CD « classique » (push-based) où le runner GitHub Actions exécuterait `kubectl apply` et devrait posséder les identifiants administrateur du cluster Kubernetes, en GitOps le cluster n'est pas exposé à l'extérieur et se synchronise lui-même depuis Git.</i>

**Difficultés rencontrées :**
<br><i>Sur macOS, l'outil `kind` n'était pas préinstallé et Docker Desktop devait être démarré avant de pouvoir créer le cluster `gitops`. Il a aussi fallu créer un Fine-grained Personal Access Token avec la permission `Contents: Read and write` sur `tasks-config` pour permettre au job `update-config` de pousser le changement de version.</i>

## Bilan

<br><i>Au cours de ces quatre TP, j'ai mis en place une chaîne CI/CD et GitOps complète de bout en bout, allant du simple test unitaire déclenché sur un `git push` jusqu'au déploiement automatisé dans un cluster Kubernetes via Argo CD. J'ai retenu l'importance d'automatiser chaque étape de vérification (lint, couverture de tests, matrice multi-versions, tests de fumée) et de protéger la branche `main` pour garantir que seule une version validée puisse être livrée. La séparation entre le dépôt applicatif (`tasks-api`) et le dépôt d'état déclaratif (`tasks-config`) m'a permis de comprendre concrètement le fonctionnement du modèle pull-based d'Argo CD, notamment la détection de dérive (`selfHeal`) et la simplicité d'un rollback par `git revert`. Les principaux points de vigilance ont concerné les permissions entre dépôts GitHub (jeton `CONFIG_REPO_TOKEN`, visibilité publique du package GHCR) et la syntaxe stricte des workflows YAML. Si je devais refaire ce projet, j'ajouterais dès le départ une validation locale des fichiers YAML/Docker et l'automatisation de la création du dépôt de configuration.</i>

## Conseils de présentation

- Des captures **lisibles** : recadrez sur la partie utile, pas l'écran entier.
- **Aucun secret visible** : ne montrez jamais un mot de passe, un jeton (token) ou une clé. Floutez-les si besoin.
- Numérotez et légendez **toutes** les captures, dans l'ordre des TP.
- Écrivez avec vos mots : un dossier copié-collé de l'énoncé ne sera pas valorisé.
