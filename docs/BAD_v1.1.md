# Business Assumption Document (BAD)
**Projet** : Outil de Recommandation de Crédit — Gamme ECL (PD / LGD / EAD)
**Client** : Fintech — Transfert de Fonds International
**Rédacteur** : Équipe Consulting
**Version** : 1.1 — Post-validation client
**Statut** : Validé client — Prêt pour Epic 0.2
**Date** : Juin 2025

---

## Résumé Exécutif

Ce document formalise les hypothèses métier, réglementaires, stratégiques et techniques issues du cadrage client. Il constitue le livrable fondateur du pré-projet et sert de référentiel partagé pour toutes les phases suivantes.

Le client est une fintech spécialisée dans le transfert de fonds international, souhaitant développer un outil de recommandation de crédit reposant sur une gamme complète de modèles ECL (PD, LGD, EAD). La version 1.1 intègre les clarifications obtenues lors de la session de relecture client, notamment les exigences précises sur la donnée.

---

## Axe 1 — Business Context

### 1.1 Business Context

| Élément | Détail |
|---|---|
| Secteur | Fintech — Services financiers |
| Activité principale | Transferts de fonds internationaux |
| Nouveau produit | Outil de recommandation de crédit |
| Positionnement | Entrée agressive sur le marché du crédit |
| Horizon MVP | 2 semaines (prototype opérationnel) |

**Problématique centrale**

Le client souhaite protéger ses clients et se prémunir contre les pertes liées aux défauts de paiement. Il ambitionne de déployer une suite complète de modèles ECL dont les trois composantes interagissent pour produire une recommandation de crédit personnalisée :

- **Modèle PD** : Prédiction de la probabilité de défaut
- **Modèle LGD** : Estimation de la perte en cas de défaut
- **Modèle EAD** : Estimation de l'exposition au moment du défaut

### 1.2 Product Vision

| Phase | Description |
|---|---|
| Phase 1 — Prototype (J0 + 2 semaines) | Interface simple conteneurisée (Docker). Modèles ECL opérationnels. Validation interne. |
| Phase 2 — Web App (évolution) | Application web complète avec interface ergonomique. Recommandation batch et temps réel. |

L'outil a une vocation initiale de recherche interne, avec pour objectif de tester et améliorer les modèles avant adoption en production.

---

## Axe 2 — Regulatory & Risk Context

### 2.1 Regulatory Scope

| Référentiel | Exigence |
|---|---|
| IFRS 9 | Provisionnement des pertes attendues (ECL). Exige des modèles PD, LGD et EAD robustes et auditables. Notion de staging (S1/S2/S3) et suivi de la détérioration du risque sur horizon temporel. |
| Bâle III | Cadre prudentiel définissant les exigences en fonds propres, les méthodes de calcul du risque de crédit et les standards de modélisation interne (approche IRB). |

### 2.2 Risk Modeling Scope

Périmètre de modélisation couvrant l'ensemble de la chaîne ECL :

- **PD** : Modèle prédictif de défaut, seuil de base fixé à 0,28
- **LGD** : Modélisation des pertes en cas de réalisation du défaut
- **EAD** : Estimation de l'exposition effective au moment du défaut

Règles d'orientation de la sortie du modèle :

- Minimisation des Faux Négatifs (FN) — éviter de classer à tort un mauvais payeur comme bon client
- Contraintes réglementaires IFRS 9 et Bâle III

---

## Axe 3 — Decision Strategy

### 3.1 Risk Appetite

| Paramètre | Valeur |
|---|---|
| Seuil de défaut (cutoff) | 0,28 |
| Recall attendu — défauts | ~90% |
| Recall attendu — bons clients | ~80% |
| Tolérance aux FN | Faible — priorité détection défaut sur précision globale |
| Stratégie de marge | Concessions accordées pour garantir une marge de sécurité |

### 3.2 Decision Policy Assumptions

- Prédiction binaire (défaut / non-défaut) avec sortie de probabilité continue
- Seuil de classification ajustable selon les résultats de validation
- Stratégie d'alerte à définir selon les niveaux de perte observés
- Intégration de règles métier dans le composant de recommandation
- Le client délègue à l'équipe la définition de la stratégie de prévention des pertes

---

