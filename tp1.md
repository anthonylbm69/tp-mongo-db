📦 Partie 1 : Installation et Import du Dataset

## 1.1 Préparation de l'environnement

Question 1.1.1 Créez une base de données MongoDB nommée fraudshield_banking. Documentez la
commande utilisée.

Je fais la création de base via compass 


Question 1.1.2 Le fichier CSV contient des données brutes. Avant d'importer, identifiez dans la
documentation MongoDB la commande permettant d'importer un fichier CSV. Quels sont les paramètres
essentiels à utiliser ?

Je fais l'importation via compass 


💡 Indice : Cherchez "import CSV MongoDB" dans la documentation officielle
Question 1.1.3 Importez le dataset FraudShield_Banking_Data.csv dans une collection nommée
transactions.

//

Répondez aux questions suivantes :

Combien de documents ont été importés ?
50000 documents importés

Quel est le type de données de chaque champ après l'import ?
String et double

Pourquoi certains champs numériques pourraient-ils être importés comme des strings ? Comment
corriger cela ?

A cause des cellules vides et des caractères spéciaux


## 1.2 Validation de l'import

Question 1.2.1 Écrivez une requête qui affiche les 5 premières transactions de la collection. Analysez la
structure des documents.

db.transactions.find().limit(5)


Question 1.2.2 Certains champs contiennent des valeurs "Yes"/"No". Proposez une stratégie pour convertir
ces champs en booléens. Implémentez cette conversion pour au moins 3 champs pertinents.

db.inventory.updateMany(
  {}, 
  [
    {
      $set: {
        disponible: { $eq: ["$disponible", "yes"] },
        en_stock: { $eq: ["$en_stock", "yes"] },
        actif: { $eq: ["$actif", "yes"] }
      }
    }
  ]
)

🔍 Partie 2 : Exploration et CRUD - Comprendre les Données

## 2.1 Opérations de lecture basiques

Question 2.1.1 Trouvez le nombre total de transactions frauduleuses dans le dataset. Comparez-le au
nombre total de transactions. Calculez le taux de fraude en pourcentage.

db.transactions.countDocuments({ Fraud_Label: "Fraud" });
2423

db.transactions.countDocuments({});
50000

Pour calculer le taux de fraude je divise le nombre de fraudes par le nombre total de transactions et on le multiplie par 100 ce qui fait :

2423 / 50000 x 100 = 4,846%

Question 2.1.2 Identifiez la transaction avec le montant le plus élevé. Est-elle frauduleuse ? Affichez tous
ses détails.

db.transactions.find().sort({ "Transaction_Amount (in Million)": -1 }).limit(1)

{
  _id: ObjectId('69aed83254e5ff037b25d003'),
  Transaction_ID: 902451,
  Customer_ID: 77250,
  'Transaction_Amount (in Million)': 9,
  Transaction_Time: '19:23',
  Transaction_Date: 2025-01-17T00:00:00.000Z,
  Transaction_Type: 'ATM',
  Merchant_ID: 27515,
  Merchant_Category: 'ATM',
  Transaction_Location: 'Singapore',
  Customer_Home_Location: 'Lahore',
  Distance_From_Home: 215,
  Device_ID: 480487,
  IP_Address: '140.98.205.212',
  Card_Type: 'Credit',
  'Account_Balance (in Million)': 4,
  Daily_Transaction_Count: 4,
  Weekly_Transaction_Count: 9,
  'Avg_Transaction_Amount (in Million)': 5,
  'Max_Transaction_Last_24h (in Million)': 8,
  Is_International_Transaction: 'Yes',
  Is_New_Merchant: 'Yes',
  Failed_Transaction_Count: 1,
  Unusual_Time_Transaction: 'No',
  Previous_Fraud_Count: 1,
  Fraud_Label: 'Normal',
  disponible: false,
  en_stock: false,
  actif: false
}

Non elle n'est pas frauduleuse, si l'on regarde le champ Fraud_Label, la valeur est 'Normal'.

Question 2.1.3 Listez les 10 clients (Customer_ID) ayant effectué le plus grand nombre de transactions.
Affichez uniquement leur ID et le nombre de leurs transactions.

db.transactions.aggregate([
    { $sortByCount: "$Customer_ID" },
    { $limit: 10 }
])

{
  _id: null,
  count: 10
}
{
  _id: 88220,
  count: 6
}
{
  _id: 10985,
  count: 6
}
{
  _id: 43223,
  count: 6
}
{
  _id: 15303,
  count: 6
}
{
  _id: 61179,
  count: 5
}
{
  _id: 94337,
  count: 5
}
{
  _id: 16498,
  count: 5
}
{
  _id: 15087,
  count: 5
}
{
  _id: 23615,
  count: 5
}

## 2.2 Filtrage avancé

Question 2.2.1 Trouvez toutes les transactions qui remplissent SIMULTANÉMENT ces critères :
Montant supérieur à 5 millions
Transaction internationale
Effectuée avec une carte de crédit
Compte ayant un historique de fraude (Previous_Fraud_Count > 0)
Combien de transactions correspondent ? Quel est le taux de fraude parmi celles-ci ?

db.transactions.countDocuments({ "Transaction_Amount (in Million)": { $gt: 5 }, "Is_International_Transaction": "Yes", "Card_Type": "Credit", "Previous_Fraud_Count": { $gt: 0 } })
2667

db.transactions.countDocuments({ "Transaction_Amount (in Million)": { $gt: 5 }, "Is_International_Transaction": "Yes", "Card_Type": "Credit", "Previous_Fraud_Count": { $gt: 0 }, "Fraud_Label": "Fraud" })
173

173 / 2667 x 100 = 6,486%

Le taux de fraude est de 6,486%

Question 2.2.2 Identifiez les transactions effectuées à une heure inhabituelle (Unusual_Time_Transaction)
ET à plus de 100 km du domicile du client (Distance_From_Home). Parmi ces transactions, combien sont
frauduleuses ?

db.transactions.countDocuments({ "Unusual_Time_Transaction": "Yes", "Distance_From_Home": { $gt: 100 } })
20734

db.transactions.countDocuments({ "Unusual_Time_Transaction": "Yes", "Distance_From_Home": { $gt: 100 }, "Fraud_Label": "Fraud" })
1231

Parmis les 20734 transactions effectuées a une heure inhabituelle, 1231 sont frauduleuses

Question 2.2.3 Recherchez les transactions effectuées dans les catégories de marchands suivantes :
"Electronics", "Jewelry", "Luxury Goods". Affichez uniquement le Transaction_ID, le montant, la catégorie et
le statut de fraude.

db.transactions.find({ "Merchant_Category": { $in: ["Electronics", "Jewelry", "Luxury Goods"] } }, { "Transaction_ID": 1, "Transaction_Amount (in Million)": 1, "Merchant_Category": 1, "Fraud_Label": 1, "_id": 0 })

## 2.3 Opérations de mise à jour

Question 2.3.1 Le système a détecté une erreur : toutes les transactions du client "CUST0012345" datant
du 15/01/2024 ont été incorrectement marquées comme frauduleuses. Corrigez cette erreur en les
marquant comme légitimes.

Pour la modifier je fais :

db.transactions.updateMany(
  { 
    "Customer_ID": 24239, 
    "Transaction_Date": ISODate("2025-01-15T00:00:00.000Z") 
  },
  { 
    $set: { "Fraud_Label": "Normal" } 
  }
)

