# Exercices 23-09-2019
## Lab 1

Ecrire les requetes qui permettent de :

- Retourner la liste de tous les **Employee**
    - `SELECT * FROM Employee`
- Retourner la liste des **Invoice** avec seulement l'ID, l'ID du client et la date
    - `SELECT invoiceid, customerid, invoicedate FROM invoice`
- Retourner le nom, prénom, et adresse de tous les **Customer**, la liste retournée doit comprendre 2 colonnes. La premiere avec Nom en majuscule, Prénom (exemple: "DUPONT, Jean"), la deuxième simplement le numéro de téléphone. Vous pouvez nommer les colonnes comme vous le souhaitez.
    - `SELECT CONCAT(UPPER(lastname), ', ', firstname) nom, address FROM customer`
- Retourner la liste des Invoice avec comme colonnes
  - ID et total au format: ID(total) 
  - La date au format: 2019.09.10 (format ANSI)
    - `SELECT CONCAT(invoiceid, ' (', total, ')'), FORMAT(invoicedate, 'yyyy.MM.dd') FROM invoice`
- Retourner la liste de TOUTES les chasons (tracks) avec comme colonnes:
  - ID
  - Nom - Compositeur (exemple: "Etnia - Chico Science") si pas de compositeur ("Now Sport / NA")
    - `SELECT trackid, CONCAT(name, ' - ', ISNULL(composer, 'N/A')) FROM track`

## Lab 2

- Retourner la liste des villes des clients en enlevant les doublons
    - `SELECT DISTINCT city FROM customer`
- Retourner les 10 plus importantes factures dans l'ordre décroissant (**Invoice**)
    - `SELECT TOP 10 * FROM invoice ORDER BY total DESC`
- Retourner toutes les informations du client dont l'ID est 47
    - `SELECT * FROM customer WHERE customerid = 47`
- Retourner la liste des clients qui vivent à Bordeaux, Chicago, Lisbon ou Berlin
    - `SELECT * FROM customer WHERE city IN ('Berlin', 'Bordeaux', 'Chicago', 'Lisbon')`
- Retourner la liste des albums dont le titre commence par la lettre "C"
    - `SELECT * FROM album WHERE title LIKE 'C%'`
- A partir de la précédente requête, changer la clause pour afficher la liste des albums dont le titre commence par la lettre "C" et dont la deuxieme lettre n'est pas un "H"
    - `SELECT * FROM album WHERE LEFT(title, 1) = 'C' AND SUBSTRING(title, 2, 1) <> 'H'`
- Retourner la liste des factures entre le 01-01-2012 et le 31-03-2013
    - `SELECT * FROM invoice WHERE InvoiceDate BETWEEN '2012-01-01' AND '2013-31-03'`

# Lab 3

- La liste des factures avec les colonnes suivantes:
  - Numéro de facture
  - Nom et prénom du client (JOINTURE)
  - Total
    - `SELECT i.invoiceid, CONCAT(UPPER(c.lastname), ', ', c.firstname), i.total FROM invoice i LEFT JOIN customer c ON i.CustomerId = c.CustomerId`
- Rajouter à la requete précédente:
  - Le nom de l'employé en charge du client
  - Le nom de l'artiste qui correspond à la chason qui a été achetée (aidez-vous du diagramme)
  
  - ` SELECT i.invoiceid, CONCAT(UPPER(c.lastname), ', ', c.firstname), i.total, e.lastname, t.name FROM invoice i
JOIN customer c ON i.CustomerId = c.CustomerId
JOIN employee e ON c.SupportRepId = e.EmployeeId
JOIN invoiceline il ON i.InvoiceId = il.InvoiceId
JOIN track t ON il.TrackId = t.TrackId`

- La liste de tous les clients avec les colonnes suivantes:
  - ID
  - Nom
  - Numéro de facture
    - `SELECT c.customerid, c.lastname, i.invoiceid FROM invoice i
JOIN customer c ON i.CustomerId = c.CustomerId`

