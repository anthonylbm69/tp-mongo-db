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



