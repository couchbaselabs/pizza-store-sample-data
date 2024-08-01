# Loading Pizza Store Sample Data to Capella

This guide will help you load pizza store sample data into Capella.

## Step 0: Prerequisites

1. You will need the `cbimport` [command line utility](https://docs.couchbase.com/cloud/reference/command-line-tools.html). For Windows, you may want to put it into your PATH.

2. Create a database cluster on Capella. Within that cluster, create a bucket called **pizzastore**. Within that bucket, create a scope called **orderapp**. Within the orderapp scope, create the following collections: customers, ingredients, items, orders, stats, stores. (You can perform all of these via the [REST API](https://docs.couchbase.com/cloud/management-api-reference/index.html#tag/Buckets-Scopes-and-Collections) if you wish).

3. Add an [allowed IP address](https://docs.couchbase.com/cloud/clusters/allow-ip-address.html) for the computer from which you will be loading this data.

4. Find and copy the connection string for cluster (in Capella, click the "☁ Connect" tab).

5. Create a [database credential](https://docs.couchbase.com/cloud/clusters/manage-database-users.html#create-database-credentials), e.g. `demouser/Password123!`

6. Download a security certificate (click "⚙ Settings" and then "Security Certificate"). Save this file as "pizzastore-root-certificate.pem" in a known location, for instance your local `/etc/ssl` folder.

### Customers data:

First, customers data.

To create a collection for customers or any other data in this guide, you can use the REST API:

```
curl -X POST -u demouser:Password123! \
https://<CONNECTION_STRING>:18091/pools/default/buckets/pizzastore/scopes/orderapp/collections \
-d name=customers --cacert /etc/ssl/pizzastore-root-certificate.pem
```

For Windows, if you don't have curl installed, you can use a tool like Postman, or here's a PowerShell example:
```
Invoke-RestMethod -Uri "https://<CONNECTION_STRING>:18091/pools/default/buckets/pizzastore/scopes/orderapp/collections" -Method Post -Credential (New-Object System.Management.Automation.PSCredential('demouser', (ConvertTo-SecureString 'Password123!' -AsPlainText -Force))) -Body @{ name = 'my_collection_name' } -Certificate (Get-PfxCertificate -FilePath ".\pizzastore-root-certificate.pem") -ContentType "application/x-www-form-urlencoded"
```

Once the collection is created, load the customers data using `cbimport json`:

```
cbimport json -c https://<CONNECTION_STRING> -u demouser -p Password123! -b pizzastore -d file://customers.json -f lines -g %customerId% --scope-collection-exp orderapp.customers -t 4 --cacert /path/to/pizzastore-root-certificate.pem
```

(For Mac/Linux, you may need to start the command with `~/bin/`).

### Orders data:

Assuming the orders collection is already create, import the data with `cbimport json`:

```
cbimport json -c https://<CONNECTION_STRING> -u demouser -p Password123! -b pizzastore -d file://orders.json -f lines -g %orderId% --scope-collection-exp orderapp.orders -t 4 --cacert /path/to/pizzastore-root-certificate.pem
```

### Items data:

Repeat for items data. Notice that the data is in *list* format this time instead of *lines* format.

```
cbimport json -c https://<CONNECTION_STRING> -u demouser -p Password123! -b pizzastore -d file://items.json -f list -g %itemId% --scope-collection-exp orderapp.items -t 4 --cacert /path/to/pizzastore-root-certificate.pem
```

### Stats data:

Repeat for stats data. This is in lines format.

```
cbimport json -c https://<CONNECTION_STRING> -u demouser -p Password123! -b pizzastore -d file://stats.json -f lines -g %game_id%_%play_id% --scope-collection-exp orderapp.stats -t 4 --cacert /path/to/pizzastore-root-certificate.pem
```

### Ingredients data

Import ingredients data from a TSV file using `cbimport csv`. Also note the `-infer-types` flag: cbimport will look at each value and decide whether it is a string, integer, or boolean value and put the inferred type into the document.

```
cbimport csv -c https://<CONNECTION_STRING> -u demouser -p Password123! -b pizzastore -d file://ingredients.tsv -g %id% --scope-collection-exp orderapp.ingredients --infer-types -t 4 --field-separator '\t' --cacert /path/to/pizzastore-root-certificate.pem
```

### Stores data

Finally, import stores data using `cbimport json`.

```
cbimport json -c https://<CONNECTION_STRING> -u demouser -p Password123! -b pizzastore -d file://stores.json> -f list -g %storeId% --scope-collection-exp orderapp.stores -t 4 --cacert /path/to/pizzastore-root-certificate.pem
```