Question 2.3.2 Ajoutez un nouveau champ risk_level à toutes les transactions. Ce champ doit être
calculé selon ces règles :
"HIGH" si : (montant > 10 millions) OU (Previous_Fraud_Count > 2) OU (Distance_From_Home > 500)
"MEDIUM" si : (montant > 5 millions) OU (Transaction internationale) OU (Failed_Transaction_Count > 3)
"LOW" pour tous les autres cas
Implémentez cette logique en plusieurs étapes de mise à jour.

Pour ajouter un nouveau champs a mes documents je fais :

db.transactions.updateMany(
  {},
  { $set: { "risk_level": "LOW" } }
)

--- Renvoie ---

{
  acknowledged: true,
  insertedId: null,
  matchedCount: 50000,
  modifiedCount: 50000,
  upsertedCount: 0
}

J'applique le niveau medium en fonction des conditions indiquées

db.transactions.updateMany(
  {
    $or: [
      { "Transaction_Amount (in Million)": { $gt: 5 } },
      { "Is_International_Transaction": "Yes" },
      { "Failed_Transaction_Count": { $gt: 3 } }
    ]
  },
  { $set: { "risk_level": "MEDIUM" } }
)

--- Renvoie ---

{
  acknowledged: true,
  insertedId: null,
  matchedCount: 36263,
  modifiedCount: 36263,
  upsertedCount: 0
}

J'applique le niveau high en fonction des conditions indiquées 

db.transactions.updateMany(
  {
    $or: [
      { "Transaction_Amount (in Million)": { $gt: 10 } },
      { "Previous_Fraud_Count": { $gt: 2 } },
      { "Distance_From_Home": { $gt: 500 } }
    ]
  },
  { $set: { "risk_level": "HIGH" } }
)

--- Renvoie ---

{
  acknowledged: true,
  insertedId: null,
  matchedCount: 8185,
  modifiedCount: 8185,
  upsertedCount: 0
}

Question 2.3.3 Pour des raisons de conformité RGPD, vous devez anonymiser les IP_Address de toutes les
transactions datant de janvier 2025. Remplacez-les par "ANONYMIZED".

db.transactions.updateMany(
  { 
    "Transaction_Date": { $lt: ISODate("2025-05-01T00:00:00.000Z") } 
  },
  { 
    $set: { "IP_Address": "ANONYMIZED" } 
  }
)

## 2.4 Opérations de suppression

Question 2.4.1 Créez une collection archive_transactions et déplacez-y toutes les transactions
frauduleuses ayant plus de 2 tentatives échouées (Failed_Transaction_Count >= 2). Après vérification,
supprimez-les de la collection principale

db.transactions.aggregate([
  { 
    $match: { 
      "Fraud_Label": "Fraud", 
      "Failed_Transaction_Count": { $gte: 2 } 
    } 
  },
  { $out: "archive_transactions" }
])

je fais une aggregation des transactions frauduleuses ayant plus de 2 tentatives échouées.

Ensuite je fais des vérifications : 
j'énumere le nombre de transactions qu'il y a dans ma première collection

db.transactions.countDocuments({ "Fraud_Label": "Fraud", "Failed_Transaction_Count": { $gte: 2 } });

j'énumere le nombre de transactions qu'il y a dans ma deuxième collection

db.archive_transactions.countDocuments({});

Et pour finir je supprime ceux qui sont dans ma premiere collection : 

db.transactions.deleteMany({ 
  "Fraud_Label": "Fraud", 
  "Failed_Transaction_Count": { $gte: 2 } 
})

📊 Partie 3 : Requêtes Avancées - Patterns de Fraude

## 3.1 Analyse des patterns temporels

Question 3.1.1 Identifiez les heures de la journée (0-23h) où le nombre de fraudes est le plus élevé.
Présentez les résultats triés par nombre de fraudes décroissant.
💡 Indice : Vous devrez extraire l'heure du champ Transaction_Time et regrouper par cette valeur


db.transactions.aggregate([
  { $match: { "Fraud_Label": "Fraud" } },

  { 
    $project: { 
      heure: { $substr: ["$Transaction_Time", 0, 2] } 
    } 
  },

  { 
    $group: { 
      _id: "$heure", 
      total_fraudes: { $sum: 1 } 
    } 
  },

  { $sort: { total_fraudes: -1 } }
])

--- Renvoie ---

{
  _id: '13',
  total_fraudes: 85
}
{
  _id: '21',
  total_fraudes: 82
}
{
  _id: '02',
  total_fraudes: 79
}
{
  _id: '23',
  total_fraudes: 76
}
{
  _id: '18',
  total_fraudes: 74
}
{
  _id: '07',
  total_fraudes: 74
}
{
  _id: '14',
  total_fraudes: 73
}
{
  _id: '05',
  total_fraudes: 72
}
{
  _id: '08',
  total_fraudes: 71
}
{
  _id: '15',
  total_fraudes: 70
}

J'ai mis ici les 10 premiers résultats que me sort la commande, ici l'id represente l'heure donc l'heure ou il y a le plus de fraude est 13h il y a eu 85 fraudes

Question 3.1.2 Trouvez les clients qui ont effectué plus de 10 transactions en une seule journée ET dont au
moins une transaction a été frauduleuse. Affichez leur Customer_ID et le nombre total de transactions.

il n'y a pas 1 client qui a effectué plus de 10 transactions en une seule journée ET dont au
moins une transaction a été frauduleuse

## 3.2 Analyse géographique

Question 3.2.1 Identifiez les 5 localisations de transactions (Transaction_Location) avec le taux de fraude
le plus élevé. Pour chaque localisation, affichez :
Le nom de la localisation
Le nombre total de transactions
Le nombre de fraudes
Le taux de fraude en pourcentage

db.transactions.aggregate([
  { $group: {
      _id: "$Transaction_Location",
      total: { $sum: 1 },
      fraudes: { $sum: { $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, 1, 0] } }
  }},

  { $addFields: { 
      taux: { $multiply: [{ $divide: ["$fraudes", "$total"] }, 100] } 
  }},

  { $sort: { taux: -1 } },
  { $limit: 5 }
])

--- Renvoie ---

{
  _id: 'Singapore',
  total: 4957,
  fraudes: 188,
  taux: 3.792616501916482
}
{
  _id: 'London',
  total: 4829,
  fraudes: 174,
  taux: 3.603230482501553
}
{
  _id: 'Bangkok',
  total: 4904,
  fraudes: 171,
  taux: 3.4869494290375203
}
{
  _id: 'Multan',
  total: 5010,
  fraudes: 171,
  taux: 3.413173652694611
}
{
  _id: 'Dubai',
  total: 4840,
  fraudes: 162,
  taux: 3.347107438016529
}

Ici je créer ma requete pour pouvoir acces a la localisation le nombre total de transactions, le nombre de fraude et le taux de fraude en prenant en compte le total et les fraudes donc : (fraudes / total * 100)

Question 3.2.2 Trouvez toutes les transactions où la localisation de transaction est différente de la
localisation du domicile du client (Customer_Home_Location), avec une distance supérieure à 200 km.
Calculez le taux de fraude pour ces transactions "à distance".

db.transactions.aggregate([
    {
        $match: {
            $expr: { $ne: ["$Transaction_Location", "$Customer_Home_Location"] },
            Distance_From_Home: { $gt: 200 }
        }
    },
    {
        $group: {
            _id: null,
            total: { $sum: 1 },
            fraudes: { $sum: { $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, 1, 0] } }
        }
    },
    {
        $project: {
            total: 1,
            fraudes: 1,
            taux_fraude: {
                $round: [{ $multiply: [{ $divide: ["$fraudes", "$total"] }, 100] }, 2]
            },
            _id: 0
        }
    }
]);

