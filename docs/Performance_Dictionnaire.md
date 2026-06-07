# Freddie Mac - Performance Data Dictionary

Ce dictionnaire de donnees correspond au fichier Freddie Mac Single-Family Loan-Level Dataset
`sample_svcg_YYYY.txt` / `historical_data_time_YYYYQn.txt`, aussi appele **Monthly Performance Data File**.

Les fichiers n'ont pas de ligne d'en-tete. Les colonnes sont separees par `|` et leur position est fixe.

Source de reference : Freddie Mac, *Single-Family Loan-Level Dataset General User Guide*.

## Header pipe-delimited

```text
loan_sequence_number|monthly_reporting_period|current_actual_upb|current_loan_delinquency_status|loan_age|remaining_months_to_legal_maturity|defect_settlement_date|modification_flag|zero_balance_code|zero_balance_effective_date|current_interest_rate|current_non_interest_bearing_upb|due_date_of_last_paid_installment|mi_recoveries|net_sale_proceeds|non_mi_recoveries|total_expenses|legal_costs|maintenance_and_preservation_costs|taxes_and_insurance|miscellaneous_expenses|actual_loss_calculation|cumulative_modification_cost|interest_rate_step_indicator|payment_deferral_flag|estimated_ltv|zero_balance_removal_upb|delinquent_accrued_interest|delinquency_due_to_disaster|borrower_assistance_status_code|current_month_modification_cost|interest_bearing_upb
```

## Colonnes

| Position | Nom technique | Nom Freddie Mac | Description courte | Type / valeurs principales |
|---:|---|---|---|---|
| 1 | `loan_sequence_number` | Loan Sequence Number | Identifiant unique du pret. | Alphanumerique, format `PYYQnXXXXXXX` |
| 2 | `monthly_reporting_period` | Monthly Reporting Period | Mois de reporting de l'information du pret. | Date `YYYYMM` |
| 3 | `current_actual_upb` | Current Actual UPB | Solde impaye courant du pret, incluant les montants non porteurs d'interet si applicables. | Montant decimal |
| 4 | `current_loan_delinquency_status` | Current Loan Delinquency Status | Statut de delinquence courant. | `0` courant, `1` 30-59 jours, `2` 60-89 jours, `3` 90-119 jours, etc.; `RA` REO Acquisition |
| 5 | `loan_age` | Loan Age | Nombre de paiements planifies depuis l'origine ou depuis modification. | Numerique |
| 6 | `remaining_months_to_legal_maturity` | Remaining Months to Legal Maturity | Nombre de mois restants jusqu'a la maturite legale. | Numerique |
| 7 | `defect_settlement_date` | Defect Settlement Date | Date de reglement d'un defaut de souscription ou servicing. | Date `YYYYMM`, vide si non applicable |
| 8 | `modification_flag` | Modification Flag | Indique une modification du pret. | `Y` modification periode courante, `P` modification periode precedente, vide sinon |
| 9 | `zero_balance_code` | Zero Balance Code | Raison pour laquelle le solde du pret est passe a zero. | `01`, `02`, `03`, `09`, `15`, `16`, `96`, vide sinon |
| 10 | `zero_balance_effective_date` | Zero Balance Effective Date | Date de l'evenement associe au zero balance code. | Date `YYYYMM`, vide si non applicable |
| 11 | `current_interest_rate` | Current Interest Rate | Taux d'interet courant du pret, tenant compte des modifications. | Decimal |
| 12 | `current_non_interest_bearing_upb` | Current Non-Interest Bearing UPB | Partie non porteuse d'interet du solde impaye courant. | Montant decimal |
| 13 | `due_date_of_last_paid_installment` | Due Date of Last Paid Installment (DDLPI) | Date jusqu'a laquelle le principal et les interets planifies ont ete payes. | Date `YYYYMM`, vide si non disponible |
| 14 | `mi_recoveries` | MI Recoveries | Recouvrements issus de l'assurance mortgage insurance en cas de perte de credit. | Montant decimal, vide si non applicable |
| 15 | `net_sale_proceeds` | Net Sale Proceeds | Produit net remis a Freddie Mac apres disposition ou vente du pret. | Montant decimal, `U` inconnu, vide si non applicable |
| 16 | `non_mi_recoveries` | Non MI Recoveries | Recouvrements hors mortgage insurance. | Montant decimal, vide si non applicable |
| 17 | `total_expenses` | Total Expenses | Total des frais supportes par Freddie Mac lors de l'acquisition, maintenance ou disposition. | Montant decimal, vide si non applicable |
| 18 | `legal_costs` | Legal Costs | Frais juridiques associes a la vente de la propriete. | Montant decimal, vide si non applicable |
| 19 | `maintenance_and_preservation_costs` | Maintenance and Preservation Costs | Frais de maintenance, preservation et reparation. | Montant decimal, vide si non applicable |
| 20 | `taxes_and_insurance` | Taxes and Insurance | Taxes et assurances dues associees a la vente de la propriete. | Montant decimal, vide si non applicable |
| 21 | `miscellaneous_expenses` | Miscellaneous Expenses | Autres frais, par exemple title fees, frais administratifs ou auction fees. | Montant decimal, vide si non applicable |
| 22 | `actual_loss_calculation` | Actual Loss Calculation | Perte effective calculee pour certains zero balance codes. | Montant decimal, vide si non applicable |
| 23 | `cumulative_modification_cost` | Cumulative Modification Cost | Cout cumule des modifications ou reports de paiement. | Montant decimal, vide si non applicable |
| 24 | `interest_rate_step_indicator` | Interest Rate Step Indicator | Indique si la modification inclut un step rate. | `Y`, `N`, vide si pret non modifie |
| 25 | `payment_deferral_flag` | Payment Deferral Flag | Indique un report de paiement courant ou precedent. | `Y`, `P`, vide sinon |
| 26 | `estimated_ltv` | Estimated Loan to Value (ELTV) | LTV courant estime via le modele AVM de Freddie Mac. | `1`-`998`, `999` inconnu, vide si non disponible |
| 27 | `zero_balance_removal_upb` | Zero Balance Removal UPB | UPB restant immediatement avant application du zero balance code. | Montant decimal, vide si non applicable |
| 28 | `delinquent_accrued_interest` | Delinquent Accrued Interest | Interets de retard dus au moment du defaut. | Montant decimal, vide si non applicable |
| 29 | `delinquency_due_to_disaster` | Delinquency Due to Disaster | Indique une difficulte liee a une catastrophe. | `Y`, vide sinon |
| 30 | `borrower_assistance_status_code` | Borrower Assistance Status Code | Type de plan d'assistance emprunteur. | `F` forbearance, `R` repayment, `T` trial period, vide sinon |
| 31 | `current_month_modification_cost` | Current Month Modification Cost | Cout de modification du mois courant. | Montant decimal, vide si non applicable |
| 32 | `interest_bearing_upb` | Interest Bearing UPB | Portion porteuse d'interet du solde courant d'un pret modifie. | Montant decimal |

