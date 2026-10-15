# Atlas DCF — Dossier de conception du prototype

## 1. Positionnement
Application pédagogique de valorisation des entreprises non financières cotées à la Bourse de Casablanca. Elle vise à rendre une analyse DCF compréhensible, reproductible et documentée. Les noms de trois entreprises servent au parcours de démonstration ; tous les chiffres fournis sont inventés.

Le prototype est une application dans le navigateur, sans serveur de données ni IA connectée. Ouvrir Atlas-DCF-prototype.html dans un navigateur moderne. Aucune installation n’est nécessaire. Les scénarios enregistrés dépendent du stockage local autorisé par le navigateur ; ils ne sont pas partagés entre appareils.

## 2. Écrans et parcours
| Écran | Contenu | Action principale |
|---|---|---|
| Valorisation | Entreprise, hypothèses, valeur par action, graphique, sensibilité, prévisions | Modifier et recalculer |
| Données & sources | Montants, unités, date, source déclarée, import JSON, ajout manuel | Vérifier et renseigner |
| Mes scénarios | Bibliothèque locale avec entreprise, résultat, WACC | Charger, comparer, retirer |
| Assistant IA | Revue simulée d’extraction avec pages et validation | Comprendre le futur parcours |
| Conception & méthode | Formules, conventions, architecture et limites | Expliquer le projet au jury |

Parcours de soutenance : choisir Maroc Telecom → consulter les données fictives → modifier le bêta → observer la valeur → tester le scénario pessimiste → enregistrer → consulter la sensibilité → exporter les prévisions → présenter le parcours IA simulé.

## 3. Paramètres du moteur
- Taux sans risque, bêta, prime de risque actions, prime pays.
- Coût de la dette avant impôt et taux du bouclier fiscal.
- Croissance annuelle du chiffre d’affaires et marge EBIT pour chacune des dix années.
- Marge de long terme ; sa modification aligne 2030–2035 et le terminal.
- Impôt prévisionnel, impôt terminal, investissement net en pourcentage du CA.
- Croissance terminale et ROIC terminal.
- Données d’entreprise : CA initial, marge initiale, trésorerie, dette, actions et cours.

Les paramètres numériques marocains devront être recherchés et sourcés avant un usage réel. Aucun taux fiscal ou financier du prototype n’est présenté comme une valeur officielle.

## 4. Calculs et conventions
Date de valorisation : 31 décembre 2025. Horizon 2026–2035. Unités : millions de MAD et millions d’actions, donc résultat en MAD/action.

Ke = Rf + bêta × ERP + prime pays.
E = cours × nombre d’actions ; D = dette comptable, approximation de la valeur de marché.
WACC = Ke × E/(D+E) + Kd × (1−taux du bouclier fiscal) × D/(D+E).
CA(t) = CA(t−1) × (1+croissance(t)).
EBIT = CA × marge ; NOPAT = EBIT − max(EBIT, 0) × taux d’impôt.
FCFF = NOPAT − investissement net.
VA(FCFF t) = FCFF t / (1+WACC)^(t−0,5).
NOPAT terminal = EBIT terminal − max(EBIT terminal, 0) × impôt terminal.
FCFF terminal = NOPAT terminal × (1−g/ROIC).
VA terminale = [FCFF terminal/(WACC−g)]/(1+WACC)^10.
Valeur d’entreprise = somme des flux actualisés + VA terminale.
Capitaux propres = valeur d’entreprise + trésorerie − dette.
Valeur par action = capitaux propres / nombre d’actions.

L’investissement net agrège capex − amortissements + variation du BFR. Les pertes ne créent pas de crédit d’impôt. La dette et les actions restent constantes. Il n’y a pas de dilution, d’intérêts minoritaires ou d’actifs hors exploitation distincts dans ce prototype.

Le tableau de sensibilité impose pour chaque colonne une même marge de 2030 à 2035 et au terminal. Si l’utilisateur a personnalisé ces six marges séparément, la cellule centrale ne reproduit donc pas nécessairement son scénario courant.

## 5. Règles de validation
- CA initial, cours et actions strictement positifs ; trésorerie et dette non négatives.
- Champs numériques bornés ; valeurs vides ou hors bornes signalées.
- WACC strictement supérieur à g.
- ROIC terminal strictement supérieur à g dans le modèle retenu.
- Sources importées déclaratives, jamais considérées comme vérifiées automatiquement.
- Une valorisation négative est affichée, pas remplacée artificiellement par zéro.
- Les valeurs terminales très importantes font l’objet d’une alerte de lecture.