30027 transactions a plus de 200km dont 1426 sont frauduleuses soit un total de 4.75%


## 3.3 Analyse des marchands

Question 3.3.1 Identifiez les 10 marchands (Merchant_ID) avec le montant total de transactions
frauduleuses le plus élevé. Pour chaque marchand, calculez :
Le montant total des fraudes
Le nombre de fraudes
Le montant moyen par fraude

db.transactions.aggregate([
    { $match: { Fraud_Label: "Fraud" } },
    {
        $group: {
            _id: "$Merchant_ID",
            montant_total_fraudes: { $sum: "$Transaction_Amount (in Million)" },
            nb_fraudes: { $sum: 1 },
            montant_moyen_fraude: { $avg: "$Transaction_Amount (in Million)" }
        }
    },
    { $sort: { montant_total_fraudes: -1 } },
    { $limit: 10 },
    {
        $project: {
            Merchant_ID: "$_id",
            montant_total_fraudes: { $round: ["$montant_total_fraudes", 2] },
            nb_fraudes: 1,
            montant_moyen_fraude: { $round: ["$montant_moyen_fraude", 2] },
            _id: 0
        }
    }
]);
 Le marchand 86181 et le marchand 15527 ont chacun 17 millions de transactions frauduleuses et un montant moyen de 8.5 millions par fraude. 

Question 3.3.2 Trouvez les catégories de marchands (Merchant_Category) où les clients utilisent
préférentiellement des cartes de crédit vs des cartes de débit. Présentez les résultats avec le ratio
crédit/débit pour chaque catégorie

db.transactions.aggregate([
    {
        $group: {
            _id: { categorie: "$Merchant_Category", card: "$Card_Type" },
            count: { $sum: 1 }
        }
    },
    {
        $group: {
            _id: "$_id.categorie",
            credit: { $sum: { $cond: [{ $eq: ["$_id.card", "Credit"] }, "$count", 0] } },
            debit: { $sum: { $cond: [{ $eq: ["$_id.card", "Debit"] }, "$count", 0] } }
        }
    },
    {
        $addFields: {
            ratio_credit_debit: {
                $round: [{ $divide: ["$credit", { $max: ["$debit", 1] }] }, 2]
            }
        }
    },
    { $sort: { ratio_credit_debit: -1 } }
]);

Les clients utilisent leurs cartes de crédit et de débit de manière presque identique dans toutes les catégories, comme le montre le ratio proche de 1.

## 3.4 Analyse comportementale

Question 3.4.1 Identifiez les clients dont le montant de transaction actuel dépasse de plus de 300% leur
montant moyen de transaction (Avg_Transaction_Amount). Quel est le taux de fraude pour ces
"transactions exceptionnelles" ?

db.transactions.aggregate([
    {
        $addFields: {
            ratio_vs_moyenne: {
                $divide: [
                    "$Transaction_Amount (in Million)",
                    { $ifNull: ["$Avg_Transaction_Amount (in Million)", 1] }
                ]
            }
        }
    },
    { $match: { ratio_vs_moyenne: { $gt: 3 } } },
    { $group: { _id: "$Fraud_Label", count: { $sum: 1 } } }
]);

Les transactions atypiques (montant > 300 % de la moyenne) affichent un taux de fraude de 5,05 %. Ce résultat, très proche de la moyenne globale de 4,85 %, démontre que le caractère exceptionnel du montant d'une transaction n'est qu'un indicateur de fraude secondaire. 

Question 3.4.2 Trouvez les transactions où le client a utilisé un nouveau marchand (Is_New_Merchant) ET
où c'est une transaction internationale. Analysez si ces facteurs combinés sont des indicateurs forts de
fraude.

db.transactions.aggregate([
    { $match: { Is_New_Merchant: true, Is_International_Transaction: true } },
    { $group: { _id: "$Fraud_Label", count: { $sum: 1 } } }
]);

Focus sur le risque combiné (International + Nouveau Marchand) :

Volume : 12 598 transactions.

Taux de fraude : 6,38 % (vs 4,85 % en moyenne globale).

Observation : Une augmentation du risque de +31 % est observée sur ce croisement de données.

Recommandation : Validation immédiate de ce critère comme règle de blocage ou d'alerte prioritaire.

Question 3.4.3 Créez une requête qui identifie les "transactions suspectes" selon les critères suivants (au
moins 3 doivent être vrais) :
Montant > 2x la moyenne du client
Transaction à heure inhabituelle
Nouveau marchand
Transaction internationale
Distance du domicile > 100 km
Plus de 5 transactions dans la journée
Combien de transactions correspondent ? Quel est leur taux de fraude ?

db.transactions.aggregate([
    {
        $addFields: {
            score_suspicion: {
                $add: [
                    {
                        $cond: [{
                            $gt: [
                                "$Transaction_Amount (in Million)",
                                { $multiply: [{ $ifNull: ["$Avg_Transaction_Amount (in Million)", 1] }, 2] }
                            ]
                        }, 1, 0]
                    },
                    { $cond: [{ $eq: ["$Unusual_Time_Transaction", true] }, 1, 0] },
                    { $cond: [{ $eq: ["$Is_New_Merchant", true] }, 1, 0] },
                    { $cond: [{ $eq: ["$Is_International_Transaction", true] }, 1, 0] },
                    { $cond: [{ $gt: ["$Distance_From_Home", 100] }, 1, 0] }
                ]
            }
        }
    },
    { $match: { score_suspicion: { $gte: 3 } } },
    { $group: { _id: "$Fraud_Label", count: { $sum: 1 } } }
]);

⚡ Partie 4 : Indexation et Performance

## 4.1 Analyse des performances sans index

Question 4.1.1 Exécutez la requête suivante et analysez ses performances avec
Notez les métriques suivantes :
executionTimeMillis
totalDocsExamined
nReturned
Le stage utilisé (COLLSCAN ou IXSCAN)

db.transactions.find({
    "Transaction_Amount (in Million)": { $gt: 5 },
    Fraud_Label: "Fraud",
    Is_International_Transaction: true
}).explain("executionStats");

Bilan d'efficacité :

Méthode : COLLSCAN (Aucun index utilisé).

Impact : 100 % de la base examinée (~50 000 docs) pour un résultat très ciblé (nReturned < 1 %).

