# PD Modeling Strategy - Freddie Mac Dataset

**Projet** : Outil de Recommandation de Credit - Gamme ECL (PD / LGD / EAD)  
**Dataset** : Freddie Mac Single-Family Loan-Level Dataset  
**Statut** : Decision de cadrage modelisation  
**Priorite** : Behavioral PD apres historique, puis Origination PD

---

## 1. Objectif

Ce document statue sur la maniere de modeliser la probabilite de defaut (PD) a partir
des donnees Freddie Mac. Il isole les variables indispensables a l'origination, les
variables de performance utilisables apres historique, les variables cibles, et les
regles anti-leakage.

Principe directeur :

> Une feature est autorisee uniquement si elle est connue au moment exact ou la prediction est faite.

---

## 2. Angles de modelisation retenus

| Angle | Description | Priorite | Usage |
|---|---|---:|---|
| Behavioral PD | Prediction mensuelle apres observation d'un historique de performance | 1 | Monitoring, servicing, deterioration du risque |
| Origination PD | Prediction au moment de l'octroi du pret | 2 | Baseline underwriting / score initial |
| Dernier suivi de performance | Prediction a partir d'une photographie a date de cutoff | 3 | Scoring portefeuille, uniquement avec cutoff commun |

L'approche "derniere ligne disponible par pret" est exclue comme approche de base,
car elle introduit un risque eleve de leakage : la derniere ligne contient souvent
l'evenement final du pret.

---

## 3. Definition du defaut

Pour la phase MVP, la cible principale est le defaut credit severe.

### Definition cible retenue

Un pret est considere en defaut si, dans l'horizon futur observe, au moins une des
conditions suivantes est vraie :

| Condition | Source | Justification |
|---|---|---|
| `current_loan_delinquency_status >= 3` | Performance | 90+ DPD, definition standard d'un defaut severe |
| `current_loan_delinquency_status = RA` | Performance | REO Acquisition, evenement de defaut avance |
| `zero_balance_code in (02, 03, 09)` | Performance | Third Party Sale, Short Sale / Charge Off, REO Disposition |

### Definitions secondaires possibles

| Variante | Definition | Usage |
|---|---|---|
| Defaut strict | `current_loan_delinquency_status >= 4` ou evenement REO / charge-off | Stress test |
| Defaut early warning | `current_loan_delinquency_status >= 2` | Detection precoce, plus sensible |
| Liquidation default | `zero_balance_code in (02, 03, 09)` uniquement | LGD / pertes finales |

Decision MVP : utiliser la definition principale, puis comparer avec une variante
early warning lors des tests de sensibilite.

---

## 4. Origination PD

### Unite d'observation

Une ligne par pret, issue du fichier d'origination.

### Moment de prediction

Au moment de l'origination ou du premier paiement connu.

### Horizon cible recommande

| Horizon | Recommandation |
|---|---|
| 12 mois | Utile mais peu d'evenements pour certains millesimes |
| 24 mois | Bon compromis |
| 36 mois | Recommande pour le premier modele origination PD |

Target retenue :

```text
origination_default_36m = 1 si le pret fait defaut entre first_payment_date et first_payment_date + 36 mois
origination_default_36m = 0 sinon, si la fenetre d'observation est complete
```

Les prets sans 36 mois d'observation disponible doivent etre exclus de l'entrainement
pour cette cible, ou traites via une approche survival si l'on choisit de gerer la censure.

### Variables indispensables a l'origination

| Variable | Role attendu |
|---|---|
| `credit_score` | Qualite credit initiale de l'emprunteur |
| `first_payment_date` | Ancrage temporel, vintage, calcul de l'horizon |
| `first_time_homebuyer_flag` | Profil emprunteur |
| `maturity_date` | Verification de maturite et duree restante theorique |
| `msa_or_metropolitan_division` | Geographie fine / marche immobilier local |
| `mortgage_insurance_percentage` | Risque initial et protection assurance |
| `number_of_units` | Type d'actif / complexite du bien |
| `occupancy_status` | Residence principale, secondaire ou investissement |
| `original_cltv` | Levier economique total |
| `original_dti` | Capacite de remboursement initiale |
| `original_upb` | Taille initiale de l'exposition |
| `original_ltv` | Levier du pret |
| `original_interest_rate` | Prix du risque / conditions initiales du pret |
| `channel` | Canal d'origination |
| `amortization_type` | Type d'amortissement |
| `property_state` | Geographie macro |
| `property_type` | Type de propriete |
| `postal_code` | Geographie locale, a encoder prudemment |
| `loan_purpose` | Achat, refinance, cash-out selon codification |
| `original_loan_term` | Duree initiale |
| `number_of_borrowers` | Robustesse du foyer emprunteur |
| `seller_name` | Effet vendeur / qualite d'origination, a regrouper |
| `servicer_name` | Effet servicer initial, a regrouper |
| `super_conforming_flag` | Taille / categorie du pret |
| `special_eligibility_program` | Programme specifique |
| `property_valuation_method` | Qualite ou type de valorisation initiale |
| `interest_only_indicator` | Structure de paiement initiale |
| `mi_cancellation_indicator` | Information assurance mortgage insurance |

