## How Design a Database: Considerations and Good Practices when Creating a Database

Database modelling and design is a highly important consideration within a project. The time invested in ensuring it is setup correctly in the beginning will likely save you many multiples in maintenance and re-design time should you neglect to do so.


### What is a Database?
In the most general sense, a database is an organized collection of data. More specifically, and more appropriately for our purposes, a database is an electronic system that allows data to be easily accessed, manipulated and updated. In this article I will be largely considering the design of Relational Databases, which come in many flavours such as MySQL, MariaDB, Postgres, Oracle, MSSQL and a variety of others.

As much as I can I will be trying to be database agnostic in this instance, though there may be particular reasons for considering one over another for your project.


### Design Considerations
When considering the design of your database there are several important areas to think about.

The first of these is identifying your entities. Essentially there are four types of entity: people, things, events and locations. All data you will be wanting to put into a database should fit into one of these (anything outside of this should probably be considered an attribute). Knowing what entities you have will inform how you structure your data into the appropriate tables.  
   
   
#### Entity Relationships
Once you have established the entities (and thus many of the initial tables you will need), you will also want to identify how these entities relate to one another.  

There are several ways entities can relate to one another.  

The first of these is a one-to-one relationship where an entity can have one and only one of the second entity (i.e. a customer may have a social security number). It is less common as such data is usually stored within the same table, but there are sometimes reasons such to isolate part of a table for security reasons, or to store information that applies only to a subset of the main table which mean this is worth doing. Both tables must share a common field to be able to provide such a link.  

The second much more common type of relationship is a one-to-many where one record in the first table is linked to many records in the second table (i.e. the relationship between a Customer and an Order table)  

The third type is a many-to-many where multiple records in the first table are linked to many records in the second table (i.e. the relationship between an Order and a Product table). To detect existing many-to-many relationships between your tables, it is important that you consider both sides of the relationship as it may be bi-directional.
In order to maintain uniqueness and to be able to join the tables together, there must be a primary key.  

To represent a many-to-many relationship, you will need to create a third table (often called a junction table) that breaks down the many-to-many relationship into two one-to-many relationships. By inserting the primary key from each of the two tables into the third table, it can record each occurrence, or instance, of the relationship. For example, as in my earlier example, an order can have many products, and each product can appear on many orders.  

Once you have defined a simple table structure, you will need to look into what sort of data should belong in each of the tables.  

#### Normalisation
One of the first things you will need to do is called normalisation of data and involves a number of key features including no repeating groups of columns in an entity, all attributes of an entity bing fully dependent on the whole primary key and not on other attributes.
The first area then to look at is the removal of unnecessary duplication of data and thus redundancy. Consider a situation where you have a sales table like this:  

| Customer | Item Purchased | Purchase Price | Sales date |
|-------------|---------------|--------|---------------|
| Fred Bloggs | T-shirt (red) | £10.99 | June 4th 2017 |
| Jane Doe | Skirt | £20.00 | July 12th 2017 |
| Bob Smith | T-shirt (green) | £10.99 | July 3rd 2017 |
| Mary Jones | Trousers | £15.99 | August 28th 2017 |
| Anne Peters | Trousers | £15.99 | August 30th 2017 |