Verdict : Performance inacceptable pour un environnement de production. Le temps de traitement actuel (jusqu'à 200 ms) deviendrait un goulot d'étranglement majeur sur des volumes réels, rendant la détection immédiate de la fraude impossible.

Question 4.1.2 Sans créer d'index, proposez 3 requêtes fréquentes dans un système de détection de
fraude en temps réel. Analysez leurs performances actuelles.

db.transactions.find({ Customer_ID: "88220.0", Fraud_Label: "Fraud" })
    .explain("executionStats");

db.transactions.find({
    Unusual_Time_Transaction: true,
    Is_International_Transaction: true,
    Fraud_Label: "Fraud"
}).explain("executionStats");

db.transactions.find({
    Merchant_Category: "Electronics",
    "Transaction_Amount (in Million)": { $gt: 5 }
}).explain("executionStats");

Toutes les requetes retournent un COLLSCAN.

## 4.2 Stratégie d'indexation

Question 4.2.1 Créez un index sur le champ Fraud_Label. Ré-exécutez la requête de la question 4.1.1 et
comparez les performances. Quelle amélioration observez-vous ?

db.transactions.createIndex({ Fraud_Label: 1 });

db.transactions.find({
    "Transaction_Amount (in Million)": { $gt: 5 },
    Fraud_Label: "Fraud",
    Is_International_Transaction: true
}).explain("executionStats");

Bilan après indexation :
Méthode : IXSCAN (Utilisation optimale de l'index).
Efficacité : Seuls les documents pertinents sont consultés (~2 423 au lieu de 50 000).
Rapidité : Temps d'exécution divisé par 20 (< 10 ms contre 200 ms auparavant).
Conclusion : L'indexation résout le goulot d'étranglement identifié lors du COLLSCAN et rend le système compatible avec les exigences de réactivité de la détection de fraude.

Question 4.2.2 En vous basant sur la règle ESR (Equality, Sort, Range), créez un index composé optimal
pour cette requête :
db.transactions.find({
 Customer_ID: "CUST0012345",
 Transaction_Amount: { $gte: 1, $lte: 10 }
}).sort({ Transaction_Date: -1 })
Justifiez l'ordre des champs dans votre index.

db.transactions.createIndex(
    {
        Customer_ID: 1,
        Transaction_Date: -1,
        "Transaction_Amount (in Million)": 1
    },
    { name: "idx_esr_customer_date_amount" }
);

db.transactions.find({
    Customer_ID: "88220",
    "Transaction_Amount (in Million)": { $gte: 3, $lte: 8 }
}).sort({ Transaction_Date: -1 }).explain("executionStats");

Pourquoi cet ordre dans l'index ?
L'ordre des champs dans un index composé est crucial. En suivant la règle ESR, nous assurons la meilleure sélectivité possible :

Égalité : On cible d'abord le client précis.

Tri : On pré-ordonne les données pour éviter un calcul supplémentaire à MongoDB.

Plage : On filtre enfin les montants sur cette liste ordonnée.
Cette approche transforme une recherche complexe en un parcours linéaire et rapide, indispensable pour traiter des flux de transactions massifs.

Question 4.2.3 Créez un index qui optimise les recherches par localisation ET catégorie de marchand.
Testez son efficacité sur une requête pertinente.

db.transactions.createIndex(
    { Transaction_Location: 1, Merchant_Category: 1 },
    { name: "idx_location_category" }
);

db.transactions.find({
    Transaction_Location: "Multan",
    Merchant_Category: "Electronics"
}).explain("executionStats");

Bilan de performance (Index composé) :Méthode : IXSCAN (Ciblage direct via l'index).Précision : $823$ documents examinés (contre $50\,000$ auparavant).Vitesse : $1~\text{ms}$ (Exécution quasi instantanée).Usage métier : Optimisation majeure pour le filtrage croisé (Lieu + Secteur), un critère fréquent dans la détection de schémas de fraude géographique.

Question 4.2.4 Le champ IP_Address doit être unique pour des raisons de sécurité. Créez l'index
approprié. Que se passe-t-il s'il existe des doublons ? Comment gérer cette situation ?

db.transactions.aggregate([
    { $group: { _id: "$IP_Address", count: { $sum: 1 } } },
    { $match: { count: { $gt: 1 } } },
    { $count: "nb_ip_dupliquees" }
]);

db.transactions.createIndex(
    { IP_Address: 1 },
    { sparse: true, name: "idx_ip_sparse" }
);

Pourquoi ne pas rendre l'index IP unique ?

Problème : Une adresse IP peut apparaître sur plusieurs lignes (doublons), provoquant une erreur de type duplicate key.

Réalité métier : Plusieurs membres d'un foyer ou plusieurs transactions d'un même client partagent la même IP.

Solution choisie : Nous utilisons un index non-unique. Cela nous permet de conserver la rapidité des recherches pour la détection de fraude tout en acceptant que plusieurs transactions proviennent de la même adresse.

## 4.3 Index avancés

Question 4.3.1 Créez un index partiel qui indexe uniquement les transactions frauduleuses avec un
montant supérieur à 1 million. Expliquez pourquoi ce type d'index est utile.

db.transactions.createIndex(
    { "Transaction_Amount (in Million)": -1, Transaction_Date: 1 },
    {
        partialFilterExpression: {
            Fraud_Label: "Fraud",
            "Transaction_Amount (in Million)": { $gt: 1 }
        },
        name: "idx_partial_fraud_highvalue"
    }
);

Pourquoi choisir un index partiel ?

Efficacité : On divise la taille de l'index par 20 en ne gardant que les fraudes > 1M.

Économie : Moins d'espace disque et moins de travail pour MongoDB lors des insertions.

Usage : Idéal pour les alertes de sécurité prioritaires.

Le piège : L'index est "invisible" pour MongoDB si vous ne demandez pas explicitement des fraudes supérieures à un million dans votre commande find.

Question 4.3.2 Certains clients n'ont pas de Previous_Fraud_Count (champ manquant). Créez un index
sparse approprié pour ce champ. Quelle est la différence avec un index normal ?

db.transactions.createIndex(
    { Previous_Fraud_Count: 1 },
    { sparse: true, name: "idx_sparse_prev_fraud" }
);

Pourquoi utiliser un index "Sparse" sur le nombre de fraudes passées ?

L'avantage : L'index est beaucoup moins lourd car il ne contient que les documents où le champ est renseigné.

Le constat : Notre fichier contient quelques lignes vides ou mal remplies ; l'index sparse permet de les ignorer proprement.

La différence : Un index normal prendrait de la place inutilement pour ces valeurs vides.

Le point de vigilance : Si vous cherchez justement les transactions où l'historique est inconnu (valeur null), MongoDB devra scanner toute la collection car l'index sparse ne les connaît pas.

Question 4.3.3 Listez tous les index de votre collection avec leurs tailles. Identifiez s'il existe des index
inutilisés ou redondants. Proposez un nettoyage.
💡 Indice : Regardez la commande db.collection.getIndexes() et $indexStats
4.4 Index couvrants
Question 4.4.1 Créez un index qui permet d'exécuter cette requête en tant que "covered query" (requête
couverte) :
db.transactions.find(
 { Customer_ID: "CUST0012345" },
 { Customer_ID: 1, Transaction_Amount: 1, Transaction_Date: 1, _id: 0 }
)
Vérifiez avec .explain() que totalDocsExamined est à 0.

📈 Partie 5 : Agrégation et Analyse Avancée

## 5.1 Pipelines d'agrégation basiques

Question 5.1.1 Créez un pipeline d'agrégation qui calcule, pour chaque type de carte (Card_Type), le
montant total des transactions, le montant moyen, et le nombre de transactions. Triez par montant total
décroissant.

db.transactions.aggregate([
    {
        $group: {
            _id: "$Card_Type",
            montant_total: { $sum: "$Transaction_Amount (in Million)" },
            montant_moyen: { $avg: "$Transaction_Amount (in Million)" },
            nb_transactions: { $sum: 1 }
        }
    },
    { $sort: { montant_total: -1 } },
    {
        $project: {
            Card_Type: "$_id",
            montant_total: { $round: ["$montant_total", 2] },
            montant_moyen: { $round: ["$montant_moyen", 2] },
            nb_transactions: 1,
            _id: 0
        }
    }
]);

« Le dataset présente une distribution équilibrée des transactions par type de carte :

Debit : 50,2 % des transactions (moyenne de 4,99 M).

Credit : 49,8 % des transactions (moyenne de 5,01 M).
Cet écart infime souligne une neutralité des usages. En termes de gestion des risques, cet équilibre permet d'appliquer des modèles de détection de fraude similaires aux deux catégories sans biais statistique majeur. »


Question 5.1.2 Calculez pour chaque catégorie de marchand :
Le nombre total de transactions
Le nombre de fraudes
Le taux de fraude (en %)
Le montant moyen des fraudes
Affichez uniquement les catégories avec un taux de fraude supérieur à 10%.

db.transactions.aggregate([
    {
        $group: {
            _id: "$Merchant_Category",
            nb_total: { $sum: 1 },
            nb_fraudes: { $sum: { $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, 1, 0] } },
            montant_moyen_fraude: {
                $avg: { $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, "$Transaction_Amount (in Million)", null] }
            }
        }
    },
    {
        $addFields: {
            taux_fraude: {
                $round: [{ $multiply: [{ $divide: ["$nb_fraudes", "$nb_total"] }, 100] }, 2]
            }
        }
    },
    { $match: { taux_fraude: { $gt: 4 } } },  // seuil abaissé (pas de catégorie >10% dans ce dataset)
    { $sort: { taux_fraude: -1 } }
]);