## Codes utiles

### Current Loan Delinquency Status

| Code | Signification |
|---|---|
| `0` | Courant ou moins de 30 jours de retard |
| `1` | 30-59 jours de retard |
| `2` | 60-89 jours de retard |
| `3` | 90-119 jours de retard |
| `4+` | Delinquance plus avancee, par tranches de 30 jours |
| `RA` | REO Acquisition |

### Zero Balance Code

| Code | Signification |
|---|---|
| `01` | Prepaid or Matured, remboursement volontaire ou maturite |
| `02` | Third Party Sale |
| `03` | Short Sale or Charge Off |
| `09` | REO Disposition |
| `15` | Whole Loan Sale |
| `16` | Reperforming Loan Securitization |
| `96` | Defect prior to other termination event |

### Borrower Assistance Status Code

| Code | Signification |
|---|---|
| `F` | Forbearance |
| `R` | Repayment |
| `T` | Trial Period |
| Vide | Aucun plan de workout ou non applicable |

## Exemple fourni

Ligne :

```text
F25Q10000021|202506|142000.00|0|004|356|||||6.625|0.00||||||||||||||56||||||142000.00
```

Mapping :

| Position | Nom technique | Valeur |
|---:|---|---|
| 1 | `loan_sequence_number` | `F25Q10000021` |
| 2 | `monthly_reporting_period` | `202506` |
| 3 | `current_actual_upb` | `142000.00` |
| 4 | `current_loan_delinquency_status` | `0` |
| 5 | `loan_age` | `004` |
| 6 | `remaining_months_to_legal_maturity` | `356` |
| 7 | `defect_settlement_date` | vide |
| 8 | `modification_flag` | vide |
| 9 | `zero_balance_code` | vide |
| 10 | `zero_balance_effective_date` | vide |
| 11 | `current_interest_rate` | `6.625` |
| 12 | `current_non_interest_bearing_upb` | `0.00` |
| 13 | `due_date_of_last_paid_installment` | vide |
| 14 | `mi_recoveries` | vide |
| 15 | `net_sale_proceeds` | vide |
| 16 | `non_mi_recoveries` | vide |
| 17 | `total_expenses` | vide |
| 18 | `legal_costs` | vide |
| 19 | `maintenance_and_preservation_costs` | vide |
| 20 | `taxes_and_insurance` | vide |
| 21 | `miscellaneous_expenses` | vide |
| 22 | `actual_loss_calculation` | vide |
| 23 | `cumulative_modification_cost` | vide |
| 24 | `interest_rate_step_indicator` | vide |
| 25 | `payment_deferral_flag` | vide |
| 26 | `estimated_ltv` | `56` |
| 27 | `zero_balance_removal_upb` | vide |
| 28 | `delinquent_accrued_interest` | vide |
| 29 | `delinquency_due_to_disaster` | vide |
| 30 | `borrower_assistance_status_code` | vide |
| 31 | `current_month_modification_cost` | vide |
| 32 | `interest_bearing_upb` | `142000.00` |

