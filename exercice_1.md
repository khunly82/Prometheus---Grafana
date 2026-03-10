# 1. Configuration

## Préparation 

- Ajouter un serveur SQL Server

```sh
docker run -d -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Test1234=" -p 1433:1433 --name sql2022 -d mcr.microsoft.com/mssql/server:2022-latest
```
- Création de la db

```sql
CREATE DATABASE PizzaShop;
GO
USE PizzaShop;
GO

CREATE TABLE Produits (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Nom VARCHAR(50),
    Prix DECIMAL(10,2),
    Stock INT
);

INSERT INTO Produits (Nom, Prix, Stock) VALUES 
('Margherita', 9.00, 50),
('Regina', 11.50, 30),
('Calzone', 12.00, 15),
('4 Fromages', 13.00, 0);
GO
```
## Exercices

1.1. Ajouter un exporter pour les metrics (burnedikt/mssql_exporter)

1.2. Modifier prometheus pour enregistrer ces metrics

# 2. Recupération des metrics

## Préparation

- Éxécuter ces requètes sur le serveur

```sql
USE PizzaShop;
GO

SELECT Nom FROM Produits WHERE Stock > 10;
GO

INSERT INTO Produits (Nom, Prix) VALUES ('Pizza Erreur', 'PAS_UN_CHIFFRE');
GO

DECLARE @Counter INT = 0;
WHILE @Counter < 500000
BEGIN
    SET @Counter = @Counter + 1;
END
SELECT 1;
GO

SELECT 100 / (SELECT Stock FROM Produits WHERE Nom = '4 Fromages');
GO
```

## Exercices

2.1. Combien de requêtes ont échoué au total ?

2.2. Quel est le temps de réponse moyen de l'instance ?

2.3. Quel est le taux de succès (en %) ?

2.4. Est-ce que ma base de données étouffe ?

