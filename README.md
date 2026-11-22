# Auditeur de Sécurité CI/CD & Détecteur de Fuite de Secrets | TAVARDT France

[![Licence](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![CI/CD Security](https://img.shields.io/badge/CI%2FCD-Security%20Hardened-00d26a.svg)](README.md)
[![Language](https://img.shields.io/badge/Language-Français-green.svg)](README.md)
[![Engineering](https://img.shields.io/badge/TAVARDT-Cybersecurity-5b3cf5.svg)](https://ag.tavardt.com/fr/ciberseguranca/)

Auditeur statique défensif et linter de sécurité pour les chaînes d'intégration et de déploiement continus (GitHub Actions). Détecte les configurations à haut risque de supply chain, les déclencheurs vulnérables aux pull requests externes, les permissions excessives sur le GITHUB_TOKEN et la fuite de secrets d'entreprise.

## Application Web Interactive

Accédez à l'outil en ligne pour auditer vos manifestes YAML en temps réel dans votre navigateur:

- [https://tavardt-france.github.io/ci-cd-security-auditor-fr/](https://tavardt-france.github.io/ci-cd-security-auditor-fr/)

---

## 1. Vue d'Ensemble de l'Auditeur CI/CD

Les pipelines d'automatisation et d'intégration continue sont devenus la cible prioritaire des cyberattaques de supply chain. Les attaquants compromettent des comptes développeurs et exploitent des déclencheurs mal configurés pour exécuter du code arbitraire sur les runners et dérober des variables protégées, jetons cloud (AWS, GCP, Azure) et clés de déploiement.

L'Auditeur de Sécurité CI/CD TAVARDT inspecte les manifestes YAML de façon passive et déterministe en confrontant chaque règle à l'OWASP Top 10 pour la sécurité CI/CD et au référentiel SLSA.

---

## 2. Vecteurs de Vulnérabilité Audités

| Vecteur de Risque | Scénario dans le Pipeline | Impact Technique |
| :--- | :--- | :--- |
| **pull_request_target Non Sécurisé** | Le workflow s'exécute dans le contexte de la branche principale et télécharge le code de la PR (`ref: ${{ github.event.pull_request.head.sha }}`). | Exécution de code à distance (RCE) immédiate sur le runner avec accès intégral aux secrets du dépôt. |
| **Permissions write-all Excessives** | Le pipeline utilise `permissions: write-all` ou omet le bloc de restriction des accès au niveau racine. | Le jeton hérite de droits d'écriture complets sur les paquets, releases et le code, facilitant l'altération de la supply chain. |
| **Injection de Commandes Shell en run:** | Des variables de contexte non assainies (ex: `${{ github.event.issue.title }}`) sont insérées directement dans les scripts shell. | Permet l'évasion de syntaxe et l'exécution de commandes système arbitraires sur le runner. |
| **Actions Liées à des Tags Mutables** | Les directives `uses:` ciblent des tags mutables comme `@v3` ou `@main` plutôt que des commit SHAs immuables. | Risque direct d'empoisonnement de dépendances si le compte d'un mainteneur de l'action est compromis. |
| **Exfiltration et Secrets Statiques** | Présence de clés privées en clair ou commandes `curl` transmettant des secrets vers des serveurs tiers. | Compromission immédiate des infrastructures cloud connectées et violation des politiques de conformité. |

---

## 3. Guide d'Installation et Exécution Locale (CLI)

Le scanner s'exécute dans un terminal sans dépendance externe, grâce aux bibliothèques natives de Python 3.

### Cloner le Dépôt

```bash
git clone https://github.com/TAVARDT-France/ci-cd-security-auditor-fr.git
cd ci-cd-security-auditor-fr
```

### Auditer un Fichier YAML Spécifique

```bash
python scanner.py .github/workflows/deploy.yml
```

### Auditer Tout un Dossier avec Rapport JSON

```bash
python scanner.py .github/workflows/ --json
```

---

## 4. Blindage Professionnel et Support Technique

Si l'audit a révélé des faiblesses dans vos pipelines, des secrets exposés ou la nécessité d'une gouvernance Zero Trust:

- **Portail Institutionnel:** [TAVARDT France](https://ag.tavardt.com/fr/ciberseguranca/)
- **Centre d'Outils:** [Outils Défensifs TAVARDT](https://tools.tavardt.com/fr/)
- **Assistance Ingénierie:** [Support Technique et Réponse aux Incidents](https://ag.tavardt.com/fr/contato/)

---

## Licence

Distribué sous licence open source MIT. Voir le fichier [`LICENSE`](LICENSE) pour plus de détails.  
Copyright (c) 2026 TAVARDT France.