## 6. Architecture cible pour une version réelle
Interface web → API métier → moteur DCF versionné → base de données relationnelle.
Service d’import séparé → stockage des rapports → extraction texte/OCR → IA structurante → validation humaine → données approuvées.
Les clés API doivent rester côté serveur. Les fichiers nécessitent des limites de taille, un contrôle de format et des règles de conservation. Les données personnelles ne sont pas nécessaires au calcul financier lui-même.

### Entités proposées
| Entité | Champs principaux |
|---|---|
| Entreprise | id, nom, ticker, secteur, devise |
| Rapport | id, entreprise_id, exercice, URL, date de publication, empreinte du fichier |
| Donnée financière | id, rapport_id, indicateur, montant, unité, page, statut, validateur |
| Donnée de marché | entreprise_id, cours, date, source, délai éventuel |
| Valorisation | id, entreprise_id, date, version du moteur, jeu de données, hypothèses |
| Scénario | id, valorisation_id, auteur_id, nom, hypothèses, résultats, date |
| Revue IA | rapport_id, propositions, confiance, corrections, décision |

### Accès cibles
Visiteur : démonstration. Analyste : ses modèles et scénarios. Validateur : approbation des données. Administrateur : entreprises, accès et journal des modifications. Ces comptes et rôles ne sont pas implémentés dans la maquette.

## 7. IA : utilité et limites
Fonction prioritaire : identifier les postes financiers d’un rapport, proposer des valeurs structurées et conserver la provenance (page, extrait, unité). Le modèle ne calcule pas la valeur DCF à la place du moteur.

Évaluation à prévoir : exactitude des montants, reconnaissance des unités, bonne année, précision des pages, confusion consolidé/social, couverture des postes et taux de corrections humaines. Constituer un petit jeu de rapports annotés à la main ; ne pas inventer un taux de performance avant les essais.

La confiance du modèle ne constitue pas une preuve. La validation humaine est obligatoire avant promotion des données en statut approuvé.

## 8. Périmètre livré / futur
Fonctionnel : DCF, paramètres, trois jeux fictifs, scénarios central/pessimiste/optimiste, sauvegarde locale, chargement, registre des sources, ajout manuel, import JSON, export CSV, impression PDF via navigateur, thème clair/sombre.
Simulé : extraction et revue IA.
Futur : import Excel et PDF réel, couverture élargie de la BVC, prix sourcés, authentification, partage d’équipe, stockage partagé, traçabilité serveur.
Non couvert : banque et assurance, trading, transactions, conseil personnalisé et finance décentralisée opérationnelle.

## 9. Organisation du groupe
| Responsable | Lot | Livrable |
|---|---|---|
| Étudiant 1 | Données | Dictionnaire des postes et rapports sourcés |
| Étudiant 2 | Finance | Modèle DCF et rapprochement avec un calcul indépendant |
| Étudiant 3 | Produit | Parcours, maquettes, accessibilité et ergonomie |
| Étudiant 4 | IA | Extraction expérimentale et évaluation annotée |
| Étudiant 5 | Intégration | Assemblage, validation, démonstration et rapport |

Planning proposé : semaine 1, cadrage et données ; semaine 2, moteur financier ; semaine 3, interface ; semaine 4, expérimentation IA ; semaine 5, intégration et validation ; semaine 6, répétition de la soutenance.

## 10. Critères de réception
Un changement d’hypothèse recalcule les résultats ; un scénario chargé restaure ses données ; un taux terminal invalide bloque les résultats ; les exports indiquent la source et les unités ; les chiffres de démonstration sont identifiables ; aucune extraction simulée n’est présentée comme réelle.

Vérification réalisée : syntaxe JavaScript, six invariants financiers (cas analytique, WACC/g, ROIC/g, division par les actions, effet de la trésorerie, effet du taux), ouverture dans le navigateur, changement de scénario et message de blocage WACC/g. Les integrations externes et la couverture complète de tous les navigateurs ne sont pas validées.

## 11. Inspiration
Référence fonctionnelle fournie par le groupe : https://zi-cong-tan.github.io/DCF-Calculator-Aesthetic/
Le prototype utilise une interface et une implémentation propres. Les données synthétiques n’ont pas été reprises comme des données financières vérifiées.