In this situation, separating the customers out would allow you to refer to them in this instance but also to refer to them in different scenarios without having to duplicate the data, essentially decoupling the entities. Doing this allows you more flexibility and the ability to modify the data around the customer more easily which will help in preventing the data getting out of line, with some places having one set of data and other having another (i.e. if they changed their name etc. the sales data which is associated with their customer record would not be lost). Doing this also prevent you confusing data between two people who share some data (i.e. there are two Bob Smiths in my example so you could end up linking together data which is not really theirs if you don't add in a way of separating out which one you are referring to).  
From the data provided in the example above, the first level of decoupling you would want to achieve would be to separate out the customers into their own table and the sales items into their own table.  
At this point it is worth noting the importance of good naming for tables and fields. For a name to be effective it should be meaningful and contain neither numbers nor spaces. To connect multiple words together you should use camel case (i.e. Item Name becomes ItemName). When creating tables names use the plural (i.e. Items or Customers) but for field names the singular should be used (i.e. ItemName, CustomerAddress). Abbreviations can be used as long as it is easily understood what is meant (i.e. CustName)  

As discussed above, the Customers table would then hold the customer names (also other associated information like shipping address etc.) and the Items table would hold the information about the item  

| CustId | CustFirstName | CustLastName | CustAddrLine1 | CustCity | CustPcode | Title |
|---|------|--------|-------------|-------|----------|----|
| 1 |	Fred | Bloggs | 11 Baker St | Leeds | LE10 6TF | Mr |
| 2	| Jane | Doe | 10 Acacia Avenue | Birmingham | B12 3AB | Ms |
| 3	| Bob | Smith | 123 Penny Lane | Manchester | M14 8QH | Prof |
| 4	| Mary | Jones | 42 Abbey Rd | Liverpool | L32 1NS | Mrs |
| 5 |	Bob | Smith | 123 Imagination St | London | SE1 1JS | Mr |
| 6 |	Anne | Peters | 1 Gable Rd | Oxford | OX 1JP | Miss |  

| ItemId | ItemDescription | Item Colour | ItemPrice | Currency |
|---|---------|-----|-------|---|
| 1 |	T-shirt	| Red	| 10.99	| £ |
| 2	| T-shirt	| Green	| 10.99	| £ |
| 3	| Skirt	| Black	| 20.00	| £ |
| 4	| Trousers	| Black	| 15.99	| £ |  

This would then result in a third table which holds the same data we saw in the initial example, but now contained in a relational form like this.  

| CustomerId | ItemId | SalesDate |
|---|---|------------|
| 1 | 1 | 2017-06-04 |
| 5 | 2 | 2017-07-03 |
| 2 | 3 | 2017-07-12 |
| 4 | 4 | 2017-08-28 |
| 6 | 4 | 2017-08-30 |  

We could further abstract the data by creating a CustomerAddresses table which was attached to the Customer record by id which might be useful if we want to record multiple addresses on a single record (i.e. a Billing address and a Shipping address). Alternatively we could add a Currencies table which would show the correct currency and associated symbols etc. if that was required or even a Countries table (which might go even further and tell you language, currency, relevant symbols, currency separators/format etc. if your application was multilingual) or perhaps adding a Titles table.  
Each of the fields in the database should be defined around the type of data it is expected to hold to ensure consistency. Date fields should be held as either Date or Datetime fields to prevent inconsistencies with formatting such as the difference between how UK and US dates are displayed, numeric fields should be defined as int, float or similar and you should specify the decimal places you wish to hold (the money datatype in MSSQL for example already defines 4 decimal places, however others will require you to specify) and whether both positive and negative values are permitted.  Another thing you may need to consider is whether the data that you will be putting the table is a required piece of information or whether it is optional and if so what you want the database to put if it is not provided. If you chose not to permit NULL values, you will need to provide a default value. If you record a NULL value (which is explicitly a blank field and is different to a field with a value of zero) this may mean having to write exception code if the data is missing so that you don’t get SQL exceptions.  

You will also need to consider if your data is dependent on any other pieces of data. In the example the cross reference table is meaningless without the other tables and so the Primary Key (PK) which is the unique way of identifying a record in the table should be matched by a Foreign Key (FK) in the second table that matches it and thus provides the data with referential integrity and ensures that no data can be added or deleted that would leave a query unable to retrieve all the required information.  Often the Primary Key is just a single field with an auto-incrementing integer value, however it can be multiple values which together make a unique key to the table.  

It’s also worth bearing in mind if you data is likely to have a fixed set of answers (like yes or no or a rating of 1 to 5) or whether the data is likely to be free text entry which be very large and might need to be typed as a BLOB or CLOB.  

For situations where the answer is one of two fixed answers you should define the data type as binary and where there are a finite number of possible answers it is probably worth creating a cross reference look up table so that they can be stored as integer data which is faster for lookups.  

It is unwise to define either image or BLOB/CLOB data columns in frequently queried tables because it is like to cause performance issues and these should be treated as a special case and separated out.  

Also, it is worth considering the security of authentication data such as passwords if these are being stored as part of your database. The first rule is never store passwords as plaintext. Secondly, you shouldn’t encrypt them either (as what can be encrypted can equally be decrypted). Instead you should be storing them as a hash which takes an input and returns as a new string (hash) which is difficult to reverse. Popular algorithms for hashing passwords are SHA-2 or SHA-3 (In the past MD-5 and SHA-1 were also popular, but MD-5 has been broken and a theoretical attack on SHA-1 has been reported too so probably better to avoid using them for password hashing). It is also probably worth considering separating the password hashes into a separate table rather than storing them with the username for a further level of protection.  
  
  
   
  
#### Denormalisation.  
 
Having spent a chunk of time ensuring your data is as decoupled and unduplicated as it can be, now is the time where you now need to look at this will work for queries in the real world.

It is important to understand how your database will be used. Will there be lots of queries for small amounts of data or fewer large queries to provide data for a report or graph or suchlike?

If you have larger queries which will require multiple joins as the normalised structure stands, you may decide that in order to decrease the performance penalty of lookup times against these tables, some of your abstraction and de-duplication will work against you and therefore consider strategically reintroduce duplicate data into some of the tables to improve performance.

It is a trade-off and probably not a step that you will want to introduce until you have gone live and done some analysis on how it performs, but it is something to bear in mind as a final stage in database design.

#### A further note around maintenance and documentation…

When you create your database be sure to create a SQL script so that this can be replicated and reproduced elsewhere with minimum effort.  This will allow you to have test and production servers that mirror each other easily as well as providing a full history of the database development.

Additionally, as the project develops and you add or change columns, these updates should be written as SQL scripts too (preferably stored with the date they were first applied in the file name for quick date ordering purposes) so that they can be applied over the top of a second copy of the database and can get it back into line quickly and easily.

Further, even though you should have structured and named your database in such a way as to be relatively easy to understand, that does not replace the need for proper documentation.