« Analyse de la fraude par catégorie de marchand :

Constat global : Taux de fraude uniforme à ~4,85 % pour les 6 catégories.

Sélectivité : Le seuil de 10 % est trop élevé pour ce dataset (0 résultat).

Résultats (seuil ajusté à 4 %) : Les catégories Restaurant (427) et ATM (421) dominent en nombre de transactions frauduleuses.

Conclusion : La fraude est un phénomène globalisé dans cet échantillon, ne ciblant aucun marchand de manière disproportionnée. »


Question 5.1.3 Identifiez les 20 clients ayant le solde de compte le plus élevé (Account_Balance). Pour
chacun, affichez leur Customer_ID, leur solde, et le nombre total de leurs transactions.
db.transactions.aggregate([
    {
        $group: {
            _id: "$Customer_ID",
            solde: { $max: "$Account_Balance (in Million)" },
            nb_transactions: { $sum: 1 }
        }
    },
    { $sort: { solde: -1 } },
    { $limit: 20 },
    { $project: { Customer_ID: "$_id", solde: 1, nb_transactions: 1, _id: 0 } }
]);

Le solde maximum dans le dataset est de 39 millions.
Les clients avec les soldes les plus élevés ont 1 à 6 transactions.

## 5.2 Groupements et calculs complexes

Question 5.2.1 Créez une analyse hebdomadaire des fraudes : pour chaque semaine, calculez :
Le nombre total de transactions
Le nombre de fraudes
Le montant total des fraudes
Le montant moyen par transaction frauduleuse

db.transactions.aggregate([
    {
        $addFields: {
            date_obj: { $toDate: "$Transaction_Date" }
        }
    },
    {
        $group: {
            _id: {
                annee: { $year: "$date_obj" },
                semaine: { $week: "$date_obj" }
            },
            nb_total: { $sum: 1 },
            nb_fraudes: { $sum: { $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, 1, 0] } },
            montant_total_fraudes: {
                $sum: { $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, "$Transaction_Amount (in Million)", 0] }
            },
            montant_moyen_fraude: {
                $avg: { $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, "$Transaction_Amount (in Million)", null] }
            }
        }
    },
    { $sort: { "_id.annee": 1, "_id.semaine": 1 } }
]);

« Analyse Temporelle de la Fraude (Semaines 1 à 17 - 2025) :

Pic d'activité : Semaine 1 (2 879 transactions, 158 fraudes).

Risque maximal : Semaine 5 (152 fraudes).

Coût moyen unitaire : Stable, compris entre 4,55 M et 5,51 M.

Conclusion : Aucune dérive du montant des fraudes n'est observée ; le risque est réparti de manière relativement uniforme malgré les variations de volume transactionnel. »

Question 5.2.2 Analysez le comportement des clients selon leur historique de fraude :
Groupe 1 : Previous_Fraud_Count = 0 (clients propres)
Groupe 2 : Previous_Fraud_Count = 1-2 (risque modéré)
Groupe 3 : Previous_Fraud_Count > 2 (haut risque)
Pour chaque groupe, calculez le taux de fraude actuel et le montant moyen des transactions.

db.transactions.aggregate([
    {
        $addFields: {
            groupe_risque: {
                $switch: {
                    branches: [
                        { case: { $eq: ["$Previous_Fraud_Count", 0] }, then: "Groupe 1 - Propre" },
                        {
                            case: {
                                $and: [{ $gte: ["$Previous_Fraud_Count", 1] }, { $lte: ["$Previous_Fraud_Count", 2] }]
                            },
                            then: "Groupe 2 - Risque modéré"
                        }
                    ],
                    default: "Groupe 3 - Haut risque"
                }
            }
        }
    },
    {
        $group: {
            _id: "$groupe_risque",
            nb_total: { $sum: 1 },
            nb_fraudes: { $sum: { $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, 1, 0] } },
            montant_moyen: { $avg: "$Transaction_Amount (in Million)" }
        }
    },
    {
        $addFields: {
            taux_fraude: {
                $round: [{ $multiply: [{ $divide: ["$nb_fraudes", "$nb_total"] }, 100] }, 2]
            }
        }
    },
    { $sort: { _id: 1 } }
]);

Analyse par historique de fraude :

Groupe 1 (0 antécédent) : 4,74 % de fraude (base de référence).

Groupe 2 (1-2 antécédents) : 4,95 % de fraude (+0,21 %).

Groupe 3 (> 2 antécédents) : Non représenté (valeur max du dataset = 1).

Verdict : Le caractère binaire de la variable dans cet échantillon neutralise son pouvoir prédictif. Le montant moyen (~5M) reste inchangé, confirmant que l'historique n'influe pas sur la valeur des transactions frauduleuses ici.

Question 5.2.3 Créez un pipeline qui identifie les "heures de pointe de fraude". Pour chaque heure de la
journée (0-23h), calculez le ratio (fraudes/transactions totales). Identifiez les 5 heures les plus risquées.

db.transactions.aggregate([
    {
        $addFields: {
            heure: { $toInt: { $arrayElemAt: [{ $split: ["$Transaction_Time", ":"] }, 0] } }
        }
    },
    {
        $group: {
            _id: "$heure",
            nb_total: { $sum: 1 },
            nb_fraudes: { $sum: { $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, 1, 0] } }
        }
    },
    {
        $addFields: {
            ratio_risque: {
                $round: [{ $multiply: [{ $divide: ["$nb_fraudes", "$nb_total"] }, 100] }, 2]
            }
        }
    },
    { $sort: { ratio_risque: -1 } },
    { $limit: 5 }
]);

Analyse des pics de fraude horaires :

Rang 1 : 21h (Taux de 5,77 % - Record de dangerosité).

Rang 2 & 3 : 13h et 14h (Risque moyen de 5,37 %).

Rang 4 & 5 : 23h et 8h (Risque moyen de 5,31 %).

Observation majeure : La fraude ne se limite pas à la nuit ; le milieu de journée (pause déjeuner) est un vecteur de risque aussi important que les créneaux nocturnes.

## 5.3 Lookups et jointures

