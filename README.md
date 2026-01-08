# gitudemy
# GUIDE DE DÉPLOIEMENT ODOO 18 ENTERPRISE
## Mission: Configuration Complète pour Distribution B2B - Cameroun

---

# TABLE DES MATIÈRES

1. [Module Ventes](#1-module-ventes)
2. [Module POS Offline - Fieldsales](#2-module-pos-offline---fieldsales)
3. [Module Stock](#3-module-stock)
4. [Module Achats](#4-module-achats)
5. [Module Comptabilité](#5-module-comptabilité)
6. [Module CRM](#6-module-crm)
7. [Tableaux de Bord et Reporting](#7-tableaux-de-bord-et-reporting)
8. [Harmonisation POS/Ventes](#8-harmonisation-posventes)

---

# 1. MODULE VENTES

## 1.1 Activation et Paramètres Généraux

**Chemin: Paramètres → Ventes**

### Étape 1: Activer les fonctionnalités
| Fonctionnalité | Emplacement | Action |
|----------------|-------------|--------|
| Devis et Commandes | Paramètres Ventes | ☑ Activer |
| Variantes de produits | Paramètres Ventes | ☑ Activer |
| Listes de prix | Paramètres Ventes | ☑ Plusieurs listes de prix |
| Remises | Paramètres Ventes | ☑ Remises sur lignes |
| Marges | Paramètres Ventes | ☑ Afficher les marges |
| Unités de mesure | Paramètres Ventes | ☑ Activer |

### Étape 2: Configuration des délais
```
Chemin: Paramètres → Ventes → Facturation

☑ Facturation automatique: Quantités commandées
☑ Délai de paiement par défaut: 2 jours
☑ Politique de livraison: Dès que possible
```

---

## 1.2 Conditions de Paiement (Max 2 Jours)

**Chemin: Comptabilité → Configuration → Conditions de paiement**

### Créer les conditions de paiement

#### Condition 1: Paiement Comptant
```
Nom: Paiement Comptant
Description: Paiement immédiat à la commande

Termes:
  Type: Solde
  Valeur: 0
  Nombre de jours: 0
  Jour du mois: 0
```

#### Condition 2: Paiement 48H
```
Nom: Paiement 48H
Description: Paiement sous 2 jours maximum

Termes:
  Type: Solde
  Valeur: 0
  Nombre de jours: 2
  Jour du mois: 0
```

#### Condition 3: Paiement à réception
```
Nom: À réception de facture
Description: Paiement dès réception

Termes:
  Type: Solde
  Valeur: 0
  Nombre de jours: 0
  Fin de mois: ☐
```

---

## 1.3 Processus de Vente Complet (Devis → Commande → Facture)

### Flux de vente configuré

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   DEVIS     │ → │  COMMANDE   │ → │  LIVRAISON  │ → │   FACTURE   │
│  (Draft)    │    │ (Confirmed) │    │ (Picking)   │    │ (Invoice)   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
      ↓                  ↓                  ↓                  ↓
   Validation      Confirmation        Validation        Paiement
   commerciale     automatique         des stocks        enregistré
```

### Étape 1: Créer un modèle de devis
**Chemin: Ventes → Configuration → Modèles de devis**

```
Nom du modèle: Devis Standard KAPS
Validité du devis: 7 jours
Confirmation automatique: ☐ Non
Mail de confirmation: ☑ Oui

Lignes par défaut:
  (Optionnel) Produits fréquents

Conditions générales:
  "Conditions de vente KAPS SARL
   - Paiement sous 48 heures maximum
   - Livraison sous 24-48h selon zone
   - Retours acceptés sous 7 jours
   - Prix en FCFA, TVA 19,25% incluse"
```

### Étape 2: Configurer la facturation automatique
**Chemin: Paramètres → Ventes → Facturation**

```
Politique de facturation: 
  ☑ Facturer les quantités livrées (recommandé)
  OU
  ☐ Facturer les quantités commandées

Création automatique de facture:
  ☑ Créer la facture à la validation de la commande
```

---

## 1.4 Géolocalisation des Clients

### Étape 1: Activer la géolocalisation
**Chemin: Paramètres → Paramètres généraux**

```
☑ Géolocalisation (Google Maps / OpenStreetMap)
Clé API Google Maps: [Votre clé API] (optionnel)
```

### Étape 2: Configurer les champs clients
**Chemin: Contacts → [Client] → Onglet Adresse**

| Champ | Description | Exemple |
|-------|-------------|---------|
| Rue | Adresse complète | Rue 1234, Mendong |
| Ville | Ville | Yaoundé |
| Région | Région | Centre |
| Pays | Pays | Cameroun |
| Latitude | Coordonnées GPS | 3.8480 |
| Longitude | Coordonnées GPS | 11.5021 |

### Étape 3: Segmentation géographique
**Chemin: Contacts → Configuration → Tags de contacts**

Créer les tags de zone:
```
- Zone Centre-Yaoundé
- Zone Littoral-Douala
- Zone Ouest
- Zone Nord
- Zone Sud
```

### Étape 4: Champs personnalisés pour quincailleries
**Chemin: Paramètres → Technique → Champs personnalisés**

```
Modèle: res.partner (Contacts)

Champs à créer:
1. x_type_client
   - Type: Sélection
   - Options: quincaillerie, particulier, entreprise, revendeur
   
2. x_zone_commerciale
   - Type: Sélection
   - Options: zone_nord, zone_sud, zone_est, zone_ouest, zone_centre
   
3. x_credit_autorise
   - Type: Monétaire
   - Devise: XAF
   
4. x_limite_credit_jours
   - Type: Entier
   - Valeur par défaut: 2
```

---

## 1.5 Planification des Tournées Commerciales

### Option A: Utiliser le module Route Planner (si disponible)

**Chemin: Ventes → Tournées**

```
Configuration de tournée:
  Nom: Tournée Zone Yaoundé Centre
  Commercial: [Fieldsale assigné]
  Jour de la semaine: Lundi, Mercredi, Vendredi
  
  Clients de la tournée:
    1. Quincaillerie ABC - 08:00
    2. Quincaillerie XYZ - 09:30
    3. Client Particulier - 11:00
    ...
```

### Option B: Configuration manuelle avec Calendrier

**Chemin: Calendrier → Créer un événement**

```
Pour chaque visite:
  Nom: Visite [Nom Client]
  Début: [Date/Heure]
  Fin: [Date/Heure estimée]
  Responsable: [Commercial]
  Client: [Lier au contact]
  Tags: Tournée commerciale
  
  Rappel: 1 heure avant
```

### Optimisation des tournées
```
Conseils:
1. Regrouper les clients par zone géographique
2. Planifier les visites de 8h à 17h
3. Maximum 8-10 clients par jour
4. Prévoir temps de déplacement (30-45 min entre clients)
5. Prioriser les clients avec commandes en attente
```

---

## 1.6 Alertes de Retard de Paiement

### Étape 1: Configurer les niveaux de relance
**Chemin: Comptabilité → Configuration → Niveaux de relance**

#### Niveau 1: Rappel Amical (Jour J+1)
```
Nom: Rappel J+1
Jours après l'échéance: 1
Action: Envoyer email
Modèle d'email: Rappel paiement - Amical

Contenu email:
"Cher client,
Nous vous rappelons que votre facture [REF] d'un montant de [MONTANT] FCFA 
est arrivée à échéance hier.
Merci de procéder au règlement dans les meilleurs délais.
Cordialement, KAPS SARL"
```

#### Niveau 2: Relance Ferme (J+3)
```
Nom: Relance J+3
Jours après l'échéance: 3
Action: Envoyer email + SMS
Modèle d'email: Relance paiement - Ferme

Contenu:
"URGENT: Facture impayée [REF]
Montant: [MONTANT] FCFA
Retard: [JOURS] jours
Merci de régulariser immédiatement."
```

#### Niveau 3: Mise en demeure (J+7)
```
Nom: Mise en demeure J+7
Jours après l'échéance: 7
Action: Envoyer email + Bloquer le compte
Modèle d'email: Mise en demeure

Action automatique:
  ☑ Bloquer les nouvelles commandes
  ☑ Notifier le responsable commercial
```

### Étape 2: Assigner aux clients
**Chemin: Contacts → [Client] → Onglet Ventes/Achats**

```
Politique de relance: [Sélectionner le niveau créé]
```

---

## 1.7 Traçabilité des Encaissements

### Configuration des modes de paiement
**Chemin: Comptabilité → Configuration → Modes de paiement**

#### Mode 1: Espèces
```
Nom: Espèces
Type: Entrant
Journal: Caisse (57)
Compte de transition: 571 - Caisse siège
Identification bancaire: Non requise
```

#### Mode 2: Virement Bancaire
```
Nom: Virement Bancaire
Type: Entrant
Journal: Banque principale
Compte de transition: 521 - Banque locale
Identification bancaire: Requise
```

#### Mode 3: Chèque
```
Nom: Chèque
Type: Entrant
Journal: Chèques à encaisser
Compte de transition: 512 - Chèques en attente
Identification bancaire: Numéro de chèque
```

#### Mode 4: Mobile Money
```
Nom: MTN Mobile Money
Type: Entrant
Journal: Mobile Money
Compte de transition: 531 - Mobile Money

Nom: Orange Money
Type: Entrant
Journal: Mobile Money
Compte de transition: 532 - Orange Money
```

### Enregistrement des paiements
**Chemin: Comptabilité → Clients → Factures → [Facture] → Enregistrer un paiement**

```
Formulaire de paiement:
  Journal: [Sélectionner le mode]
  Montant: [Montant payé]
  Date de paiement: [Date]
  Mémo: [Référence du paiement]
  
Pour les chèques:
  Numéro de chèque: [N° chèque]
  Banque émettrice: [Nom banque]
  
Pour Mobile Money:
  Référence transaction: [Code transaction]
```

---

## 1.8 Équipes Commerciales

**Chemin: Ventes → Configuration → Équipes commerciales**

### Équipe 1: Fieldsales Mobiles
```
Nom de l'équipe: Fieldsales Terrain
Responsable: [Chef des ventes]
Email: fieldsales@kaps.cm

Membres:
  - Commercial 1 (Zone Yaoundé)
  - Commercial 2 (Zone Douala)
  - Commercial 3 (Zone Ouest)

Canal de vente: POS Mobile
Objectif mensuel: 15 000 000 FCFA
```

### Équipe 2: Point de Vente Fixe
```
Nom de l'équipe: Dovv Mendong
Responsable: [Responsable magasin]
Email: mendong@kaps.cm

Membres:
  - Vendeur 1
  - Vendeur 2

Canal de vente: Ventes directes
Objectif mensuel: 20 000 000 FCFA
```

---

# 2. MODULE POS OFFLINE - FIELDSALES

## 2.1 Installation et Activation

**Chemin: Applications → Rechercher "Point of Sale"**

### Modules à installer
```
1. Point de Vente (point_of_sale) - Module de base
2. POS Restaurant (pos_restaurant) - Pour fonctions avancées
3. Synchronisation hors ligne intégrée dans Odoo 18 Enterprise
```

### Activer le mode hors ligne
**Chemin: Paramètres → Point de Vente**

```
☑ Autoriser le mode hors ligne
☑ Synchronisation automatique
☑ Cache des produits local
```

---

## 2.2 Configuration des 3 POS Mobiles Fieldsales

**Chemin: Point de Vente → Configuration → Points de vente**

### POS 1: Fieldsale Yaoundé
```
Nom: POS Mobile - Yaoundé
Type de point de vente: Mobile

ONGLET GÉNÉRAL:
  Société: KAPS SARL
  Entrepôt: Stock Véhicule Yaoundé
  Liste de prix: Liste prix standard
  Journal des ventes: Journal POS Yaoundé
  
ONGLET PAIEMENTS:
  Modes de paiement:
    ☑ Espèces
    ☑ Mobile Money (MTN/Orange)
    ☑ Chèque
    ☑ Paiement différé (crédit 48h max)
  
ONGLET COMPTABILITÉ:
  Journal de ventes: POS Yaoundé (JV-POS-YDE)
  Compte de revenus: 701 - Ventes de marchandises
  
ONGLET AVANCÉ:
  ☑ Mode hors ligne activé
  ☑ Synchronisation automatique au retour en ligne
  Délai sync max: 24 heures
  ☑ Téléchargement produits pour usage offline
  ☑ Téléchargement clients pour usage offline
  
ONGLET INTERFACE:
  ☑ Interface tactile optimisée
  ☑ Grand format boutons (tablettes)
  ☑ Afficher images produits
  ☑ Recherche rapide par code-barres
  Thème: Clair (meilleure visibilité extérieur)
```

### POS 2: Fieldsale Douala
```
Nom: POS Mobile - Douala
(Configuration identique, changer entrepôt et journal)
  Entrepôt: Stock Véhicule Douala
  Journal: POS Douala (JV-POS-DLA)
```

### POS 3: Fieldsale Zone Ouest
```
Nom: POS Mobile - Ouest
  Entrepôt: Stock Véhicule Ouest
  Journal: POS Ouest (JV-POS-OUEST)
```

---

## 2.3 Configuration Mode Hors Ligne Avancé

### Paramètres de cache local
**Chemin: Point de Vente → Configuration → [POS] → Onglet Avancé**

```
CACHE LOCAL:
  ☑ Activer le cache local
  Produits à télécharger: Tous les produits actifs
  Clients à télécharger: Tous les clients du commercial
  
  Données mises en cache:
    ☑ Produits (nom, code, prix, stock)
    ☑ Catégories de produits
    ☑ Listes de prix
    ☑ Clients (nom, adresse, crédit)
    ☑ Taxes
    ☑ Modes de paiement
    
  Taille max du cache: 500 Mo
  Durée de validité: 7 jours
  
SYNCHRONISATION:
  Mode: Bidirectionnel automatique
  Déclencheur: Connexion WiFi/4G détectée
  Fréquence: Toutes les 5 minutes (en ligne)
  
  Données synchronisées:
    ↑ Vers serveur: Commandes, Paiements, Nouveaux clients
    ↓ Depuis serveur: Stocks, Prix, Nouveaux produits
```

### Gestion des conflits
```
Règles de résolution automatique:

1. STOCKS: Priorité serveur central
2. PRIX: Dernière modification gagne
3. CLIENTS: Fusion des informations
4. COMMANDES: Toujours synchroniser vers serveur
```

---

## 2.4 Gestion des Stocks Mobiles (Véhicules)

### Créer les entrepôts véhicules
**Chemin: Inventaire → Configuration → Entrepôts**

```
Entrepôt Véhicule Yaoundé:
  Nom: Véhicule Fieldsale Yaoundé
  Code: VEH-YDE
  Emplacement stock: VEH-YDE/Stock

Entrepôt Véhicule Douala:
  Nom: Véhicule Fieldsale Douala
  Code: VEH-DLA

Entrepôt Véhicule Ouest:
  Nom: Véhicule Fieldsale Ouest
  Code: VEH-OUEST
```

### Transferts de réapprovisionnement
**Chemin: Inventaire → Opérations → Transferts**

```
Processus quotidien:
1. MATIN: Transfert Entrepôt Principal → Véhicule
2. JOURNÉE: Ventes via POS Mobile
3. SOIR: Synchronisation + Inventaire reliquat
```

---

## 2.5 Moyens de Paiement POS

**Chemin: Point de Vente → Configuration → Modes de paiement**

| Mode | Journal | Compte OHADA | Offline |
|------|---------|--------------|---------|
| Espèces | Caisse POS | 571 | ☑ Oui |
| MTN MoMo | Mobile Money | 531 | ☑ Oui |
| Orange Money | Mobile Money | 532 | ☑ Oui |
| Chèque | Chèques | 512 | ☑ Oui |
| Crédit 48H | Clients | 411 | ☑ Oui |

---

## 2.6 Impression Mobile Bluetooth

**Chemin: Point de Vente → Configuration → [POS] → Matériel**

```
Imprimantes compatibles:
  - Epson TM-P20/P80
  - Star SM-L200
  - HPRT MT800
  
Configuration:
  Connexion: Bluetooth
  Largeur papier: 58mm ou 80mm
  ☑ Impression automatique ticket
  
Format ticket:
  - Logo + Infos société
  - NIU / RCCM
  - Détail vente + TVA 19,25%
  - Mode paiement
  - N° transaction
```

---

## 2.7 Gestion des Retours

```
Configuration retours POS:
  ☑ Autoriser les retours
  ☑ Retours possibles hors ligne
  Délai max: 7 jours
  
Motifs obligatoires:
  - Produit défectueux
  - Erreur de commande
  - Non conforme
```

---

# 3. MODULE STOCK

## 3.1 Paramètres Généraux

**Chemin: Inventaire → Configuration → Paramètres**

```
OPÉRATIONS:
  ☑ Emplacements de stockage
  ☑ Routes en plusieurs étapes
  ☑ Packages
  ☑ Numéros de lot / Série
  
TRAÇABILITÉ:
  ☑ Expiration sur lots
  ☑ Afficher les alertes
  
VALORISATION:
  ☑ Valorisation des stocks
  Méthode: Coût Moyen Pondéré (CMP)
  
ENTREPÔTS:
  ☑ Multi-entrepôts
```

---

## 3.2 Valorisation au Coût Moyen Pondéré (CMP)

**Chemin: Inventaire → Configuration → Catégories de produits**

### Configuration par catégorie
```
Catégorie: Peintures
  Méthode de coût: Coût moyen (AVCO)
  Valorisation des stocks: Automatisée
  Compte de stock: 31 - Marchandises
  Compte de variation: 603 - Variation stocks
  
Catégorie: Enduits
  Méthode de coût: Coût moyen (AVCO)
  ...

Catégorie: Outillage
  Méthode de coût: Coût moyen (AVCO)
  ...
```

### Formule CMP
```
Nouveau CMP = (Stock actuel × CMP actuel + Nouvelle Qté × Nouveau Prix)
              ÷ (Stock actuel + Nouvelle Qté)

Exemple:
  Stock: 100 unités à 1000 FCFA = 100 000 FCFA
  Réception: 50 unités à 1200 FCFA = 60 000 FCFA
  Nouveau CMP = (100 000 + 60 000) ÷ 150 = 1066,67 FCFA
```

---

## 3.3 Gestion des Bonus Fournisseurs

### Créer un type de remise fournisseur
**Chemin: Achats → Configuration → Remises fournisseurs**

```
Type 1: Remise quantitative
  Nom: Bonus volume 5%
  Condition: Achat > 1 000 000 FCFA
  Remise: 5%
  Impact coût: Réduire le coût unitaire

Type 2: Remise fidélité
  Nom: Bonus annuel
  Condition: CA annuel > 10 000 000 FCFA
  Remise: 3%
  Application: Avoir fin d'année
```

### Intégration dans le calcul des coûts
```
Coût réel = Prix achat - Bonus + Frais transport + Droits douane

Configuration produit:
  Prix achat fournisseur: 10 000 FCFA
  Bonus négocié: -5% = -500 FCFA
  Transport: +200 FCFA
  Coût final CMP: 9 700 FCFA
```

---

## 3.4 Unités de Mesure Multiples

**Chemin: Inventaire → Configuration → Unités de mesure**

### Catégorie: Quantité
```
Unité de référence: Unité(s)
Ratio: 1.0

Unités secondaires:
  - Pièce | Ratio: 1.0
  - Seau (5L) | Ratio: 1.0
  - Seau (10L) | Ratio: 1.0
  - Seau (20L) | Ratio: 1.0
```

### Catégorie: Poids
```
Unité de référence: Kilogramme (kg)
Ratio: 1.0

Unités secondaires:
  - Gramme (g) | Ratio: 0.001
  - Sac 25kg | Ratio: 25.0
  - Sac 50kg | Ratio: 50.0
```

### Configuration produit multi-UoM
```
Produit: Peinture Vinylique
  UoM de vente: Seau (20L)
  UoM d'achat: Palette (48 seaux)
  UoM de stock: Seau (20L)
  
  Conversions:
    1 Palette = 48 Seaux
    1 Seau 20L = 4 × Seau 5L
```

---

## 3.5 Seuils de Stock et Alertes

**Chemin: Inventaire → Configuration → Règles de réapprovisionnement**

### Créer une règle par produit
```
Produit: Peinture Satinée Blanche 20L
Entrepôt: Entrepôt Principal KAPS
Emplacement: Stock

Quantité minimum: 50 unités
Quantité maximum: 200 unités
Quantité à commander: Multiple de 10

Délai fournisseur: 7 jours
Jours de sécurité: 3 jours

Action si stock < minimum:
  ☑ Créer demande de prix automatique
  ☑ Envoyer alerte email
```

### Alertes par email
**Chemin: Paramètres → Technique → Automatisation → Actions planifiées**

```
Action: Alerte stock minimum
Fréquence: Quotidienne à 08:00
Modèle: stock.warehouse.orderpoint

Email:
  Destinataire: achats@kaps.cm
  Sujet: [ALERTE] Stocks sous seuil minimum
  Corps: Liste des produits à réapprovisionner
```

---

## 3.6 Structure des Entrepôts

### Entrepôt Principal
```
Nom: Entrepôt Principal KAPS
Code: KAPS
Adresse: [Adresse entrepôt]

Emplacements:
  KAPS/Stock
    ├── KAPS/Stock/Peintures
    ├── KAPS/Stock/Enduits
    ├── KAPS/Stock/Outillage
    └── KAPS/Stock/Accessoires
  KAPS/Entrée
  KAPS/Sortie
  KAPS/Contrôle Qualité
  KAPS/Quarantaine
```

### Entrepôts Véhicules (Fieldsales)
```
Nom: Véhicule Yaoundé
Code: VEH-YDE
Parent: Aucun (indépendant)

Emplacements:
  VEH-YDE/Stock
```

---

## 3.7 Inventaires Périodiques

**Chemin: Inventaire → Opérations → Inventaires**

### Planification des inventaires
```
Type 1: Inventaire tournant (hebdomadaire)
  Fréquence: Chaque lundi
  Catégories: Rotation selon planning
  Semaine 1: Peintures
  Semaine 2: Enduits
  Semaine 3: Outillage
  Semaine 4: Accessoires

Type 2: Inventaire complet (semestriel)
  Fréquence: 30 juin et 31 décembre
  Portée: Tous les produits
  Fermeture: Suspension des opérations
```

### Processus d'inventaire
```
1. Créer l'inventaire
2. Démarrer l'inventaire
3. Scanner/compter les produits
4. Saisir les quantités réelles
5. Valider les écarts
6. Comptabiliser les ajustements
```

---

# 4. MODULE ACHATS

## 4.1 Paramètres Généraux

**Chemin: Achats → Configuration → Paramètres**

```
☑ Demandes de prix
☑ Approbation des commandes
☑ Accord-cadres
☑ Contrôle factures 3 voies
☑ Gestion des variantes
☑ Unités de mesure achat
```

---

## 4.2 Workflow Validation Commandes

**Chemin: Achats → Configuration → Workflow approbation**

```
Règle 1: Commande < 500 000 FCFA
  Approbateur: Responsable achats
  Délai validation: 24h
  
Règle 2: Commande 500 000 - 2 000 000 FCFA
  Approbateur: Directeur commercial
  Délai validation: 48h
  
Règle 3: Commande > 2 000 000 FCFA
  Approbateur: Directeur Général
  Délai validation: 72h
```

---

## 4.3 Gestion Fournisseurs Locaux/Étrangers

**Chemin: Achats → Fournisseurs**

### Fournisseur Local
```
Nom: Peintures Cameroun SA
Type: Fournisseur local
Pays: Cameroun
Devise: XAF
TVA: 19,25%

Conditions:
  Condition de paiement: 30 jours
  Incoterm: EXW (départ usine)
  Délai livraison: 3-5 jours
```

### Fournisseur Étranger
```
Nom: Sigma Paints Europe
Type: Fournisseur étranger
Pays: France
Devise: EUR

Conditions:
  Condition de paiement: LC (Lettre de crédit)
  Incoterm: CIF Douala
  Délai livraison: 45-60 jours
  
Frais additionnels:
  - Droits de douane: 20%
  - Frais transit: Variable
  - Assurance: 1%
```

---

## 4.4 Gestion Multi-Devises

**Chemin: Comptabilité → Configuration → Devises**

### Activer les devises
```
Devises actives:
  ☑ XAF - Franc CFA (principale)
  ☑ EUR - Euro (taux fixe: 655.957)
  ☑ USD - Dollar américain
  
Mise à jour taux:
  Mode: Manuel (EUR fixe) / Automatique (USD)
  Source: Banque Centrale
```

### Commande en devise étrangère
```
Fournisseur: [Fournisseur EUR]
Devise commande: EUR
Taux appliqué: 655.957

Ligne:
  Produit: Peinture Import
  Quantité: 100
  Prix EUR: 25,00 €
  Prix XAF: 16 399 FCFA
```

---

## 4.5 Suivi Livraisons Partielles

**Chemin: Achats → Commandes → [Commande]**

```
Commande: PO00123
Fournisseur: Peintures Europe
Statut: En cours

Lignes:
  Produit A | Cmd: 100 | Reçu: 60 | Reste: 40
  Produit B | Cmd: 50  | Reçu: 50 | Reste: 0 ✓
  Produit C | Cmd: 200 | Reçu: 0  | Reste: 200

Réceptions:
  - BL001: 60 Produit A, 25 Produit B (15/01)
  - BL002: 25 Produit B (20/01)
  - Attendu: 40 Produit A, 200 Produit C
```

---

## 4.6 Contrôle Qualité Réception

**Chemin: Inventaire → Configuration → Contrôle qualité**

```
Point de contrôle: Réception marchandises

Vérifications:
  ☑ Quantité conforme au BL
  ☑ État des emballages
  ☑ Dates de péremption (si applicable)
  ☑ Certificats de conformité
  ☑ Échantillonnage couleur (peintures)

Actions si non-conforme:
  - Quarantaine
  - Notification fournisseur
  - Création réclamation
```

---

## 4.7 Évaluation Fournisseurs

**Chemin: Achats → Rapports → Évaluation fournisseurs**

```
Critères de notation (sur 100):

1. Délai de livraison (30 pts)
   - Livraison à temps: 30 pts
   - Retard < 3 jours: 20 pts
   - Retard > 3 jours: 0 pts

2. Qualité produits (30 pts)
   - Aucun retour: 30 pts
   - Retours < 2%: 20 pts
   - Retours > 2%: 0 pts

3. Compétitivité prix (20 pts)
   - Meilleur prix: 20 pts
   - Prix marché: 15 pts
   - Prix élevé: 5 pts

4. Service client (20 pts)
   - Réactivité, flexibilité, support
```

---

# 5. MODULE COMPTABILITÉ

## 5.1 Plan Comptable OHADA Cameroun

**Chemin: Comptabilité → Configuration → Plan comptable**

(Voir le guide OHADA détaillé dans GUIDE_CONFIGURATION_ODOO19.md)

```
Classes principales:
  1 - Ressources durables
  2 - Actif immobilisé
  3 - Stocks
  4 - Tiers
  5 - Trésorerie
  6 - Charges
  7 - Produits
  8 - Autres charges/produits
```

---

## 5.2 Configuration TVA 19,25%

**Chemin: Comptabilité → Configuration → Taxes**

### TVA Ventes
```
Nom: TVA 19,25% Ventes
Portée: Ventes
Calcul: Pourcentage du prix
Montant: 19.25%
Inclus dans le prix: ☐ Non

Comptes:
  Collecte: 4431 - TVA facturée
  Base: Automatique
```

### TVA Achats
```
Nom: TVA 19,25% Achats
Portée: Achats
Calcul: Pourcentage du prix
Montant: 19.25%

Comptes:
  Déductible: 4432 - TVA récupérable
```

### Retenues à la source
```
AIR 5,5% (Prestataires):
  Type: Retenue
  Taux: 5.5%
  Compte: 4471

Précompte 1% (Achats):
  Type: Retenue
  Taux: 1%
  Compte: 4472
```

---

## 5.3 Commissions Commerciales

### Configuration des commissions
**Chemin: Ventes → Configuration → Commissions**

```
Règle 1: Commission standard
  Applicable: Tous les commerciaux
  Base: Chiffre d'affaires HT encaissé
  Taux: 3%
  
Règle 2: Bonus performance
  Condition: CA mensuel > 10 000 000 FCFA
  Bonus additionnel: 1%
  
Règle 3: Commission nouveaux clients
  Applicable: Première commande client
  Bonus: 2% supplémentaire
```

### Comptabilisation
```
Journal: Commissions (OD)
Compte charge: 6411 - Commissions sur ventes
Compte dette: 421 - Personnel, rémunérations dues

Fréquence: Mensuelle
Calcul: Fin de mois
Paiement: 10 du mois suivant
```

---

## 5.4 Rapprochement Bancaire

**Chemin: Comptabilité → Banque → [Compte] → Rapprochement**

```
Processus:
1. Importer relevé bancaire (format OFX/CSV)
2. Rapprochement automatique par:
   - Montant exact
   - Référence de paiement
   - Date ± 3 jours
3. Rapprochement manuel des écarts
4. Validation et clôture

Fréquence recommandée: Quotidienne
```

---

# 6. MODULE CRM

## 6.1 Paramètres Généraux

**Chemin: CRM → Configuration → Paramètres**

```
☑ Leads (Pistes)
☑ Opportunités
☑ Activités planifiées
☑ Prévisions de ventes
☑ Lead Mining (si disponible)
```

---

## 6.2 Segmentation Clients

**Chemin: Contacts → Configuration → Tags**

### Tags de segmentation
```
TYPE CLIENT:
  - Quincaillerie
  - Particulier
  - Entreprise BTP
  - Revendeur
  - Peintre professionnel

ZONE GÉOGRAPHIQUE:
  - Yaoundé Centre
  - Yaoundé Périphérie
  - Douala
  - Zone Ouest
  - Zone Nord
  - Zone Sud

POTENTIEL:
  - Client VIP
  - Client régulier
  - Nouveau client
  - Client dormant
  - Prospect chaud
```

### Champs personnalisés clients
**Chemin: Paramètres → Technique → Modèles → res.partner**

```
x_type_client (Sélection):
  - quincaillerie
  - particulier
  - entreprise
  - revendeur

x_ca_annuel (Monétaire):
  Devise: XAF

x_frequence_achat (Sélection):
  - hebdomadaire
  - mensuel
  - trimestriel
  - occasionnel

x_commercial_assigne (Many2one → res.users):
  Commercial attitré
```

---

## 6.3 Historique Interactions

**Chemin: Contacts → [Client] → Historique**

```
Types d'activités à configurer:
  - Appel téléphonique
  - Visite terrain
  - Email envoyé
  - Devis envoyé
  - Réclamation
  - Relance paiement

Chaque interaction enregistre:
  - Date/Heure
  - Type d'activité
  - Responsable
  - Résumé
  - Prochaine action
```

---

## 6.4 Géolocalisation et Cartographie

### Configuration Google Maps
**Chemin: Paramètres → Intégrations → Google Maps**

```
Clé API Google Maps: [Votre clé API]
Services activés:
  ☑ Geocoding API
  ☑ Maps JavaScript API
  ☑ Directions API
```

### Affichage carte clients
**Chemin: CRM → Clients → Vue Carte**

```
Affichage:
  - Points clients sur carte
  - Couleur par type (quincaillerie=bleu, particulier=vert)
  - Clustering pour zones denses
  - Filtres par commercial/zone
```

---

## 6.5 Pipeline Commercial

**Chemin: CRM → Configuration → Étapes**

### Étapes du pipeline
```
1. NOUVEAU (Probabilité: 10%)
   Délai max: 3 jours
   Action: Qualifier le prospect

2. QUALIFIÉ (Probabilité: 30%)
   Délai max: 7 jours
   Action: Premier contact/visite

3. PROPOSITION (Probabilité: 50%)
   Délai max: 5 jours
   Action: Envoi devis

4. NÉGOCIATION (Probabilité: 70%)
   Délai max: 7 jours
   Action: Suivi et ajustements

5. GAGNÉ (Probabilité: 100%)
   Action: Conversion en commande

6. PERDU (Probabilité: 0%)
   Action: Analyse raison perte
```

---

# 7. TABLEAUX DE BORD ET REPORTING

## 7.1 KPIs par Département

### KPIs Ventes
**Chemin: Ventes → Rapports → Tableau de bord**

```
INDICATEURS QUOTIDIENS:
  - CA du jour
  - Nombre de commandes
  - Panier moyen
  - Taux de conversion devis→commande

INDICATEURS MENSUELS:
  - CA mensuel vs objectif
  - Évolution vs mois précédent
  - Top 10 clients
  - Top 10 produits
  - CA par commercial
  - CA par zone géographique
```

### KPIs Stock
```
  - Valeur totale du stock
  - Rotation des stocks
  - Produits sous seuil minimum
  - Produits sans mouvement (>90 jours)
  - Taux de rupture
```

### KPIs Achats
```
  - Volume achats mensuel
  - Délai moyen livraison fournisseurs
  - Taux de conformité réceptions
  - Top fournisseurs
```

---

## 7.2 Tableau de Bord Direction Générale

**Chemin: Tableau de bord → Configuration → Créer**

### Vue synthétique DG
```
┌─────────────────────────────────────────────────────────────┐
│                    TABLEAU DE BORD DG                        │
├─────────────────┬─────────────────┬─────────────────────────┤
│  CA MENSUEL     │  MARGE BRUTE    │  CRÉANCES CLIENTS       │
│  25.5M FCFA     │  32%            │  8.2M FCFA              │
│  ▲ +12% vs M-1  │  ▼ -2% vs M-1   │  Dont échu: 2.1M        │
├─────────────────┴─────────────────┴─────────────────────────┤
│  RENTABILITÉ PAR CANAL                                       │
│  ├── POS Mobile (Fieldsales): 45% du CA | Marge: 28%        │
│  └── Point de Vente Fixe: 55% du CA | Marge: 35%            │
├─────────────────────────────────────────────────────────────┤
│  TOP 5 PRODUITS RENTABLES    │  BOTTOM 5 PRODUITS           │
│  1. Peinture Satinée (+40%)  │  1. Diluant Standard (-5%)   │
│  2. Enduit Déco (+38%)       │  2. Pinceau Eco (-3%)        │
│  3. Vernis Bois (+35%)       │  3. Bâche Plastique (-2%)    │
└─────────────────────────────────────────────────────────────┘
```

---

## 7.3 Analyse Rentabilité par Produit

**Chemin: Comptabilité → Rapports → Analyse rentabilité**

### Configuration du rapport
```
Dimensions d'analyse:
  - Par produit individuel
  - Par catégorie de produits
  - Par famille de produits

Métriques:
  - Chiffre d'affaires HT
  - Coût des ventes (CMP)
  - Marge brute (valeur et %)
  - Marge nette (après frais)
  
Filtres:
  - Période (jour/semaine/mois/année)
  - Canal de vente (POS/Ventes directes)
  - Commercial
  - Zone géographique
```

### Formules de rentabilité
```
Marge brute = CA HT - Coût CMP
Marge brute % = (Marge brute / CA HT) × 100

Marge nette = Marge brute - Frais variables
Frais variables = Commissions + Transport + Emballage

ROI produit = Marge nette / Stock moyen immobilisé
```

---

## 7.4 Analyse par Segment Client

```
SEGMENT QUINCAILLERIES:
  - Nombre: 45 clients
  - CA: 15M FCFA/mois
  - Marge moyenne: 25%
  - Délai paiement moyen: 5 jours

SEGMENT PARTICULIERS:
  - Nombre: 120 clients
  - CA: 8M FCFA/mois
  - Marge moyenne: 35%
  - Délai paiement: Comptant

SEGMENT ENTREPRISES:
  - Nombre: 15 clients
  - CA: 5M FCFA/mois
  - Marge moyenne: 20%
  - Délai paiement: 15 jours
```

---

## 7.5 Performance par Commercial (Fieldsale)

**Chemin: Ventes → Rapports → Par commercial**

```
┌────────────────────────────────────────────────────────────┐
│  PERFORMANCE COMMERCIAUX - MOIS EN COURS                   │
├──────────────┬──────────┬──────────┬─────────┬─────────────┤
│ Commercial   │ CA Réel  │ Objectif │ Atteinte│ Commission  │
├──────────────┼──────────┼──────────┼─────────┼─────────────┤
│ Commercial 1 │ 8.5M     │ 8M       │ 106%    │ 255 000     │
│ Commercial 2 │ 6.2M     │ 7M       │ 89%     │ 186 000     │
│ Commercial 3 │ 7.8M     │ 7M       │ 111%    │ 234 000     │
├──────────────┼──────────┼──────────┼─────────┼─────────────┤
│ TOTAL        │ 22.5M    │ 22M      │ 102%    │ 675 000     │
└──────────────┴──────────┴──────────┴─────────┴─────────────┘

Détail par commercial:
  - Nombre de visites clients
  - Nombre de commandes
  - Panier moyen
  - Nouveaux clients acquis
  - Taux de recouvrement
```

---

## 7.6 Comparatif Réel vs Objectifs

```
Configuration des objectifs:
  Chemin: Ventes → Configuration → Objectifs

  Objectif mensuel par équipe:
    - Fieldsales: 15 000 000 FCFA
    - Point de vente fixe: 20 000 000 FCFA
    
  Objectif par commercial:
    - Basé sur historique + 10% croissance
    
Tableau comparatif:
  - CA Réel vs Objectif (valeur et %)
  - Marge Réelle vs Marge cible
  - Écarts analysés par:
    * Produit
    * Client
    * Zone
    * Période
```

---

# 8. HARMONISATION POS/VENTES

## 8.1 Numérotation Cohérente des Factures

**Chemin: Comptabilité → Configuration → Journaux**

### Séquence unifiée
```
Journal unique pour toutes les ventes:
  Nom: Journal des Ventes Unifié
  Code: VTE
  Type: Ventes
  
Séquence de facturation:
  Préfixe: FAC/%(year)s/
  Taille: 5 chiffres
  Incrément: 1
  
Résultat: FAC/2026/00001, FAC/2026/00002...

Configuration par POS:
  - Tous les POS utilisent le même journal de ventes
  - Synchronisation des séquences au serveur central
```

---

## 8.2 Synchronisation Stocks Temps Réel

```
Architecture:
  ┌─────────────────┐
  │ Serveur Central │
  │  Stock global   │
  └────────┬────────┘
           │
    ┌──────┴──────┬──────────────┐
    │             │              │
┌───▼───┐    ┌───▼───┐     ┌────▼────┐
│POS YDE│    │POS DLA│     │POS OUEST│
│Stock  │    │Stock  │     │Stock    │
│local  │    │local  │     │local    │
└───────┘    └───────┘     └─────────┘

Processus de synchronisation:
1. Vente POS → Décrémente stock local
2. Synchronisation → Décrémente stock central
3. Stock central = Σ stocks locaux
4. Conflits → Stock serveur prioritaire
```

---

## 8.3 Reporting Consolidé

```
Rapport consolidé quotidien:
  
  VENTES TOTALES KAPS - [DATE]
  ═══════════════════════════════════════
  
  CANAL               │ CA HT    │ TX    │ Marge
  ────────────────────┼──────────┼───────┼──────
  POS Yaoundé         │ 850 000  │ 12    │ 28%
  POS Douala          │ 620 000  │ 8     │ 26%
  POS Ouest           │ 430 000  │ 6     │ 30%
  Point Vente Mendong │ 1 200 000│ 25    │ 32%
  ────────────────────┼──────────┼───────┼──────
  TOTAL               │ 3 100 000│ 51    │ 29%
  
  Écarts détectés: [Liste si applicable]
```

---

## 8.4 Conditions Paiement Uniformes

```
Règle appliquée tous canaux:
  - Délai max crédit: 48 heures
  - Vérification automatique crédit disponible
  - Blocage si dépassement

Configuration dans POS:
  Chemin: POS → Configuration → [POS] → Paiements
  
  ☑ Appliquer limites de crédit
  ☑ Vérifier crédit en temps réel (si online)
  ☑ Vérifier crédit au cache (si offline)
  ☑ Alerter si proche limite
```

---

## 8.5 Traçabilité Client Unique

```
Base clients centralisée:
  - ID unique par client (tous canaux)
  - Historique complet fusionné
  - Cumul CA tous canaux
  - Solde crédit unifié

Vue client 360°:
  Chemin: Contacts → [Client] → Vue 360
  
  ┌─────────────────────────────────────┐
  │ CLIENT: Quincaillerie ABC           │
  ├─────────────────────────────────────┤
  │ CA Total 2026: 2 500 000 FCFA       │
  │   └── POS Mobile: 1 200 000         │
  │   └── Magasin: 1 300 000            │
  │                                     │
  │ Dernière commande: 02/01/2026       │
  │ Crédit utilisé: 150 000 / 500 000   │
  │ Factures échues: 0                  │
  │ Commercial attitré: Commercial 1    │
  └─────────────────────────────────────┘
```

---

## 8.6 Consolidation Encaissements

```
Suivi trésorerie centralisé:
  
  ENCAISSEMENTS JOUR - [DATE]
  ═══════════════════════════════════════
  
  MODE          │ POS    │ MAGASIN │ TOTAL
  ──────────────┼────────┼─────────┼───────
  Espèces       │ 800K   │ 900K    │ 1.7M
  Mobile Money  │ 350K   │ 200K    │ 550K
  Chèques       │ 100K   │ 150K    │ 250K
  Virements     │ 0      │ 300K    │ 300K
  ──────────────┼────────┼─────────┼───────
  TOTAL         │ 1.25M  │ 1.55M   │ 2.8M

Rapprochement:
  Chemin: Comptabilité → Trésorerie → Consolidation
  
  Vérifier quotidiennement:
    ☑ Espèces comptées = Espèces système
    ☑ Transactions MoMo = Relevé opérateur
    ☑ Chèques remis = Chèques enregistrés
```

---

# 9. CHECKLIST DE DÉPLOIEMENT

## Phase 1: Installation (Jour 1-2)
```
☐ Installation Odoo 18 Enterprise
☐ Configuration serveur et base de données
☐ Installation modules requis
☐ Configuration société et devises
```

## Phase 2: Configuration Base (Jour 3-5)
```
☐ Plan comptable OHADA
☐ Taxes TVA 19,25%
☐ Journaux comptables
☐ Comptes bancaires et Mobile Money
☐ Catégories de produits
☐ Unités de mesure
```

## Phase 3: Configuration Métier (Jour 6-10)
```
☐ Entrepôts et emplacements
☐ Règles de réapprovisionnement
☐ Configuration POS (3 mobiles + 1 fixe)
☐ Mode offline et synchronisation
☐ Équipes commerciales
☐ Conditions de paiement
```

## Phase 4: Données (Jour 11-15)
```
☐ Import produits
☐ Import clients
☐ Import fournisseurs
☐ Inventaire initial
☐ Soldes comptables d'ouverture
```

## Phase 5: Tests (Jour 16-20)
```
☐ Test cycle vente complet
☐ Test POS offline/online
☐ Test synchronisation stocks
☐ Test facturation et paiements
☐ Test rapports et tableaux de bord
```

## Phase 6: Formation (Jour 21-25)
```
☐ Formation commerciaux (POS mobile)
☐ Formation vendeurs (magasin)
☐ Formation comptabilité
☐ Formation direction (tableaux de bord)
```

## Phase 7: Go-Live (Jour 26-30)
```
☐ Passage en production
☐ Support intensif première semaine
☐ Ajustements et corrections
☐ Documentation utilisateur
```

---

**FIN DU GUIDE DE DÉPLOIEMENT**

*Document créé le: 03/01/2026*
*Version: 1.0*
*Auteur: Consultant Odoo*






# FICHE DE COLLECTE DES DONNÉES CLIENT

## Projet: Déploiement Odoo 18 Enterprise
**Client:** _____________________________  
**Date:** _____________________________  
**Responsable collecte:** _____________________________

---

## 1. INFORMATIONS GÉNÉRALES

### Questions préliminaires

| Question | Réponse |
|----------|---------|
| Système actuel utilisé ? | ☐ Excel ☐ Autre ERP ☐ Logiciel caisse ☐ Aucun |
| Nombre de produits actifs ? | __________ |
| Nombre de clients actifs ? | __________ |
| Nombre de fournisseurs ? | __________ |
| Date de bascule souhaitée ? | ___/___/______ |
| Créances clients en cours ? | ☐ Oui (_____ FCFA) ☐ Non |
| Dettes fournisseurs en cours ? | ☐ Oui (_____ FCFA) ☐ Non |
| Commandes en attente ? | ☐ Oui ☐ Non |

---

## 2. DONNÉES PRODUITS (OBLIGATOIRE)

### Format requis: Excel (.xlsx) ou CSV (UTF-8)

| Colonne | Description | Obligatoire | Exemple |
|---------|-------------|-------------|---------|
| `reference` | Code produit interne | ✅ Oui | PEIN-SAT-BLC-20L |
| `nom` | Nom complet du produit | ✅ Oui | Peinture Satinée Blanche 20L |
| `categorie` | Catégorie/Famille | ✅ Oui | Peintures |
| `prix_vente` | Prix de vente HT (FCFA) | ✅ Oui | 25000 |
| `cout` | Prix d'achat/Coût (FCFA) | ✅ Oui | 18000 |
| `unite_mesure` | Unité de vente | ✅ Oui | Seau 20L |
| `code_barre` | Code-barres EAN | ☐ Optionnel | 6171234567890 |
| `stock_actuel` | Quantité en stock | ✅ Oui | 45 |
| `variante` | Attributs (taille, couleur) | ☐ Si applicable | Couleur: Blanc |

### Checklist produits
- [ ] Fichier Excel fourni
- [ ] Toutes les colonnes obligatoires remplies
- [ ] Prix en FCFA (sans espaces ni symboles)
- [ ] Catégories cohérentes
- [ ] Pas de doublons de référence

---

## 3. DONNÉES CLIENTS (OBLIGATOIRE)

### Format requis: Excel (.xlsx) ou CSV (UTF-8)

| Colonne | Description | Obligatoire | Exemple |
|---------|-------------|-------------|---------|
| `nom` | Nom ou Raison sociale | ✅ Oui | Quincaillerie ABC |
| `type` | Type de client | ✅ Oui | Quincaillerie / Particulier / Entreprise |
| `adresse` | Adresse complète | ✅ Oui | BP 1234, Yaoundé |
| `ville` | Ville | ✅ Oui | Yaoundé |
| `region` | Région/Zone | ☐ Optionnel | Centre |
| `telephone` | Téléphone principal | ✅ Oui | 699123456 |
| `telephone2` | Téléphone secondaire | ☐ Optionnel | 677654321 |
| `email` | Adresse email | ☐ Optionnel | contact@abc.cm |
| `niu` | Numéro Identifiant Unique | ☐ Si entreprise | M012345678901A |
| `rccm` | Registre de commerce | ☐ Si entreprise | RC/YAO/2020/B/1234 |
| `limite_credit` | Plafond crédit (FCFA) | ✅ Oui | 500000 |
| `solde_actuel` | Créance actuelle (FCFA) | ✅ Oui | 150000 |
| `commercial` | Commercial attitré | ☐ Optionnel | Jean Dupont |

### Checklist clients
- [ ] Fichier Excel fourni
- [ ] Noms complets (pas d'abréviations ambiguës)
- [ ] Téléphones au format local (6XXXXXXXX)
- [ ] Soldes vérifiés et à jour
- [ ] Types clients cohérents

---

## 4. DONNÉES FOURNISSEURS

### Format requis: Excel (.xlsx) ou CSV (UTF-8)

| Colonne | Description | Obligatoire | Exemple |
|---------|-------------|-------------|---------|
| `nom` | Raison sociale | ✅ Oui | Peintures Cameroun SA |
| `pays` | Pays | ✅ Oui | Cameroun |
| `adresse` | Adresse | ✅ Oui | Zone Industrielle Douala |
| `telephone` | Téléphone | ✅ Oui | 233445566 |
| `email` | Email contact | ☐ Optionnel | commandes@peintures.cm |
| `niu` | NIU fiscal | ☐ Si local | M098765432101B |
| `devise` | Devise transactions | ✅ Oui | XAF / EUR / USD |
| `delai_paiement` | Conditions paiement | ✅ Oui | 30 jours |
| `produits` | Produits fournis | ☐ Optionnel | Peintures, Vernis |

### Checklist fournisseurs
- [ ] Fichier Excel fourni
- [ ] Devises correctement indiquées
- [ ] Fournisseurs actifs uniquement

---

## 5. INVENTAIRE / STOCK INITIAL

### Format requis: Excel (.xlsx) ou CSV (UTF-8)

| Colonne | Description | Obligatoire | Exemple |
|---------|-------------|-------------|---------|
| `reference` | Code produit | ✅ Oui | PEIN-SAT-BLC-20L |
| `emplacement` | Entrepôt/Zone | ✅ Oui | Entrepôt Principal |
| `quantite` | Quantité comptée | ✅ Oui | 45 |
| `cout_moyen` | Coût moyen actuel | ✅ Oui | 18500 |
| `date_inventaire` | Date du comptage | ✅ Oui | 15/01/2026 |
| `lot` | Numéro de lot | ☐ Si applicable | LOT2025-001 |

### Checklist inventaire
- [ ] Inventaire physique réalisé
- [ ] Date de l'inventaire notée
- [ ] Coûts moyens calculés
- [ ] Écarts identifiés et justifiés

---

## 6. SOLDES COMPTABLES D'OUVERTURE

### Pour la bascule comptable (si applicable)

| Compte | Description | Solde Débiteur | Solde Créditeur |
|--------|-------------|----------------|-----------------|
| 311 | Marchandises | ______________ | |
| 411 | Clients | ______________ | |
| 401 | Fournisseurs | | ______________ |
| 521 | Banque | ______________ | |
| 571 | Caisse | ______________ | |
| 531 | Mobile Money | ______________ | |

### Documents requis
- [ ] Balance générale au jour J-1 de bascule
- [ ] Détail des créances clients (par facture)
- [ ] Détail des dettes fournisseurs (par facture)
- [ ] Relevés bancaires récents
- [ ] État de caisse

---

## 7. HISTORIQUE (OPTIONNEL)

### Pour analyse et reporting

| Document | Période | Format | Fourni |
|----------|---------|--------|--------|
| Factures de vente | 12 derniers mois | Excel/PDF | ☐ |
| Commandes en cours | À ce jour | Excel | ☐ |
| Avoirs en attente | À ce jour | Excel | ☐ |
| Statistiques ventes | 12 derniers mois | Excel | ☐ |

---

## 8. RÉCAPITULATIF FICHIERS À FOURNIR

| # | Fichier | Format | Statut |
|---|---------|--------|--------|
| 1 | Liste des produits | Excel (.xlsx) | ☐ Reçu |
| 2 | Liste des clients | Excel (.xlsx) | ☐ Reçu |
| 3 | Liste des fournisseurs | Excel (.xlsx) | ☐ Reçu |
| 4 | Inventaire physique | Excel (.xlsx) | ☐ Reçu |
| 5 | Balance comptable | Excel/PDF | ☐ Reçu |
| 6 | Détail créances | Excel (.xlsx) | ☐ Reçu |
| 7 | Détail dettes | Excel (.xlsx) | ☐ Reçu |

---

## 9. NOTES ET OBSERVATIONS

```
_____________________________________________________________________________

_____________________________________________________________________________

_____________________________________________________________________________

_____________________________________________________________________________

_____________________________________________________________________________
```

---

## 10. VALIDATION

| Rôle | Nom | Signature | Date |
|------|-----|-----------|------|
| Client | | | |
| Consultant | | | |

---

**IMPORTANT:** 
- Tous les fichiers doivent être au format **Excel (.xlsx)** ou **CSV encodé UTF-8**
- Les montants doivent être en **FCFA** sans espaces ni symboles monétaires
- Les dates au format **JJ/MM/AAAA**
- Éviter les cellules fusionnées dans Excel

---

*Document généré le: 08/01/2026*  
*Version: 1.0*