### Variables d'origination a exclure ou a traiter prudemment

| Variable | Decision | Raison |
|---|---|---|
| `loan_sequence_number` | Exclure des features | Identifiant technique |
| `prepayment_penalty_mortgage_flag` | Optionnel / souvent faible variance | Variable potentiellement peu informative selon millesime |
| `pre_relief_refinance_loan_sequence_number` | Exclure au MVP | Peut relier a des evenements ou programmes specifiques |
| `relief_refinance_indicator` | Optionnel | A valider selon couverture et objectif |
| `postal_code` | Encoder prudemment | Risque de haute cardinalite et surapprentissage geographique |
| `seller_name`, `servicer_name` | Regrouper les rares | Haute cardinalite |

---

## 5. Behavioral PD apres historique

### Unite d'observation

Une observation correspond a un couple :

```text
loan_sequence_number + observation_month_t
```

Un meme pret peut donc produire plusieurs lignes d'entrainement.

### Moment de prediction

A la fin du mois `t`, on utilise uniquement les informations connues jusqu'a `t`.

### Horizon cible recommande

Target principale :

```text
behavioral_default_12m(t) = 1 si le pret fait defaut entre t+1 et t+12
behavioral_default_12m(t) = 0 sinon, si la fenetre t+1 a t+12 est observable
```

Target secondaire :

```text
behavioral_default_6m(t) = 1 si le pret fait defaut entre t+1 et t+6
```

Decision MVP : entrainer d'abord un modele `behavioral_default_12m`, puis tester `6m`
si le signal est trop dilue ou si l'usage metier demande une alerte plus courte.

### Variables d'origination a joindre

Les variables d'origination suivantes sont jointes a chaque observation mensuelle :

| Groupe | Variables |
|---|---|
| Qualite emprunteur | `credit_score`, `first_time_homebuyer_flag`, `number_of_borrowers` |
| Capacite / levier initial | `original_dti`, `original_ltv`, `original_cltv`, `mortgage_insurance_percentage` |
| Pret | `original_upb`, `original_interest_rate`, `original_loan_term`, `amortization_type`, `interest_only_indicator` |
| Bien | `occupancy_status`, `property_type`, `number_of_units`, `property_valuation_method` |
| Geographie | `property_state`, `msa_or_metropolitan_division`, `postal_code` |
| Origination | `loan_purpose`, `channel`, `seller_name`, `servicer_name`, `special_eligibility_program`, `super_conforming_flag` |
| Temps | `first_payment_date`, `maturity_date`, vintage d'origination derive |

### Variables de performance indispensables au mois t

| Variable | Role attendu |
|---|---|
| `monthly_reporting_period` | Date d'observation `t` |
| `current_actual_upb` | Exposition courante |
| `current_loan_delinquency_status` | Signal comportemental principal |
| `loan_age` | Maturite comportementale du pret |
| `remaining_months_to_legal_maturity` | Temps restant |
| `current_interest_rate` | Taux courant, incluant modifications |
| `current_non_interest_bearing_upb` | Montant non porteur d'interet |
| `modification_flag` | Modification connue jusqu'a `t` |
| `payment_deferral_flag` | Report de paiement connu jusqu'a `t` |
| `interest_rate_step_indicator` | Structure de modification connue jusqu'a `t` |
| `estimated_ltv` | Levier courant estime |
| `borrower_assistance_status_code` | Forbearance / repayment / trial period courant |
| `interest_bearing_upb` | Solde porteur d'interet courant |

### Features historiques derivees recommandees

Ces variables doivent etre calculees uniquement avec les mois `<= t`.

| Feature derivee | Description |
|---|---|
| `months_observed` | Nombre de mois observes jusqu'a `t` |
| `ever_30dpd_to_t` | Le pret a deja ete 30+ DPD avant ou a `t` |
| `ever_60dpd_to_t` | Le pret a deja ete 60+ DPD avant ou a `t` |
| `max_dpd_status_to_t` | Pire statut de delinquence observe jusqu'a `t` |
| `delinq_months_3m` | Nombre de mois delinquent sur les 3 derniers mois |
| `delinq_months_6m` | Nombre de mois delinquent sur les 6 derniers mois |
| `delinq_months_12m` | Nombre de mois delinquent sur les 12 derniers mois |
| `current_or_recent_delinquency` | Delinquence courante ou recente |
| `months_since_last_delinquency` | Recence du dernier incident |
| `cure_count_to_t` | Nombre de retours a courant apres delinquence |
| `upb_ratio_to_original` | `current_actual_upb / original_upb` |
| `upb_change_3m` | Variation d'UPB sur 3 mois |
| `upb_change_6m` | Variation d'UPB sur 6 mois |
| `rate_change_from_origination` | `current_interest_rate - original_interest_rate` |
| `estimated_ltv_current` | ELTV courant si disponible |
| `estimated_ltv_missing_flag` | Indicateur de disponibilite ELTV |
| `ever_modified_to_t` | Modification deja observee avant ou a `t` |
| `ever_payment_deferral_to_t` | Report de paiement deja observe avant ou a `t` |
| `ever_borrower_assistance_to_t` | Assistance emprunteur deja observee avant ou a `t` |