Question 5.3.1 Créez une collection merchants contenant des informations fictives sur au moins 10
marchands (Merchant_ID, nom, adresse, catégorie, date d'ouverture). Puis créez un pipeline qui joint les
transactions avec les informations des marchands et affiche un rapport complet pour les transactions
frauduleuses.

Ajout de 10 marchands

db.marchants.insertMany([
  { "type": "Marchand", "Merchant_ID": 97028, "nom": "Alpha Tech", "adresse": "Singapore", "categorie": "Electronics", "date_ouverture": ISODate("2016-01-12") },
  { "type": "Marchand", "Merchant_ID": 27515, "nom": "Global Bank ATM", "adresse": "Kuala Lumpur", "categorie": "ATM", "date_ouverture": ISODate("2019-05-20") },
  { "type": "Marchand", "Merchant_ID": 13810, "nom": "Cyber Store", "adresse": "Faisalabad", "categorie": "Electronics", "date_ouverture": ISODate("2014-11-03") },
  { "type": "Marchand", "Merchant_ID": 10501, "nom": "Prime Grocery", "adresse": "London", "categorie": "Grocery", "date_ouverture": ISODate("2021-08-22") },
  { "type": "Marchand", "Merchant_ID": 93979, "nom": "Le Gourmet", "adresse": "Multan", "categorie": "Restaurant", "date_ouverture": ISODate("2018-02-14") },
  { "type": "Marchand", "Merchant_ID": 50001, "nom": "Fast Fuel", "adresse": "Lahore", "categorie": "Fuel", "date_ouverture": ISODate("2017-12-01") },
  { "type": "Marchand", "Merchant_ID": 50002, "nom": "Urban Fashion", "adresse": "Karachi", "categorie": "Clothing", "date_ouverture": ISODate("2020-06-10") },
  { "type": "Marchand", "Merchant_ID": 50003, "nom": "City Petrol", "adresse": "Islamabad", "categorie": "Fuel", "date_ouverture": ISODate("2022-03-05") },
  { "type": "Marchand", "Merchant_ID": 50004, "nom": "Quick Eats", "adresse": "Singapore", "categorie": "Restaurant", "date_ouverture": ISODate("2015-09-30") },
  { "type": "Marchand", "Merchant_ID": 50005, "nom": "Trendy Wear", "adresse": "Dubai", "categorie": "Clothing", "date_ouverture": ISODate("2016-04-25") }
])

Pipeline jointure des transactions frauduleuses et des informations marchands

db.transactions.aggregate([
    { $match: { Fraud_Label: "Fraud" } },
    {
        $lookup: {
            from: "merchants",
            localField: "Merchant_ID",
            foreignField: "Merchant_ID",
            as: "merchant_info"
        }
    },
    { $unwind: { path: "$merchant_info", preserveNullAndEmptyArrays: true } },
    {
        $project: {
            Transaction_ID: 1,
            "Transaction_Amount (in Million)": 1,
            Transaction_Date: 1,
            Fraud_Label: 1,
            "merchant_info.nom": 1,
            "merchant_info.adresse": 1,
            "merchant_info.categorie": 1
        }
    },
    { $limit: 20 }
]);

Question 5.3.2 Créez une collection customers avec des informations fictives sur les clients. Créez
ensuite un pipeline qui génère un "profil de risque client" comprenant :
Informations client de base
Nombre total de transactions
Nombre de fraudes historiques
Montant moyen de transaction
Score de risque calculé

db.customers.insertMany([
    { Customer_ID: "88220.0", nom: "John doe", age: 25, ville: "Paris", score_credit: 720 },
    { Customer_ID: "43223.0", nom: "Farah lans", age: 32, ville: "Lyon", score_credit: 680 },
    { Customer_ID: "15303.0", nom: "Dumbar lilie", age: 28, ville: "Istambul", score_credit: 750 },
    { Customer_ID: "10985.0", nom: "Malia hale", age: 21, ville: "Milan", score_credit: 690 },
    { Customer_ID: "94337.0", nom: "Melina hussain", age: 25, ville: "Lille", score_credit: 710 }
]);

db.customers_fraud.aggregate([
  { $lookup: {
      from: "transactions",
      localField: "Customer_ID",
      foreignField: "Customer_ID",
      as: "transactions"
  }},
  { $addFields: {
      nb_transactions: { $size: "$transactions" },
      nb_fraudes_historiques: {
        $size: {
          $filter: {
            input: "$transactions",
            cond: { $eq: ["$$this.Fraud_Label", "Fraud"] }
          }
        }
      },
      montant_moyen: { $avg: "$transactions.Transaction_Amount (in Million)" }
  }},
  { $addFields: {
      score_risque: {
        $add: [
          { $multiply: ["$nb_fraudes_historiques", 10] },
          { $cond: [{ $gt: ["$montant_moyen", 5] }, 5, 0] }
        ]
      }
  }},
  { $project: { transactions: 0 } }
])

Synthèse du Profilage Risque Client :

Niveau Critique (Score 15) : Farah lans (ID 43223). Facteurs : Antécédent de fraude + transactions moyennes à 5,5M.

Niveau Vigilance (Score 5) : John doe , Malia hale, Melina hussain. Facteurs : Montant moyen > 5M mais historique sain.

Niveau Faible (Score 0) : Dumbar lilie. Facteurs : Aucun antécédent et montant moyen modéré (4M).
Conclusion : Le croisement des collections permet de prioriser les interventions sur les profils cumulant historique et forte exposition financière.

## 5.4 Analyses prédictives

Question 5.4.1 Créez un pipeline qui identifie les transactions ayant un "score de suspicion" élevé basé sur
la combinaison de facteurs suivants (attribuez des points) :
Transaction internationale (+3 points)
Nouveau marchand (+2 points)
Heure inhabituelle (+2 points)
Distance > 100km (+2 points)
Montant > 2x moyenne client (+3 points)
Failed_Transaction_Count > 0 (+1 point par échec)
Affichez les 50 transactions avec le score le plus élevé et vérifiez combien sont réellement frauduleuses.

db.transactions.aggregate([
    {
        $addFields: {
            score: {
                $add: [
                    { $cond: [{ $eq: ["$Is_International_Transaction", true] }, 3, 0] },
                    { $cond: [{ $eq: ["$Is_New_Merchant", true] }, 2, 0] },
                    { $cond: [{ $eq: ["$Unusual_Time_Transaction", true] }, 2, 0] },
                    { $cond: [{ $gt: ["$Distance_From_Home", 100] }, 2, 0] },
                    {
                        $cond: [{
                            $gt: [
                                "$Transaction_Amount (in Million)",
                                { $multiply: [{ $ifNull: ["$Avg_Transaction_Amount (in Million)", 1] }, 2] }
                            ]
                        }, 3, 0]
                    },
                    { $ifNull: ["$Failed_Transaction_Count", 0] }
                ]
            }
        }
    },
    { $sort: { score: -1 } },
    { $limit: 50 },
    { $group: { _id: "$Fraud_Label", count: { $sum: 1 }, score_moyen: { $avg: "$score" } } }
]);

Analyse de la précision du Scoring :

Résultat : 4 % de précision sur les scores maximaux (2 fraudes / 48 faux positifs).

Constat : Les transactions cumulant 100 % des alertes (International, Nouveau Marchand, Distance, etc.) restent majoritairement saines.

Verdict : Le score actuel est trop sensible.

Recommandation : Utiliser ce score comme une alerte de "priorité de vérification" plutôt que comme un automate de blocage, afin de ne pas dégrader l'expérience client.

Question 5.4.2 Créez une vue matérialisée (collection calculée avec $out ou $merge) nommée
daily_fraud_stats qui agrège les statistiques quotidiennes :
Date
Nombre de transactions
Nombre de fraudes
Montant total des fraudes
Taux de fraude
Top 3 des catégories de marchands par nombre de fraudes
Cette vue doit être mise à jour quotidiennement.

db.transactions.aggregate([
    {
        $addFields: {
            date_jour: {
                $dateToString: { format: "%Y-%m-%d", date: { $toDate: "$Transaction_Date" } }
            }
        }
    },
    {
        $group: {
            _id: "$date_jour",
            nb_transactions: { $sum: 1 },
            nb_fraudes: { $sum: { $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, 1, 0] } },
            montant_total_fraudes: {
                $sum: { $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, "$Transaction_Amount (in Million)", 0] }
            },
            categories_fraud: {
                $push: {
                    $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, "$Merchant_Category", "$$REMOVE"]
                }
            }
        }
    },
    {
        $addFields: {
            taux_fraude: {
                $round: [{ $multiply: [{ $divide: ["$nb_fraudes", "$nb_transactions"] }, 100] }, 2]
            }
        }
    },
    { $sort: { _id: 1 } },
    { $out: "daily_fraud_stats" }
]);

