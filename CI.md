# Configuration CI/CD - Documentation

## CI Version 1 : Pipeline de base pour projet Symfony

### 🤔 Qu'est-ce que la CI/CD ?

**CI** (Continuous Integration) = **Intégration Continue**  
**CD** (Continuous Deployment) = **Déploiement Continu**

Imaginez que vous travaillez en équipe sur un projet. Chaque développeur écrit du code et l'ajoute au projet principal. Sans CI/CD, vous devriez :
- Vérifier manuellement que le nouveau code ne casse rien
- Tester tous les composants à la main
- S'assurer que le code respecte les standards
- Déployer manuellement en production

**La CI/CD automatise tout cela !** 🚀

### 🎯 Objectifs de notre CI Version 1

Notre pipeline automatise 4 vérifications essentielles :

1. **Tests automatisés** : "Le code fonctionne-t-il correctement ?"
2. **Qualité du code** : "Le code est-il bien écrit et maintenable ?"
3. **Sécurité** : "Y a-t-il des vulnérabilités connues ?"
4. **Assets frontend** : "Les fichiers CSS/JS se compilent-ils correctement ?"

### 🔄 Comment ça fonctionne ?

#### Déclenchement automatique
```yaml
on:
  push:
    branches: [ master, develop ]
  pull_request:
    branches: [ master, develop ]
```

**En français** : "Lance la CI à chaque fois que quelqu'un pousse du code sur `master` ou `develop`, ou crée une Pull Request vers ces branches"

**Pourquoi ?** Cela garantit que chaque modification est vérifiée avant d'être intégrée au projet principal.

---

## 🧪 Job 1 : Tests Automatisés

### Qu'est-ce qu'on fait ?
- On teste notre code PHP sur **2 versions différentes** (8.2 et 8.3)
- On utilise une **vraie base de données MySQL** pour les tests
- On mesure la **couverture de code** (quel pourcentage du code est testé)

### Pourquoi c'est important ?
- **Multi-version** : S'assurer que notre code marche sur différentes versions de PHP
- **Base de données réelle** : Les tests sont plus fiables qu'avec une base de données fictive
- **Couverture** : Voir quelles parties du code ne sont pas testées

### Étapes détaillées :

#### 1. Préparation de l'environnement
```yaml
strategy:
  matrix:
    php-version: [8.2, 8.3]
```
**= "Teste sur PHP 8.2 ET PHP 8.3"**

#### 2. Base de données de test
```yaml
services:
  mysql:
    image: mysql:8.0
    env:
      MYSQL_DATABASE: cooked_test
```
**= "Crée une base MySQL propre pour les tests"**

#### 3. Installation et cache
```yaml
- name: Cache Composer packages
  uses: actions/cache@v3
```
**= "Évite de retélécharger les mêmes packages à chaque fois"**

#### 4. Configuration des tests
```bash
cp .env .env.test.local
echo "DATABASE_URL=mysql://root:root@127.0.0.1:3306/cooked_test" >> .env.test.local
```
**= "Configure l'application pour utiliser la base de test"**

#### 5. Exécution des tests
```bash
php bin/phpunit --coverage-clover coverage.xml
```
**= "Lance tous les tests et génère un rapport de couverture"**

---

## 🔍 Job 2 : Qualité du Code

### Qu'est-ce qu'on fait ?
- **PHP CS Fixer** : Vérifie que le code respecte les standards PHP
- **PHPStan** : Analyse le code pour détecter les erreurs potentielles

### Pourquoi c'est important ?
- **Cohérence** : Tout le code suit les mêmes règles de formatage
- **Maintenabilité** : Code plus facile à lire et à modifier
- **Prévention d'erreurs** : Détecte les problèmes avant qu'ils deviennent des bugs

### Ce qu'on vérifie :

#### PHP CS Fixer
```bash
vendor/bin/php-cs-fixer fix --dry-run
```
**Exemples de vérifications :**
- Indentation correcte (espaces vs tabulations)
- Placement des accolades
- Espaces autour des opérateurs
- Import des classes non utilisées

#### PHPStan (niveau 6)
```bash
vendor/bin/phpstan analyse src --level=6
```
**Exemples de détections :**
- Variables non définies
- Appels de méthodes sur des objets null
- Types incorrects dans les paramètres
- Code mort (jamais exécuté)

---

## 🛡️ Job 3 : Sécurité

### Qu'est-ce qu'on fait ?
- On scanne toutes nos **dépendances PHP** (packages Composer)
- On vérifie s'il y a des **vulnérabilités connues**

### Pourquoi c'est important ?
- **Protection** : Évite d'utiliser des packages avec des failles de sécurité
- **Mise à jour proactive** : Alerte quand une dépendance devient dangereuse
- **Conformité** : Respecte les bonnes pratiques de sécurité

### Comment ça marche ?
```yaml
- name: Security Check
  uses: symfonycorp/security-checker-action@v5
```

