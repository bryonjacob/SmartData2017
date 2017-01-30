<h2 id="setup-dw">Creating a data.world account</h2>

1.  Navigate to <a href="https://data.world/">https://data.world/</a>

1.  Click `Join` in the upper right-hand corner

1.  Fill in the form and verify your email address

<h2 id="setup-dataset">Create a dataset for the tutorial</h2>

1. Under your name on the right-hand side of the page, see `Your Datasets`.  Click the  `+` sign to create a new one. 

1. Call the dataset SmartData2017  (no spaces; give it this exact name to make the rest of the tutorial go smoothly)

1. Click `Create Dataset`

<h2 id="setup-r">Setting up R Studio</h2>

1.  First, you need to make sure you have R installed:

    [https://cran.rstudio.com/](https://cran.rstudio.com/)

1.  Next, you can download and run the RStudio installer for your platform
    
    [https://www.rstudio.com/products/rstudio/download/](https://www.rstudio.com/products/rstudio/download/)
   
1.  Once you have R and R Studio installed, we need to install the R package dependencies that this project depends upon - there is only one package we need to install, the `ggplot2` package.  From within R Studio, you can install it from the `Console` tab by typing:

    ```
    install.packages("ggplot2")
    require ("ggplot2")
    ```

<h2 id="download-data">Download the data file</h2>

We will be working from <a href="https://github.com/bryonjacob/SmartData2017/raw/master/docs/CatanSettlementBuilders-2016-H2.xlsx">this spreadsheet</a>

<h2 id="queries">Queries</h2>
<h3>Transactions by SKU</h3>

```
PREFIX t: <http://data.world/bryon/smartdata-2017/CatanSettlementBuilders-2016-H2.xlsx/TRANSACTIONS#>

SELECT DISTINCT ?txid ?date ?price ?sku WHERE {

# Picks out the transaction id, date, price and sku from the transaction table
    ?record  t:txid ?txid ; t:date ?date ; t:price ?price ; t:sku ?sku .

}
ORDER BY ?date
```
<h3>Transactions by UNSPSC</h3>

```
PREFIX t: <http://data.world/bryon/smartdata-2017/CatanSettlementBuilders-2016-H2.xlsx/TRANSACTIONS#>
PREFIX p: <http://data.world/bryon/smartdata-2017/CatanSettlementBuilders-2016-H2.xlsx/PRODUCTS#>

SELECT DISTINCT ?txid ?date ?price ?unspsc WHERE {

# Picks out the transaction id, date, price from the transaction table
# Join the sku with the SKU table, and look up the UNSPSC code for that product
    ?record  t:txid ?txid ; t:date ?date ; t:price ?price ; t:sku/^p:sku/p:unspsc_raw ?unspsc .

}
ORDER BY ?date
```
<h3>UNSPSC Structure</h3>
```
PREFIX p: <http://data.world/bryon/smartdata-2017/CatanSettlementBuilders-2016-H2.xlsx/PRODUCTS#>
PREFIX t: <http://data.world/bryon/smartdata-2017/CatanSettlementBuilders-2016-H2.xlsx/TRANSACTIONS#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX unspsc: <http://workingontologist.org/vocabularies/unspsc#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT  ?code ?desc ?type ?parent ?pdesc ?ptype
WHERE {
  SERVICE <https://query.data.world/sparql/dallemang/unspsc-codes-in-utf-8> {
       ?record  skos:notation ?code . 
       ?record skos:prefLabel ?desc . 
       ?record a / rdfs:label ?type .
       ?record skos:broader ?precord . 
       ?precord skos:notation ?parent . 
       ?precord skos:prefLabel  ?pdesc . 
       ?precord a / rdfs:label ?ptype . 
    }
} LIMIT 100
```

<h3>Transactions by Class</h3>
```
PREFIX p: <http://data.world/bryon/smartdata-2017/CatanSettlementBuilders-2016-H2.xlsx/PRODUCTS#>
PREFIX t: <http://data.world/bryon/smartdata-2017/CatanSettlementBuilders-2016-H2.xlsx/TRANSACTIONS#>
PREFIX unspsc: <http://workingontologist.org/vocabularies/unspsc#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT DISTINCT ?txid ?date ?price ?class   WHERE {

# Picks out the transaction id, date, price from the transaction table
# Join the sku with the SKU table, and look up the UNSPSC code for that product
    ?record t:txid ?txid ; t:date ?date ; t:price ?price ; t:sku/^p:sku/p:unspsc_raw ?unspsc .

# Looks up that UNSPSC code in the UNSPSC table, and finds all broader codes.  Filter to find the one that is 
# a "Class" in the UNSPSC hierarchy.  Get its label (call it ?class)
    SERVICE <https://query.data.world/sparql/dallemang/unspsc-codes-in-utf-8> {
        ?unspsc ^skos:notation/skos:broader* ?cat . 
        ?cat a unspsc:Class ; skos:prefLabel ?class .
    } 
}
ORDER BY ?date
```
<h3>Transactions by Business Sector</h3>
```
PREFIX c: <http://data.world/bryon/smartdata-2017/CatanSettlementBuilders-2016-H2.xlsx/COMPANIES#>
PREFIX p: <http://data.world/bryon/smartdata-2017/CatanSettlementBuilders-2016-H2.xlsx/PRODUCTS#>
PREFIX t: <http://data.world/bryon/smartdata-2017/CatanSettlementBuilders-2016-H2.xlsx/TRANSACTIONS#>
PREFIX naics: <http://workingontologist.org/vocabularies/naics/2012#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT DISTINCT ?txid ?date ?price ?sku ?supplier ?purchaser ?supplier_category ?purchaser_category WHERE {
   ?record t:txid ?txid ; t:date ?date ; t:price ?price ; t:sku ?sku ;
           t:purchaser ?purchaser ; t:supplier ?supplier ;
           # Find NAICS codes for the supplier and purchaser
           t:purchaser/^c:company/c:naics_raw ?purchaser_naics ; 
           t:supplier/^c:company/c:naics_raw ?supplier_naics .

 # Look up the Sector for those NAICS codes
    SERVICE <https://query.data.world/sparql/dallemang/naics-codes-2012> {
        ?supplier_naics ^skos:notation/skos:broader* ?scat . 
        ?scat a naics:Sector ; skos:prefLabel ?supplier_category .
        ?purchaser_naics ^skos:notation/skos:broader* ?pcat. 
        ?pcat a naics:Sector ; skos:prefLabel ?purchaser_category .
    } 
}
ORDER BY ?date
```
<h3>Transactions by Product and Business Sector</h3>
```
PREFIX c: <http://data.world/bryon/smartdata-2017/CatanSettlementBuilders-2016-H2.xlsx/COMPANIES#>
PREFIX p: <http://data.world/bryon/smartdata-2017/CatanSettlementBuilders-2016-H2.xlsx/PRODUCTS#>
PREFIX t: <http://data.world/bryon/smartdata-2017/CatanSettlementBuilders-2016-H2.xlsx/TRANSACTIONS#>
PREFIX naics: <http://workingontologist.org/vocabularies/naics/2012#>
PREFIX unspsc: <http://workingontologist.org/vocabularies/unspsc#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT DISTINCT ?txid ?date ?price ?sku ?product_class ?supplier ?purchaser ?supplier_category ?purchaser_category WHERE {
  ?record t:txid ?txid ; t:date ?date ; t:price ?price ; t:sku ?sku ;
          t:purchaser ?purchaser ; t:supplier ?supplier ;
          # Find UNSPSC code for the product
          t:sku/^p:sku/p:unspsc_raw ?unspsc ;

          # Find NAICS codes for the supplier and purchaser
          t:purchaser/^c:company/c:naics_raw ?purchaser_naics ; 
          t:supplier/^c:company/c:naics_raw ?supplier_naics  .

    # Look up that UNSPSC code in the UNSPSC table, and finds all broader codes.  Filter to find the one that is 
    # a "Class" in the UNSPSC hierarchy.  Get its label (call it ?class)
    SERVICE <https://query.data.world/sparql/dallemang/unspsc-codes-in-utf-8> {
        ?unspsc ^skos:notation/skos:broader* ?cat . 
        ?cat a unspsc:Class ; skos:prefLabel ?product_class .
    } 

    # Look up the Sector for those NAICS codes
    SERVICE <https://query.data.world/sparql/dallemang/naics-codes-2012> {
        ?supplier_naics ^skos:notation/skos:broader* ?scat . 
        ?scat a naics:Sector ; skos:prefLabel ?supplier_category .
        ?purchaser_naics ^skos:notation/skos:broader* ?pcat. 
        ?pcat a naics:Sector ; skos:prefLabel ?purchaser_category .
    } 
}
ORDER BY ?date
```
<h3>R code</h3>
```
> ggplot (df, aes(x=as.Date(date), y=price, fill=product_class)) +
     geom_bar(stat="identity") +
     facet_grid (supplier_category ~ purchaser_category)
```
```
> ggplot (df, aes(x=product_class, y=price)) +
     geom_boxplot() +
     geom_jitter(position = position_jitter(0.2)) +
     facet_grid (supplier_category ~ purchaser_category)
```
```
> ggplot (df[df$purchaser_category == "Construction" & df$product_class == "Livestock",], aes(x=date, y=price, fill=product_class)) +
     geom_point(stat="identity") +
     facet_grid (supplier ~ purchaser)
```