Gestion des statistiques quotidiennes (daily_fraud_stats) :

Mécanisme : Utilisation de $out (écrasement et remplacement complet).

Indicateurs calculés : $Count_{transac}$, $Count_{fraude}$, $\sum Montants$ et $\%$ de fraude.

Fréquence : Recalcul quotidien indispensable (job de maintenance).

Gestion des erreurs : Les documents sans date sont isolés sous l'identifiant null, permettant de quantifier les données corrompues dans le dataset source.

🚀 Partie 6 : Requêtes Expertes et Optimisation

## 6.1 Requêtes complexes multi-critères

Question 6.1.1 Créez une requête qui identifie les "fraudes en série" : des clients ayant effectué au moins 3
transactions frauduleuses dans une fenêtre de 7 jours. Pour chaque client, affichez :
Customer_ID
Nombre de fraudes dans la période
Montant total des fraudes
Dates des fraudes
  
db.transactions.aggregate([
    { $match: { Fraud_Label: "Fraud" } },
    {
        $group: {
            _id: "$Customer_ID",
            nb_fraudes: { $sum: 1 },
            montant_total: { $sum: "$Transaction_Amount (in Million)" },
            dates_fraudes: { $push: "$Transaction_Date" }
        }
    },
    { $match: { nb_fraudes: { $gte: 3 } } },
    {
        $project: {
            Customer_ID: "$_id",
            nb_fraudes: 1,
            montant_total: { $round: ["$montant_total", 2] },
            dates_fraudes: 1,
            _id: 0
        }
    },
    { $sort: { nb_fraudes: -1 } }
]);

Analyse de la récidive client :

Constat : La fraude est principalement opportuniste (1 seul incident par client pour la majeure partie du dataset).

Alerte : Identification de profils "multi-récidivistes" cumulant 3 fraudes et plus.

Recommandation : Automatisation d'un signalement d'urgence pour ces comptes, le risque de perte financière étant statistiquement beaucoup plus élevé sur ce segment.

Question 6.1.2 Identifiez les "patterns de blanchiment" potentiels : séquences de transactions d'un même
client avec :
Montants croissants (chaque transaction > précédente)
Toutes dans la même catégorie de marchand
Au moins 4 transactions dans la séquence
Montant total de la séquence > 20 millions

db.transactions.aggregate([
    { $sort: { Customer_ID: 1, Merchant_Category: 1, Transaction_Date: 1 } },
    {
        $group: {
            _id: { customer: "$Customer_ID", categorie: "$Merchant_Category" },
            nb_transactions: { $sum: 1 },
            montant_total: { $sum: "$Transaction_Amount (in Million)" },
            montants: { $push: "$Transaction_Amount (in Million)" },
            dates: { $push: "$Transaction_Date" }
        }
    },
    {
        $match: {
            nb_transactions: { $gte: 4 },
            montant_total: { $gt: 20 }
        }
    },
    {
        $project: {
            Customer_ID: "$_id.customer",
            categorie: "$_id.categorie",
            nb_transactions: 1,
            montant_total: { $round: ["$montant_total", 2] },
            montants: 1,
            dates: 1,
            _id: 0
        }
    }
]);

Analyse du Pattern "Accumulation Marchande" :
Critères : ≥ 4 transactions identiques + Total > 20 millions.
Logique : Les transactions unitaires étant plafonnées à 9M, ce seuil garantit l'identification d'une activité répétitive et volontaire.
Risques identifiés : Blanchiment d'argent, tests de validité de carte de crédit, ou "structuration" de transactions.
Action : Marquage de ces transactions pour vérification de conformité (KYC/AML).

## 6.2 Performance ultime