## Axe 4 — Data Considerations *(mis à jour v1.1)*

### 4.1 Data Acquisition Strategy

Le client ne dispose pas d'historique de données de crédit propriétaire. La stratégie retenue est l'utilisation d'un dataset public pour la phase de recherche.

**Critères de sélection du dataset — v1.1**

Suite à la session de validation client, les critères suivants ont été précisés et sont désormais contraignants :

| Critère | Exigence |
|---|---|
| Suivi longitudinal | Données mensuelles par contrat (DPD, encours) — pas de données agrégées |
| Trajectoire complète | Suivi du contrat de l'origination jusqu'au défaut ou clôture |
| Variables LGD | Montants dus au défaut, montants récupérés, délais de recouvrement, pertes finales |
| Variables EAD | Exposition avant défaut, tirage supplémentaire éventuel, évolution de l'encours |
| Compatibilité IFRS 9 | Notion de staging (S1/S2/S3), horizon temporel, suivi de la détérioration du risque |
| Profil client | Population peu bancarisée, profil international — proche de la clientèle fintech |
| Variable cible | Variable de défaut binaire claire et exploitable |

> **Note v1.1** : Le dataset Home Credit Default Risk (Kaggle) a été évalué et rejeté lors de la session de validation. Il ne fournit pas de suivi longitudinal mensuel, ni les variables nécessaires au LGD et à l'EAD. Il ne permet d'expérimenter que le PD. Deux pistes alternatives sont soumises à l'équipe Data : **Lending Club Loan Data** et **Freddie Mac Single Family Loan-Level Dataset**.

### 4.2 Data Constraints

| Contrainte | Description |
|---|---|
| Data Leakage | L'équipe Data doit identifier et isoler les variables sujettes au leakage. Variables connues uniquement après le défaut à exclure de l'entraînement. |
| Temporalité | Le dataset doit respecter la temporalité inhérente au crédit. Pas d'information future par rapport à la date d'observation dans les données d'entraînement. |
| Représentativité | Se rapprocher du profil de clients fintech (usage transferts, profil international, comportement de paiement). |
| Ingestion | Mode hybride — batch pour recommandations pré-calculées, temps réel pour recommandations dynamiques. |

---

## Axe 5 — System Usage

### 5.1 Target System Vision

| Phase | Description |
|---|---|
| Phase 1 — Conteneur | Interface simple Docker. Modèles ECL exposés via API. Composant de recommandation wrappant les règles métier. |
| Phase 2 — Web App | Application web complète, interface ergonomique, recommandation batch et temps réel. |

### 5.2 Integration Expectations

- Exploitation de l'historique d'usage client pour personnaliser les recommandations
- Recommandation batch au login (produits pré-calculés selon profil)
- Recommandation temps réel selon comportement de navigation
- Composant de recommandation consommant les sorties PD / LGD / EAD en API
- Vision produit combinant logistique et crédit (achat en gros depuis l'étranger)

---

## Synthèse & Points d'Attention

### Hypothèses critiques

| # | Hypothèse | Axe | Statut |
|---|---|---|---|
| 1 | Le seuil de défaut 0,28 sera ajusté post-validation | Axe 3 | À valider |
| 2 | Lending Club ou Freddie Mac répond aux critères data v1.1 | Axe 4 | À valider — Epic 0.2 |
| 3 | L'objectif recall 90%/80% est atteignable avec le dataset retenu | Axe 3 | À valider |
| 4 | La conformité IFRS 9 s'applique dès le prototype ou uniquement en production | Axe 2 | À clarifier |
| 5 | Le mode d'ingestion hybride est faisable dans le délai de 2 semaines | Axe 4 & 5 | À valider |
| 6 | La recommandation batch au login est suffisante pour le MVP Phase 1 | Axe 5 | Assumé |

### Prochaines étapes

1. Clôturer Epic 0.1 — verser le BAD v1.1 dans le repo
2. Ouvrir Epic 0.2 — Data Exploration & Framing
3. Évaluer Lending Club et Freddie Mac au regard des critères data v1.1
4. Produire le Data Framing Report (DFR)

---

*Document versé dans le repo projet — Référentiel officiel du pré-projet*
*Prochaine révision : à l'issue de l'Epic 0.2 si les critères data évoluent*