(Si un client n'a pas de facture indiquer N/A)

- Retourner la liste des chansons qui n'ont jamais été achetées
    - `SELECT * FROM track WHERE track.trackid  NOT IN (
SELECT track.trackid FROM track
RIGHT JOIN invoiceline ON track.TrackId = invoiceline.trackid
)`

- La liste des employés avec les colonnes suivantes:
  - ID de l'employé
  - Nom
  - Email
  - Nom de son Manager
  - Email de son Manager
  - Retrouver le nom du dirigeant parmis les employés

    - `SELECT e.employeeid, e.lastname, e.email, ISNULL(s.lastname, 'Dirigeant'), ISNULL(s.Email, 'Dirigeant') FROM employee e LEFT JOIN employee s ON e.ReportsTo = s.EmployeeId`


# Lab 4

- Retourner la liste des toutes les adresses, ville et pays enregistrés dans la base (employés et clients)
    - `SELECT DISTINCT address, city, country FROM employee UNION
SELECT DISTINCT address, city, country FROM customer EXCEPT
(SELECT DISTINCT address, city, country FROM employee INTERSECT
SELECT DISTINCT address, city, country FROM customer)`
- La liste de toutes villes des tables Customer et Invoice, on veut voir s'il y a des doublons
    - `SELECT CONCAT(city, ' (Customer)'), COUNT(city) FROM customer
GROUP BY city UNION
SELECT CONCAT(billingcity, ' (Billing)'), COUNT(billingcity) FROM invoice
GROUP BY billingcity`
- La liste des adresses clients qu'on ne retrouve pas la table Invoice
    - `SELECT address FROM customer WHERE address NOT IN (SELECT billingaddress FROM invoice)`
- La liste des pays communs entre les employés et les clients
    - `SELECT country FROM employee INTERSECT
SELECT country FROM customer`

# Lab 5

- La liste des chansons avec les colonnes suivantes:
  - Id
  - Nom de l'artiste
  - Durée (en secondes, arrondi à la seconde supérieure)
    - `SELECT trackid, ISNULL(composer, 'N/A') duration, CEILING(milliseconds / 1000) FROM track`
- La liste des factures avec:
  - Id
  - date
  - Année de la facture
  - Mois de la facture (nom du mois)
  - Jour de la semaine (toujours de la facture...)
  - Les deux premiers caractères du pays
    - `SELECT i.invoiceid, CONVERT(date, i.invoicedate), YEAR(i.invoicedate), DATENAME(m, i.invoicedate), DATENAME(dw, i.invoicedate), LEFT(i.billingcountry, 2) FROM invoice i`
- Filtrer cette dernière requête pour ne faire apparaître que les factures dont le code postal est composé uniquement de chiffres
    - `SELECT i.invoiceid, CONVERT(date, i.invoicedate), YEAR(i.invoicedate), DATENAME(m, i.invoicedate), DATENAME(dw, i.invoicedate), LEFT(i.billingcountry, 2) FROM invoice i
WHERE ISNUMERIC(i.BillingPostalCode) = 1`
- Retourner le montant total des commandes passées en 2012
    - `SELECT SUM(total) FROM invoice WHERE YEAR(InvoiceDate) = 2012`
- Retourner la liste des clients avec les colonnes suivantes
  - Id du client
  - Nom, Prénom du client format (DUPONT, Jean)
  - Le nombre de commande passées
  - Le total de commandes passées
    - `SELECT c.customerid, CONCAT(UPPER(c.lastname), ', ', c.firstname), COUNT(i.invoiceid) FROM invoice i
JOIN customer c ON c.CustomerId = i.CustomerId
GROUP BY c.CustomerId, c.lastname, c.FirstName`
- A partir de la requête précédente, retourner les memes informations en deux requêtes:
  - La première qui n'affiche que les clients ayant acheté pour plus de 40 euros
  - La deuxième qui ne tient compte que des factures inférieures à 10 euros
    - `SELECT c.customerid, CONCAT(UPPER(c.lastname), ', ', c.firstname), COUNT(i.invoiceid) FROM invoice i
JOIN customer c ON c.CustomerId = i.CustomerId AND i.total > 40
GROUP BY c.CustomerId, c.lastname, c.FirstName`
    - `SELECT c.customerid, CONCAT(UPPER(c.lastname), ', ', c.firstname), COUNT(i.invoiceid) FROM invoice i
JOIN customer c ON c.CustomerId = i.CustomerId AND i.total < 10
GROUP BY c.CustomerId, c.lastname, c.FirstName`

