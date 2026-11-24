# Auditeur de Sécurité CI/CD et Durcissement de la Chaîne Logistique Logicielle

[![Live Demo](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-5b3cf5.svg)](https://tavardt-france.github.io/ci-cd-security-auditor-fr/)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Standards](https://img.shields.io/badge/Standards-CIS%20%7C%20OWASP%20CI%2FCD-blue.svg)](https://tavardt-france.github.io/ci-cd-security-auditor-fr/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Guide technique d'ingénierie et centrale corporative de cybersécurité défensive pour l'audit et la sécurisation des pipelines d'automatisation (GitHub Actions, GitLab CI) et la protection contre les attaques de Software Supply Chain.

***

## Sommaire Exécutif

Ce dépôt fournit la spécification architecturale de sécurité défensive, la matrice technique comparative des postures d'exécution et un utilitaire CLI déterministe développé en Python pour la validation structurelle des manifestes YAML de CI/CD.

* **Organisation:** TAVARDT-France
* **Hub Officiel:** [https://tavardt-france.github.io/ci-cd-security-auditor-fr/](https://tavardt-france.github.io/ci-cd-security-auditor-fr/)
* **Normes de Référence:** CIS Software Supply Chain Security Guide, OWASP Top 10 for CI/CD, SLSA Framework Level 3.

***

## Vecteurs Critiques Inspecionnés

1. **Déclencheurs Privilégiés (`pull_request_target`):** Détection de l'extraction de branches de forks non fiables dans des workflows disposant d'accès aux secrets.
2. **Permissions Globales Excessives (`write-all`):** Imposition stricte de la politique du moindre privilège (`permissions: contents: read`).
3. **Actions de Tiers sans Épinglage Immuable:** Identification des tags mutables (`@v3`, `@main`) et conversion vers des commits SHA de 40 caractères.
4. **Injection d'Expressions dans les Scripts Shell:** Assainissement des variables de contexte GitHub dans les blocs `run:`.
5. **Filtrage Réseau Sortant (Egress):** Surveillance et blocage de l'exfiltration d'identifiants pendant les compilations.

***

## Exécution de l'Utilitaire CLI

Pour exécuter l'audit déterministe directement dans le terminal ou l'intégrer à la chaîne corporative :

```bash
# 1. Télécharger le script officiel
curl -sO https://raw.githubusercontent.com/TAVARDT-France/ci-cd-security-auditor-fr/main/audit_ci_cd_security.py

# 2. Exécuter l'analyse statique sur vos workflows locaux
python audit_ci_cd_security.py .github/workflows/
```

***

© 2026 TAVARDT. Tous droits réservés.