**En pratique :**
- Analyse le fichier `composer.lock`
- Compare avec une base de données de vulnérabilités
- Signale les packages à risque
- Propose les versions correctives

---

## 🎨 Job 4 : Assets Frontend

### Qu'est-ce qu'on fait ?
- Installation des dépendances **Node.js/npm**
- **Compilation** des assets (CSS, JavaScript)
- **Tests frontend** (si configurés)

### Pourquoi c'est important ?
- **Intégrité** : S'assurer que les fichiers CSS/JS se compilent sans erreur
- **Performance** : Vérifier que les assets sont optimisés
- **Compatibilité** : Tester le JavaScript sur différents navigateurs

### Étapes :
```bash
npm ci              # Installation reproductible
npm run build       # Compilation des assets
npm test           # Tests JavaScript
```

---

## 📊 Résultats et Feedback

### Où voir les résultats ?
1. **Dans GitHub** : Onglet "Actions" de votre repository
2. **Dans les Pull Requests** : Statut vert ✅ ou rouge ❌
3. **Annotations** : Commentaires automatiques sur les lignes problématiques
4. **Codecov** : Rapports de couverture de code détaillés

### Que faire si la CI échoue ? ❌

#### Tests qui échouent
- Regarder les logs pour voir quel test échoue
- Corriger le code ou le test
- Re-pousser le code

#### Problèmes de qualité
- Lancer `composer cs-fix` pour corriger le formatage
- Corriger les erreurs PHPStan signalées
- Vérifier avec `composer phpstan` en local

#### Vulnérabilités de sécurité
- Mettre à jour les packages concernés
- Chercher des alternatives si pas de correctif

#### Erreurs d'assets
- Vérifier la compilation en local avec `npm run build`
- Corriger les erreurs JavaScript/CSS

---

## 🚀 Avantages de cette CI Version 1

### Pour le développeur
- **Confiance** : "Mon code est testé automatiquement"
- **Gain de temps** : Plus besoin de vérifications manuelles
- **Apprentissage** : Amélioration continue grâce aux feedbacks

### Pour l'équipe
- **Stabilité** : Moins de bugs en production
- **Standards** : Tout le monde suit les mêmes règles
- **Collaboration** : Pull Requests de meilleure qualité

### Pour le projet
- **Fiabilité** : Code plus robuste
- **Maintenabilité** : Facilite les évolutions futures
- **Sécurité** : Protection contre les vulnérabilités

---

## 🔧 Configuration requise

### Fichiers de configuration à créer :

1. **`.php-cs-fixer.dist.php`** : Règles de formatage PHP
2. **`phpstan.neon`** : Configuration de l'analyse statique
3. **`package.json`** : Dépendances et scripts Node.js
4. **`.env.test`** : Variables d'environnement pour les tests

### Dépendances à installer :
```bash
composer require --dev friendsofphp/php-cs-fixer phpstan/phpstan phpstan/phpstan-symfony
```

---

## 🎯 Prochaines étapes (CI Version 2)

Cette CI Version 1 est une base solide. Pour l'améliorer, on pourra ajouter :

- **Déploiement automatique** en staging/production
- **Tests d'intégration** plus poussés
- **Analyse de performance** automatisée
- **Notifications** Slack/Discord
- **Tests de charge** automatisés
- **Scan de sécurité** plus approfondi

---

## 💡 Conseils pour bien utiliser cette CI

1. **Testez localement** avant de pousser
2. **Lisez les messages d'erreur** de la CI
3. **Maintenez vos dépendances** à jour
4. **Écrivez des tests** pour nouveau code
5. **Respectez les standards** de codage

Cette CI Version 1 vous donne une excellente fondation pour développer sereinement ! 🎉

---

## 🐛 Corrections appliquées

### Problème de compatibilité PHPUnit (Résolu)

**Problème initial :**
```
phpunit/phpunit 12.4.0 requires php >=8.3 -> your php version (8.2.29) does not satisfy that requirement
```

**Cause :** PHPUnit 12.4+ nécessite PHP 8.3+, mais notre CI testait sur PHP 8.2 et 8.3.

**Solution appliquée :**
- Downgrade de PHPUnit de `^12.4` vers `^11.0` dans `composer.json`
- PHPUnit 11 est compatible avec PHP 8.2+ et offre les mêmes fonctionnalités essentielles

**Commandes pour résoudre localement :**
```bash
# Modifier composer.json pour utiliser PHPUnit 11
composer require --dev phpunit/phpunit:^11.0

# Mettre à jour le lock file
composer update phpunit/phpunit

# Vérifier que les tests fonctionnent
php bin/phpunit
```

### Option deprecated --no-suggest (Résolu)

**Problème :**
```
You are using the deprecated option "--no-suggest". It has no effect and will break in Composer 3.
```

**Solution :** Suppression de `--no-suggest` de toutes les commandes `composer install` dans le fichier CI.

Ces corrections garantissent que notre CI Version 1 fonctionne correctement sur PHP 8.2 et 8.3 ! ✅