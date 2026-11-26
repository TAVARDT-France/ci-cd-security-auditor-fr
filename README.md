<p align="center">
  <a href="https://ag.tavardt.com/fr/">
    <img src="https://ag.tavardt.com/img/tavardt-banner.png" alt="TAVARDT" width="1024" />
  </a>
</p>

# Auditeur de Sécurité CI/CD et Durcissement de la Chaîne Logistique Logicielle

[![Live Demo](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-5b3cf5.svg)](https://tavardt-france.github.io/ci-cd-security-auditor-fr/)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Standards](https://img.shields.io/badge/Standards-CIS%20%7C%20OWASP%20CI%2FCD-blue.svg)](https://tavardt-france.github.io/ci-cd-security-auditor-fr/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Guide corporatif d'ingénierie défensive, spécifications d'immunisation des pipelines d'automatisation et utilitaire d'audit déterministe en ligne de commande pour la protection de la chaîne logistique logicielle sous GitHub Actions et les plateformes modernes de CI/CD.

***

## Vue d'Ensemble Exécutive: La Menace sur les Pipelines CI/CD

Les pipelines d'Intégration Continue et de Déploiement Continu (CI/CD) constituent le cœur névralgique du développement logiciel moderne. Disposant d'accès directs aux environnements de production, de privilèges d'écriture sur les référentiels de code et de jetons d'infrastructure, les chaînes de compilation sont devenues la cible prioritaire des cyberattaques ciblant la chaîne logistique logicielle.

Lorsque les équipes techniques privilégient la rapidité de livraison au détriment des barrières de sécurité périmétrique, les processus héritent de vulnérabilités architecturales sévères:
* Des déclencheurs de compilation exécutent du code externe non vérifié au sein d'environnements privilégiés.
* Des composants et actions tierces utilisent des balises de version mutables sujettes au détournement de compte.
* Les jetons d'exécution héritent de permissions globales excessives sur l'ensemble du référentiel.
* Les nœuds d'exécution (runners) fonctionnent sans filtrage réseau en sortie, facilitant la fuite de secrets.

Ce dépôt définit une norme de défense en profondeur pour les entreprises. Il associe des modèles de configuration prêts pour la production à un outil d'analyse statique déterministe en Python (`audit_ci_cd_security.py`) garantissant une conformité technique sans télémétrie externe.

***

## Surfaces d'Attaque Critiques dans les Pipelines d'Automatisation

### 1. La Vulnérabilité Majeure du Déclencheur `pull_request_target`
Les processus amorcés par `pull_request_target` s'exécutent dans le contexte privilégié du référentiel principal et accèdent directement aux secrets de production:
* **Mécanisme de la Faille:** Tout workflow combinant cet événement avec l'extraction du commit de la demande (`github.event.pull_request.head.sha`) injecte du code soumis par des tiers au sein du runner privilégié.
* **Impact Opérationnel:** Des attaquants externes peuvent modifier les scripts de compilation, exécuter des commandes arbitraires et dérober des identifiants sans validation préalable.
* **Directive Défensive:** N'effectuez jamais l'extraction de code non vérifié sous l'événement `pull_request_target`. Isolez les vérifications dans des événements classiques `pull_request`.

### 2. Actions Tierces sans Verrouillage par Hash SHA Immuable
L'appel d'actions externes au moyen de balises de version simples (telles que `@v3` ou `@main`) introduit une dépendance directe envers des références modifiables:
* **Mécanisme de la Faille:** Les balises Git sont des pointeurs mutables susceptibles d'être réassignés si le compte d'un mainteneur est compromis.
* **Impact Opérationnel:** Un commit malveillant inséré dans une action répandue contamine instantanément et simultanément des milliers de pipelines corporatifs.
* **Directive Défensive:** Verrouillez impérativement chaque action externe au moyen de son empreinte de commit SHA de 40 caractères, accompagnée d'un commentaire documentaire.

### 3. Jetons d'Exécution aux Permissions Excessives (`permissions: write-all`)
Les automatisations s'exécutent fréquemment avec les autorisations par défaut définies au niveau de l'organisation:
* **Mécanisme de la Faille:** L'omission d'une section déclarative `permissions:` confère au jeton des droits d'écriture intégraux sur le code, les paquets et les versions publiées.
* **Impact Opérationnel:** Une dépendance compromise durant la phase de compilation peut corrompre les branches protégées ou diffuser des versions altérées.
* **Directive Défensive:** Définissez explicitement `permissions: contents: read` au sommet du manifeste, en réservant les droits d'écriture aux seules tâches isolées qui l'exigent.

### 4. Absence de Contrôle du Réseau Sortant sur les Runners
Les environnements d'exécution disposent généralement d'un accès sans restriction vers l'extérieur:
* **Mécanisme de la Faille:** Des scripts hostiles peuvent ouvrir des canaux distants et transmettre les variables d'environnement vers des serveurs tiers.
* **Directive Défensive:** Mettez en place un filtrage réseau en sortie (egress filtering) sur le runner afin de bloquer toute communication non dirigée vers les registres officiels.

***

## Modèles de Durcissement pour la Production

Ce dépôt fournit des configurations éprouvées prêtes pour une mise en œuvre en environnement corporatif:

### 1. Workflow Sécurisé pour GitHub Actions (`hardened-workflow.yml`)
Applique le principe du moindre privilège, le verrouillage par hash SHA et l'isolation des secrets:

```yaml
name: Enterprise Hardened Pipeline

on:
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  audit-and-build:
    runs-on: ubuntu-latest
    steps:
      - name: Hardened Checkout
        uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
        with:
          persist-credentials: false

      - name: Initialisation Securisee de Python
        uses: actions/setup-python@0a5c61591373683505ea898e09a3ea4f39ef2b9c # v5.0.0
        with:
          python-version: '3.11'
          cache: 'pip'

      - name: Execution de l Analyse Statique
        run: |
          python audit_ci_cd_security.py .github/workflows/
```

### 2. Script de Vérification du Verrouillage des Actions (`action-pinning.sh`)
Contrôle les manifestes de pipeline et s'assure de l'emploi d'empreintes SHA de 40 caractères:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Analyse les fichiers de workflow et verifie l emploi de hashes SHA de 40 caracteres
find .github/workflows -type f \( -name "*.yml" -o -name "*.yaml" \) | while read -r workflow; do
  echo "[*] Audit du workflow: ${workflow}"
  grep -E "uses: [a-zA-Z0-9_\-\.\/]+@[a-f0-9]{40}" "${workflow}" || echo "[!] Actions non verrouillees detectees"
done
```

### 3. Filtrage du Trafic Sortant sur le Nœud de Compilation (`step-security-harden.yml`)
Interdit l'établissement de connexions externes non autorisées depuis l'exécuteur:

```yaml
- name: Securiser le Trafic Sortant
  uses: step-security/harden-runner@63c24ba6bd7ba022e95695ff85de572c04a18142 # v2.7.0
  with:
    egress-policy: block
    allowed-endpoints: >
      github.com:443
      api.github.com:443
      pypi.org:443
      files.pythonhosted.org:443
```

***

## Utilitaire Déterministe d'Audit CLI

Les navigateurs web imposent des mécanismes stricts d'isolation qui interdisent l'accès au système de fichiers local et l'examen direct des configurations système. Pour permettre des audits techniques reproductibles et dépourvus de télémétrie externe, TAVARDT fournit un analyseur statique en Python conçu pour les terminaux locaux et les chaînes d'intégration continue.

### Fonctionnement de l'Analyseur Statique

L'utilitaire (`audit_ci_cd_security.py`) réalise quatre vérifications déterministes:
* **Audit des Déclencheurs:** Analyse les déclarations d'événements pour signaler les combinaisons dangereuses de `pull_request_target`.
* **Contrôle des Permissions:** Détecte les politiques `write-all` ou l'absence de clauses restrictives de permissions.
* **Vérification du Verrouillage:** Inspecte les instructions `uses:` au moyen d'expressions régulières afin de certifier l'emploi de hashes SHA de 40 caractères.
* **Détection de Secrets en Clair:** Scrutinise les commandes de terminal et les variables à la recherche d'identifiants exposés.

### Guide d'Exécution

Téléchargez et lancez le programme directement sur les manifestes d'automatisation de votre référentiel:

```bash
# 1. Telecharger le script officiel
curl -sO https://raw.githubusercontent.com/TAVARDT-France/ci-cd-security-auditor-fr/main/audit_ci_cd_security.py

# 2. Executer l analyse statique sur vos workflows locaux
python audit_ci_cd_security.py .github/workflows/
```

***

## Foire Aux Questions Techniques (FAQ Architecture & Sécurité)

### 1. Pourquoi le déclencheur pull_request_target constitue-t-il l'un des risques les plus critiques sous GitHub Actions?
L'événement `pull_request_target` s'exécute dans le contexte privilégié du référentiel de base avec un accès direct aux secrets de production. Lorsque le flux extrait du code soumis depuis un fork externe, des tiers non autorisés peuvent exécuter des commandes arbitraires et subtiliser des identifiants corporatifs.

### 2. Comment le verrouillage des actions par hash SHA de 40 caractères prévient-il les attaques logistiques?
Les balises comme `@v3` ou `@main` sont des pointeurs Git mutables pouvant être réassignés si le compte d'un créateur est piraté. Une empreinte de commit SHA de 40 caractères est cryptographiquement immuable, garantissant que seule la version exacte préalablement auditée sera exécutée.

### 3. Quel est l'impact de l'omission d'une déclaration de permissions dans un workflow?
En l'absence d'une section `permissions:` explicite, le jeton du runner hérite des autorisations par défaut de l'organisation, incluant souvent des privilèges d'écriture étendus. Une dépendance compromise durant la compilation peut exploiter ce jeton pour altérer le code ou injecter des versions malveillantes.

### 4. Comment le filtrage réseau en sortie (egress filtering) bloque-t-il la fuite d'identifiants?
Le filtrage en sortie restreint les communications du runner aux seuls domaines officiels préalablement autorisés. Si une dépendance compromise tente d'ouvrir une connexion inverse ou d'émettre des variables vers un serveur de commande distant, la liaison est interrompue immédiatement au niveau du réseau.

### 5. L'analyseur statique en Python nécessite-t-il une connexion internet ou des privilèges d'administration?
Non. L'outil fonctionne de manière entièrement passive et hors ligne, lisant les fichiers YAML locaux directement sur le disque sans émettre de télémétrie et sans exiger de privilèges d'administrateur système.

***

## Cadres de Conformité et Normes d'Entreprise

Architecture de sécurité alignée sur le guide OWASP Top 10 CI/CD Security Risks, le framework de référence SLSA et les exigences d'intégrité logicielle de l'ANSSI.

***

## Liens Utiles

* **Portail Entreprise:** https://ag.tavardt.com/fr/
* **Suite de Diagnostic:** https://tools.tavardt.com/fr/
* **Contact Technique:** https://ag.tavardt.com/fr/contact/

***

© 2026 TAVARDT. Tous droits réservés.
