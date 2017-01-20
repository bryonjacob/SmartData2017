<h2 id="setup-dw">Creating a data.world account</h2>

1.  Navigate to https://data.world/

1.  Click `Join` in the upper right-hand corner

1.  Fill in the form and verify your email address

<h2 id="setup-dataset">Create a dataset for the tutorial</h2>

1. Under your name on the right-hand side of the page, see "Your Dataets".  Click the '+' sign to create a new one. 

1. Call the dataset SmartData2017  (no spaces; give it this exact name to make the rest of the tutorial go smoothly)

<h2 id="setup-r">Setting up R Studio</h2>

1.  First, you need to make sure you have R installed:

    [https://cran.rstudio.com/](https://cran.rstudio.com/)

1.  Next, you can download and run the RStudio installer for your platform
    
    [https://www.rstudio.com/products/rstudio/download/](https://cran.rstudio.com/)
   
1.  Once you have R and R Studio installed, we need to install the R package dependencies that this project depends upon - there is only one package we need to install, the `ggplot2` package.  From within R Studio, you can install it from the `Console` tab by typing:

    ```
    install.packages("ggplot2")
    ```

<h2 id="download-data">Download the data file</h2>

We will be working from <a href="https://github.com/bryonjacob/SmartData2017/raw/master/docs/CatanSettlementBuilders-2016-H2.xlsx">this spreadsheet</a>

<h2 id="queries">Queries</h2>
<h3>Transactions by SKU</h3>

```
PREFIX t: <http://data.world/bryon/catan-settlement-builders-h-216/CatanSettlementBuilders-2016-H2.xlsx/TRANSACTIONS#>

SELECT DISTINCT ?txid ?date ?price ?sku WHERE {

# Picks out the transaction id, date, price and sku from the transaction table
    [ t:txid ?txid ; t:date ?date ; t:price ?price ; t:sku ?sku ] .

}
ORDER BY ?date
```
<h3>Transactions by UNSPSC</h3>

```
PREFIX t: <http://data.world/bryon/catan-settlement-builders-h-216/CatanSettlementBuilders-2016-H2.xlsx/TRANSACTIONS#>
PREFIX p: <http://data.world/bryon/catan-settlement-builders-h-216/CatanSettlementBuilders-2016-H2.xlsx/PRODUCTS#>

SELECT DISTINCT ?txid ?date ?price ?unspsc WHERE {

# Picks out the transaction id, date, price from the transaction table
# Join the sku with the SKU table, and look up the UNSPSC code for that product
    [ t:txid ?txid ; t:date ?date ; t:price ?price ; t:sku/^p:sku/p:unspsc_raw ?unspsc ] .

}
ORDER BY ?date
```
