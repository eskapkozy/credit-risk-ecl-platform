# Data Framing Report (DFR)
**Projet** : Outil de Recommandation de Crédit — Gamme ECL (PD / LGD / EAD)
**Client** : Fintech — Transfert de Fonds International
**Rédacteur** : Équipe Consulting
**Version** : 1.0
**Statut** : Validé client — Prêt pour Epic 0.3
**Date** : Juin 2025

---

## Résumé Exécutif

Ce document formalise les décisions prises lors de l'Epic 0.2 concernant la qualification du dataset public retenu pour la phase de recherche. Il atteste que le dataset Freddie Mac Single Family Loan-Level Dataset est apte à entrer en phase de modélisation, sous réserve des conditions et compromis documentés ci-dessous.

---

## 1. Dataset Retenu

| Élément | Détail |
|---|---|
| Nom | Freddie Mac Single Family Loan-Level Dataset |
| Source | Freddie Mac (public) |
| Domaine | Crédit immobilier résidentiel américain |
| Granularité | Mensuelle par contrat |
| Usage | Phase de recherche uniquement |

---

## 2. Validation des Critères BAD v1.1

| Critère | Statut | Commentaire |
|---|---|---|
| Variable cible binaire | ✅ Validé | Variable de défaut présente et exploitable |
| Suivi longitudinal mensuel | ✅ Validé | Suivi mensuel par contrat, non agrégé |
| Trajectoire complète du contrat | ✅ Validé | De l'origination jusqu'au défaut ou clôture |
| Variables PD | ✅ Validé | Score de crédit, LTV, DTI, DPD, profil emprunteur |
| Variables LGD | ⚠️ Partiel | Montants dus présents, recouvrement à affiner en modélisation |
| Variables EAD | ⚠️ Partiel | Évolution encours disponible, tirage supplémentaire à confirmer |
| Compatibilité IFRS 9 | ✅ Validé | Suivi temporel compatible staging S1/S2/S3 |
| Temporalité | ✅ Validé | Données non agrégées, temporalité mensuelle respectée |
| Représentativité | ⚠️ Compromis | Voir section 4 |

---

## 3. Gestion du Leakage

Les variables suivantes sont identifiées comme sujettes au leakage et doivent être exclues des features d'entraînement. La règle appliquée est la suivante :

> **Une variable est du leakage si elle n'est pas connue au moment où la décision de crédit est prise.**

| Variable | Raison | Usage alternatif |
|---|---|---|
| `interest_rate` | Fixé après décision de crédit | Aucun |
| `loan_age` | Calculé après origination | Aucun |
| `current_upb` | Évolue après déblocage | Feature LGD (post-origination) |
| `modification_flag` | Survient après difficulté de paiement | Signal LGD |
| `zero_balance_code` | Connu à la clôture du contrat | Variable cible LGD |
| `foreclosure_date` | Post-défaut par définition | Variable cible LGD |
| `disposition_date` | Post-liquidation | Variable cible LGD |

**Variables utilisables pour le PD** : profil emprunteur, score de crédit initial, LTV, DTI, type de bien, zone géographique, montant du prêt, durée du prêt.

**Clarification post-cadrage modélisation** : la règle de leakage dépend du moment de prédiction. Une variable post-origination reste exclue du modèle **Origination PD**, mais peut devenir admissible dans le modèle **Behavioral PD** si elle est connue à la date d'observation `t` et calculée sans information future. La stratégie détaillée, les variables retenues et les cibles PD sont documentées dans `docs/PD_Modeling_Strategy.md`.

---

## 4. Représentativité — Compromis Validé

Le dataset Freddie Mac est un dataset de crédit immobilier résidentiel américain. Il ne correspond pas directement au profil des clients de la fintech (population peu bancarisée, transferts internationaux, crédit à la consommation).

**Compromis acté en session client :**

- Le dataset est retenu pour la phase de recherche uniquement
- Il permet de valider l'architecture, les pipelines et la logique de modélisation ECL
- Représentativité estimée à ~50% des objectifs finaux
- En phase production, les modèles devront être ré-entraînés sur des données représentatives de la clientèle réelle de la fintech

---

## 5. Mode d'Ingestion & Serving

**Décision architecturale validée en session client :**

L'ingestion et le serving sont deux couches distinctes. L'architecture retenue est hybride dès le MVP — batch et stream coexistent, chacun activé selon le contexte d'appel.

| Couche | Mode | Implication |
|---|---|---|
| Ingestion des données | Hybride | Pipeline capable de recevoir données en batch ET en stream |
| Serving des recommandations | Hybride | API répondant en batch ET en temps réel selon contexte |
| Latence | Contrainte de conception | Optimisée dès le MVP, pas une optimisation future |

> **Principe retenu** : on ne construit pas un système batch à faire évoluer. On construit une architecture hybride où la latence est une contrainte dès le départ.

---

## 6. Feu Vert pour la Modélisation

| Composante | Statut |
|---|---|
| PD | ✅ Feu vert — données suffisantes |
| LGD | ⚠️ Feu vert conditionnel — variables à affiner en modélisation |
| EAD | ⚠️ Feu vert conditionnel — exposition à confirmer |

---

## 7. Points Transmis aux Epics Suivants

- Affiner la couverture LGD et EAD lors de la phase de modélisation
- Confirmer la liste de features définitive après exploration approfondie du dataset, sur la base de `docs/PD_Modeling_Strategy.md`
- Prévoir une stratégie de ré-entraînement sur données réelles en phase production
- L'architecture hybride batch/stream est une contrainte à intégrer dès l'Epic 0.3 Architecture Decision

---

*Document versé dans le repo projet*
*Constitue le feu vert officiel pour l'entrée en phase de modélisation sous conditions documentées*
