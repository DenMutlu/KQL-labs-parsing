# Lab 2 : Advanced KQL - Challenge 1 (Solution)

### Key takeaways from the Riddle

- **Each book's weight** is known.
- **Each book**, except for the missing one, has an **RFID badge**.
- **Each shelf** provides:
  - The **actual total weight** of books on it.
  - The **RFIDs** of the books it holds.

#### Details of the missing book

- **Title**: *De Revolutionibus Magnis Data*
- **Author**: Gustav Kustov
- **Published**: 1613

- OK, first step is always to see the tables content and what data we can use,
- Lets check the Books table and the shelves (take time to check all fields at disposition)

![kql6](\assets\kql6.png){ align=center }

- when we delve into the data, to just check the fields it could take lot of time to load regarding the number of events, good practice is to use the ```kql | take 1```, to see a single row.

![kq7](\assets\kql7.png){ align=center }

**The missing book is located on a shelf, so the actual weight of the self is higher than the sum of weights that are reported on that shelf because the book is missing the RFID.**

The `Books` table has the following fields: 

- RFID, Book Title, Publish Data, Author, Language, Number of Pages, and Weight Gram.

The `Shelves` table has: id, 

- RFIDs of the books on it, and the total weight.

For each shelf, we’d like to calculate the expected total weight of the books on the shelf, so we can compare it with the reported total weight.

Altogether, we are given the RFIDs on the shelf as an array: if only we had them separately, we could have used the RFID of each book to find its weight from the books table, and then sum by shelf.

Oh now how to do this? Lucky we are with KQL, the [mv-expand](https://learn.microsoft.com/en-us/kusto/query/mv-expand-operator?view=azure-data-explorer&preserve-view=true) operator that lets us to split the RFIDs array of the shelf:

```kql
Shelves
| mv-expand rf_ids to typeof(string)
```

This results in a row for each book RFID and each shelf:

![kq8](\assets\kql8.png){ align=center }

Now we’d like to sum of weights of the books on the self, so let’s [join](https://learn.microsoft.com/en-us/kusto/query/join-operator?view=azure-data-explorer) the `Shelves` table with the `Books` table, then we can utilize the [summarize](https://learn.microsoft.com/en-us/kusto/query/summarize-operator?view=azure-data-explorer) operator to do this aggregative calculation of sum of weights of books by shelf.

We’re also interested in the reported weight of the shelf, we can [take_any](https://learn.microsoft.com/en-us/kusto/query/take-any-aggregation-function?view=azure-data-explorer) of the weights of the duplicated (for each book) shelf rows.

Lastly, we’d like to find the difference in weights of the shelves between :

The actual measured weight on the shelf
The reported (according to books with RFIDs on the shelf)
Keeping in mind the missing book has no RFID tag, so the actual weight is more than reported by the missing book weight:

Ok let's try to craft a query to do this!

### KQL Query Explanation

This KQL query aims to locate the missing book titled **"De Revolutionibus Magnis Data"** by comparing shelf weights with known book weights.

#### Step-by-Step Breakdown

**step 1 Identify the Weight of the Missing Book**:

```kql
   let x = Books
   | where book_title == "De Revolutionibus Magnis Data"
   | project weight_gram;
```

- This segment of the query retrieves the weight of the missing book, **De Revolutionibus Magnis Data**, from the `Books` dataset.
- The missing book’s weight is determined to be 1764 grams.

**step 2 Expand Shelf Data to Associate Each RFID with a Shelf**:

```kql
   let id_shelf = Shelves
| mv-expand rf_ids
| project shelf, rf_id=tostring(rf_ids);
```

- This segment uses `mv-expand` to split the list of RFIDs associated with each shelf in the `Shelves` dataset.
- It creates a table (`id_shelf`) that maps each RFID to its corresponding shelf.

**step 3 Calculate the Total Weight of Books on Each Shelf Based on Known RFIDs**:

```kql
  let existing_shelf = Books
| join kind=innerunique id_shelf on $left.rf_id==$right.rf_id
| summarize calculated_weight_gram=sum(weight_gram) by shelf;
```

- This part joins the `Books` dataset with the `id_shelf` table on RFIDs to calculate the total weight of books for each shelf based on known RFID-tagged books.
- The result, `existing_shelf`, contains each shelf with its calculated total book weight.

**step 4 Identify the Shelf with Missing Book Weight Discrepancy**:

```kql
  existing_shelf
| join kind=innerunique Shelves on $left.shelf==$right.shelf
| where total_weight - calculated_weight_gram >= 1764
```

- This final part joins the `existing_shelf` results with the original `Shelves` dataset on shelf IDs.
- It filters to find shelves where the difference between the recorded total weight (`total_weight`) and the calculated weight of known books (`calculated_weight_gram`) is greater than or equal to 1764 grams (the weight of the missing book).
 - The resulting shelf is likely the location of the missing book, as it has the appropriate weight discrepancy.

 Complete query

 ```kql
    let x = Books
    | where book_title == "De Revolutionibus Magnis Data"
    | project weight_gram;
    // weight_gram = 1764
    let id_shelf = Shelves
    | mv-expand rf_ids
    | project shelf, rf_id=tostring(rf_ids);
    let existing_shelf = Books
    | join kind=innerunique id_shelf on $left.rf_id==$right.rf_id
    | summarize calculated_weight_gram=sum(weight_gram) by shelf;
    existing_shelf
    | join kind=innerunique Shelves on $left.shelf==$right.shelf
    | where total_weight - calculated_weight_gram >= 1764
 ```