### Variables de performance interdites comme features

Ces variables ne doivent pas etre utilisees comme features pour predire le defaut,
sauf si elles sont strictement connues avant `t` et agreges sous forme historique
non future. Au MVP, elles sont exclues des features directes.

| Variable | Raison |
|---|---|
| `zero_balance_code` | Encode souvent la sortie finale du pret |
| `zero_balance_effective_date` | Date de l'evenement final |
| `zero_balance_removal_upb` | Connu au moment de sortie |
| `actual_loss_calculation` | Resultat post-defaut / liquidation |
| `mi_recoveries` | Recouvrement post-defaut |
| `net_sale_proceeds` | Produit de vente post-liquidation |
| `non_mi_recoveries` | Recouvrement post-defaut |
| `total_expenses` | Frais de liquidation |
| `legal_costs` | Frais post-defaut |
| `maintenance_and_preservation_costs` | Frais post-defaut |
| `taxes_and_insurance` | Frais lies a disposition / liquidation |
| `miscellaneous_expenses` | Frais post-defaut |
| `delinquent_accrued_interest` | Montant lie au defaut |
| `current_month_modification_cost` | Cout de modification, risque de signal post-evenement |
| `cumulative_modification_cost` | Peut incorporer consequences de modification / defaut |
| `defect_settlement_date` | Evenement de reglement non predictif general |

Ces variables restent utiles pour definir des cibles LGD/EAD, analyser les pertes, ou
construire des labels, mais pas comme predictors du modele PD.

---

## 6. Comment predire le defaut a partir des historiques

Decision retenue : construire un dataset de type panel mensuel.

### Construction du dataset

Pour chaque pret :

1. Trier les lignes de performance par `monthly_reporting_period`.
2. Pour chaque mois observable `t`, creer une ligne candidate.
3. Joindre les variables d'origination via `loan_sequence_number`.
4. Calculer les features historiques avec les donnees de performance jusqu'a `t`.
5. Calculer la cible en regardant uniquement la fenetre future `t+1` a `t+12`.
6. Exclure les observations qui n'ont pas assez de futur observable pour labelliser proprement la cible.

Schema :

```text
origination features
+ performance snapshot at t
+ historical aggregates <= t
------------------------------------------------
target = default event in (t+1 ... t+12)
```

### Regle de cutoff

Pour eviter le leakage temporel, les splits train / validation / test doivent etre
faits dans le temps, pas aleatoirement ligne par ligne.

Exemple :

| Split | Periode d'observation `t` |
|---|---|
| Train | Jusqu'a 2021-12 |
| Validation | 2022-01 a 2022-12 |
| Test | 2023-01 a 2023-12 |

Les dates exactes seront fixees apres inspection de la couverture reelle du dataset.

### Regle anti-duplication

Comme un meme pret produit plusieurs observations, il faut eviter qu'un meme pret
apparaisse simultanement dans train et test si l'objectif est de mesurer la generalisation
par contrat. Deux validations sont recommandees :

| Validation | But |
|---|---|
| Split temporel | Mesurer la robustesse dans le temps |
| Split par pret ou group-time split | Mesurer la generalisation sur nouveaux contrats |

Decision MVP : commencer par split temporel, puis ajouter un controle par `loan_sequence_number`.

---

## 7. Leakage checklist

Avant d'accepter une feature, repondre a ces questions :

| Question | Si non |
|---|---|
| La variable est-elle connue a la date d'observation `t` ? | Exclure |
| La variable est-elle calculee uniquement avec des donnees `<= t` ? | Exclure |
| La variable encode-t-elle directement une sortie finale du pret ? | Exclure des features, garder pour target/analyse |
| La variable est-elle disponible en production au moment du scoring ? | Exclure ou documenter comme feature offline |
| La variable a-t-elle une cardinalite extreme ? | Encoder/regrouper prudemment |

---

## 8. Decision projet

La strategie officielle pour la premiere phase de modelisation est :

```text
Modele prioritaire : Behavioral PD 12 mois
Unite : loan_sequence_number + monthly_reporting_period
Features : origination + snapshot performance a t + historiques agreges <= t
Target : defaut severe entre t+1 et t+12
Split : temporel, avec controle par loan_sequence_number
Leakage : exclusion stricte des variables de sortie, perte, recouvrement et liquidation
```

Le modele Origination PD sera construit ensuite comme baseline :

```text
Modele secondaire : Origination PD 36 mois
Unite : une ligne par pret
Features : donnees connues a l'origination
Target : defaut severe dans les 36 mois suivant first_payment_date
```

