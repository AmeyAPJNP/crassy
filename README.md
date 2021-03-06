# crassy
A `Sparklyr-Cassandra-Connector` using the Datastax `Spark-Cassandra-Connector`.

This purpose of this library is to allow existing data in Cassandra to be analysed in R. Therefore, the available operations in this extension are to  
 
 - Load Cassandra partitions or whole tables with server-side filtering and selecting 
 - Writes (updates) Cassandra tables

## Usage

Connect to a spark cluster and get the session

```
library(sparklyr)
library(dplyr)
library(crassy)

spark_install("2.0.2")

# Not the recommended way to handle config 
# Better to put in working directory under config.yml
# See http://spark.rstudio.com/deployment.html#configuration 
# and https://github.com/rstudio/config

config <- spark_config()
config$sparklyr.defaultPackages = "com.datastax.spark:spark-cassandra-connector_2.11:2.0.0-M3"
config$spark.cassandra.connection.host = 'localhost'

sc <- spark_connect(
  master     = 'local', 
  spark_home = spark_home_dir(),
  config = config
)
```

Load some table into spark

```
spk_handle = crassy::spark_load_cassandra_table(
  sc            = sc,
  cass_keyspace = "system_auth", 
  cass_tbl      = "roles", 
  spk_tbl_name  = "spk_tbl",
  select_cols   = c("role", "can_login")
)
```

Write some table to Cassandra, assuming the table already exists

```
tbl_iris = copy_to(sc, iris, overwrite = TRUE)

# Will throw an error if the table does not exist!
crassy::spark_write_cassandra_table(tbl_iris, "test", "iris")
```

### Note

For my use case - analysis on partitions, or sections of partitions - I have found this to be just enough functionality to be workable, and use this code in production. 

The other use case I've found I needed is where I need to search for a particular row in a Cassandra table, which doesn't need to overhead of passing the data to Spark. For an example of how submitting raw CQL to the cluster can be done through R, please refer to this library - https://github.com/AkhilNairAmey/CQLConnect.

Unfortunately the library is very specific to my use case, as I believe to make a more generic package would involve some fairly heavy duty Scala introspective techniques, and I'm just not there yet.  If you have any ideas how this could be achieved, please feel free to open an issue, or find me at akhil.nair@amey.co.uk

Thanks,
Akhil
