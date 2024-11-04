# Lab 2 : Advanced KQL - Challenge 1

##The rarest book is missing!

This was supposed to be a great day for **Digitown’s National Library Museum** and all of Digitown.

The museum has just finished scanning over **325,000 rare books** so that history lovers worldwide can experience the ancient culture and knowledge of the **Digitown Explorers**. The grand book exhibition was about to re-open when the museum director noticed something alarming: the **rarest book in the world** was missing!

The missing artifact is titled *"De Revolutionibus Magnis Data"*, published in **1613** by **Gustav Kustov**.

The mayor of Digitown, Mrs. **Gaia Budskott**, has personally requested our agency to help locate this precious book.

### Key Details About the Digitown Library System

Luckily, everything is digital in the Digitown library:

- Each book has its parameters recorded, such as **number of pages** and **weight**.
- Each book is tagged with an **RFID sticker** (RFID: radio transmitter with a unique ID).
- Each shelf in the museum is equipped to:
  - Track the **RFIDs of the books** placed on it.
  - Measure the **total weight** of books on the shelf.

### The Challenge

Unfortunately, the RFID tag for *"De Revolutionibus Magnis Data"* was found detached and abandoned on the museum floor. Could you help locate the book on one of the museum shelves and save the day?

---

Add the following datas to your cluster

```kql
.execute database script <|
// Create table for the books
.create-merge table Books(rf_id:string, book_title:string, publish_date:long, author:string, language:string, number_of_pages:long, weight_gram:long)
//clear any previously ingested data if such exists
.clear table Books data
// Import data for books
// (Used data is utilizing catalogue from https://github.com/internetarchive/openlibrary )
.ingest into table Books ('https://kustodetectiveagency.blob.core.windows.net/digitown-books/books.csv.gz') with (ignoreFirstRecord=true)
// Create table for the shelves
.create-merge table Shelves (shelf:long, rf_ids:dynamic, total_weight:long) 
.clear table Shelves data
// Import data for shelves
.ingest into table Shelves ('https://kustodetectiveagency.blob.core.windows.net/digitown-books/shelves.csv.gz') with (ignoreFirstRecord=true)
```
![kql5](\assets\kql5.png){ align=center }

## Which shelf is the book on?