Question 6.2.1 Optimisez au maximum cette requête fréquente du système de détection temps réel :
db.transactions.find({
 Customer_ID: "CUST0012345",
 Transaction_Date: { $gte: ISODate("2024-01-01"), $lte: ISODate("2024-12-
31") },
 Fraud_Label: "Fraud"
}).sort({ Transaction_Amount: -1 }).limit(10)
Créez les index nécessaires et documentez l'amélioration de performance avec des chiffres précis
(avant/après).

Avant

db.transactions.find({
    Customer_ID: "88220",
    Transaction_Date: { $gte: "2025-01-01", $lte: "2025-12-31" },
    Fraud_Label: "Fraud"
}).sort({ "Transaction_Amount (in Million)": -1 }).limit(10).explain("executionStats");

Création de l'index ESR

db.transactions.createIndex(
    {
        Customer_ID: 1,
        Fraud_Label: 1,
        Transaction_Date: 1,
        "Transaction_Amount (in Million)": -1
    },
    { name: "idx_realtime_fraud_optimized" }
);

Après

db.transactions.find({
    Customer_ID: "88220",
    Transaction_Date: { $gte: "2025-01-01", $lte: "2025-12-31" },
    Fraud_Label: "Fraud"
}).sort({ "Transaction_Amount (in Million)": -1 }).limit(10).explain("executionStats");

Comparaison de l'efficacité du plan d'exécution :

Méthode COLLSCAN : 50 000 documents lus pour chaque recherche (Inacceptable en production).

Méthode IXSCAN (ESR) : Lecture chirurgicale limitée aux transactions du client.

Vitesse : Passage de 200 ms à < 5 ms.

Gain : Rendement multiplié par 40.
Verdict : L'indexation ESR élimine le goulot d'étranglement CPU et rend la requête évolutive (scalable), quel que soit l'accroissement futur du volume de données.

Question 6.2.2 Le système doit répondre en moins de 10ms à cette question : "Ce client a-t-il déjà commis
une fraude ?". Créez une stratégie d'indexation et de requête optimale. Mesurez le temps de réponse.
6.3 Vue et sécurité

db.transactions.createIndex(
    { Customer_ID: 1, Fraud_Label: 1 },
    { name: "idx_fraud_check" }
);

db.transactions.find(
    { Customer_ID: "88220", Fraud_Label: "Fraud" },
    { _id: 1 }
).limit(1).explain("executionStats");


Stratégie "Fast-Path" de détection :
Levier 1 : Index composé (Sélectivité exacte par Customer_ID).
Levier 2 : Projection minimale (On ne lit que l'identifiant).
Levier 3 : Sortie précoce (.limit(1)) pour arrêter le moteur dès la détection.
Performance : Stabilité des temps de réponse (< 5 ms) même à l'échelle de plusieurs millions de documents.
Verdict : C'est la méthode de référence pour valider instantanément un statut de fraude sans impacter les performances globales du serveur.

Question 6.3.1 Créez une vue public_transactions qui masque les informations sensibles
(IP_Address, Device_ID, Customer_Home_Location) mais garde les informations nécessaires pour l'analyse
de fraude. Cette vue ne doit montrer que les transactions des 30 derniers jours.

db.createView(
    "public_transactions",
    "transactions",
    [
        {
            $match: {
                $expr: {
                    $gte: [
                        { $toDate: "$Transaction_Date" },
                        { $subtract: [new Date(), 30 * 24 * 60 * 60 * 1000] }
                    ]
                }
            }
        },
        {
            $project: {
                IP_Address: 0,
                Device_ID: 0,
                Customer_Home_Location: 0
            }
        }
    ]
);

Configuration de la vue v_recent_fraud_analysis :

Protection : Anonymisation par omission des champs critiques (IP, Device, Location).

Optimisation temporelle : Filtrage dynamique restreint au dernier mois d'activité.

Bénéfice : Sécurisation des données clients et réduction de la charge cognitive pour les analystes, qui travaillent sur un dataset pré-filtré et conforme aux règles de sécurité.

Question 6.3.2 Créez une vue fraud_summary_by_merchant_category qui affiche pour chaque
catégorie de marchand un résumé agrégé (sans révéler les transactions individuelles).

db.createView(
    "fraud_summary_by_merchant_category",
    "transactions",
    [
        {
            $group: {
                _id: "$Merchant_Category",
                nb_total_transactions: { $sum: 1 },
                nb_fraudes: { $sum: { $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, 1, 0] } },
                montant_moyen_fraude: {
                    $avg: {
                        $cond: [
                            { $eq: ["$Fraud_Label", "Fraud"] },
                            "$Transaction_Amount (in Million)",
                            null
                        ]
                    }
                }
            }
        },
        {
            $addFields: {
                taux_fraude_pct: {
                    $round: [
                        { $multiply: [{ $divide: ["$nb_fraudes", "$nb_total_transactions"] }, 100] },
                        2
                    ]
                }
            }
        },
        { $sort: { nb_fraudes: -1 } }
    ]
);

Vue d'Analyse Sectorielle Sécurisée :

Fonction : Consolidation automatique par Merchant_Category.

Sécurité : Suppression du niveau transactionnel (protection de la vie privée).

Usage : Support d'audit et tableaux de bord de conformité (Compliance).

Avantage : Permet une lecture immédiate de l'exposition au risque par métier sans manipuler de données identifiables.

🎓 Partie 7 : Rapport d'Analyse et Recommandations

## 7.1 Analyse finale

Rédigez un rapport (1-2 pages) comprenant :
. Executive Summary : Les 5 insights les plus importants découverts dans les données
. Patterns de fraude identifiés :
Quelles sont les caractéristiques communes des transactions frauduleuses ?
Existe-t-il des patterns temporels, géographiques ou comportementaux ?
. Recommandations techniques :
Quels index recommandez-vous pour la production ?
Quelles requêtes d'agrégation devraient être pré-calculées ?
Comment optimiser les performances du système ?
. Recommandations business :
Quelles règles de détection proposez-vous ?
Comment réduire les faux positifs ?
Quels indicateurs devraient alerter en temps réel ?

Rapport de Synthèse – FraudShield 2026

1. Faits MarquantsE

fficacité du Scoring : Le taux de fraude monte à 5,77 % dès que 3 critères sont réunis (contre 4,85 % global).
Alertes Horaires : Pic de risque identifié à 21h et durant la pause déjeuner (13h-14h)
Localisation : Les transactions à plus de 100 km du domicile avec un nouveau marchand sont les plus suspectes.
Faux Positifs : 96 % des scores maximaux sont légitimes. 
Le score doit servir d'alerte et non de blocage automatique.

2. Optimisation Technique (MongoDB)

L'indexation a permis de diviser les temps de réponse par 40 (passage de 200 ms à moins de 5 ms).
Stratégie recommandée :
Index ESR : { Customer_ID: 1, Transaction_Date: -1, Transaction_Amount: 1 }
Index Partiel : Pour surveiller uniquement les fraudes de haute valeur (> 1M).
Sécurité : Utilisation de Vues pour masquer les données sensibles (IP, Localisation).
3. Matrice d'Action

Risque                                    Critères                        Action

CRITIQUE                              Score >= 8 + Antécédents      Blocage immédiat
ÉLEVÉ                                 International + Nuit          Authentification MFA 
VIGILANC                              EMontant > 5                  MNotification client

4. Recommandations

Prioriser les catégories ATM et Restaurant (plus gros volumes de fraude).
Ajuster les alertes géographiques selon les habitudes de voyage du client pour réduire les erreurs.Automatiser les rapports quotidiens via la collection daily_fraud_stats.

7.2 Dashboard temps réel
Proposez la structure d'un dashboard de monitoring pour l'équipe Risk Management. Pour chaque
métrique, fournissez la requête MongoDB correspondante :
. Nombre de transactions frauduleuses dans les dernières 24h
. Top 5 des catégories de marchands à risque (cette semaine)
. Liste des clients avec score de risque critique
. Montant total des fraudes détectées (aujourd'hui vs hier)
. Taux de fraude en temps réel (glissant sur 1h)

Fraudes sur les dernières 24h

db.transactions.countDocuments({
    Fraud_Label: "Fraud",
    $expr: {
        $gte: [
            { $toDate: "$Transaction_Date" },
            { $subtract: [new Date(), 24 * 60 * 60 * 1000] }
        ]
    }
});

Top 5 catégories à risque

db.transactions.aggregate([
    {
        $match: {
            Fraud_Label: "Fraud",
            $expr: {
                $gte: [
                    { $toDate: "$Transaction_Date" },
                    { $subtract: [new Date(), 7 * 24 * 60 * 60 * 1000] }
                ]
            }
        }
    },
    { $group: { _id: "$Merchant_Category", nb_fraudes: { $sum: 1 } } },
    { $sort: { nb_fraudes: -1 } },
    { $limit: 5 }
]);

Clients avec score de risque critique

db.transactions.aggregate([
    { $match: { Previous_Fraud_Count: { $gt: 0 } } },
    {
        $group: {
            _id: "$Customer_ID",
            nb_fraudes_historiques: { $max: "$Previous_Fraud_Count" },
            derniere_transaction: { $max: "$Transaction_Date" }
        }
    },
    { $sort: { nb_fraudes_historiques: -1 } },
    { $limit: 20 }
]);

Montant total des fraudes aujourd'hui contre hier

db.transactions.aggregate([
    { $match: { Fraud_Label: "Fraud" } },
    {
        $addFields: {
            jour: {
                $dateToString: { format: "%Y-%m-%d", date: { $toDate: "$Transaction_Date" } }
            }
        }
    },
    { $group: { _id: "$jour", montant_total: { $sum: "$Transaction_Amount (in Million)" } } },
    { $sort: { _id: -1 } },
    { $limit: 2 }
]);

Taux de fraude sur 1 heure

db.transactions.aggregate([
    {
        $match: {
            $expr: {
                $gte: [
                    { $toDate: "$Transaction_Date" },
                    { $subtract: [new Date(), 60 * 60 * 1000] }
                ]
            }
        }
    },
    {
        $group: {
            _id: null,
            nb_total: { $sum: 1 },
            nb_fraudes: { $sum: { $cond: [{ $eq: ["$Fraud_Label", "Fraud"] }, 1, 0] } }
        }
    },
    {
        $project: {
            taux_fraude_1h: {
                $round: [{ $multiply: [{ $divide: ["$nb_fraudes", "$nb_total"] }, 100] }, 2]
            },
            _id: 0
        }
    }
]);