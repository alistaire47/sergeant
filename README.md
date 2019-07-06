
<!-- README.md is generated from README.Rmd. Please edit that file -->

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.1248912.svg)](https://doi.org/10.5281/zenodo.1248912)
[![Travis-CI Build
Status](https://travis-ci.org/hrbrmstr/sergeant.svg?branch=master)](https://travis-ci.org/hrbrmstr/sergeant)
[![Coverage
Status](https://codecov.io/gh/hrbrmstr/sergeant/branch/master/graph/badge.svg)](https://codecov.io/gh/hrbrmstr/sergeant)
[![CRAN\_Status\_Badge](https://www.r-pkg.org/badges/version/sergeant)](https://cran.r-project.org/package=sergeant)

# 💂 sergeant

Tools to Transform and Query Data with ‘Apache’ ‘Drill’

## \*\* IMPORTANT \*\*

Version 0.7.0 (a.k.a. the main branch) splits off the JDBC interface
into a separate package `sergeant.caffeinated`
([GitLab](https://gitlab.com/hrbrmstr/sergeant-caffeinated);
[GitHub](https://github.com/hrbrmstr/sergeant-caffeinated)).

I\# Description

Drill + `sergeant` is (IMO) a streamlined alternative to Spark +
`sparklyr` if you don’t need the ML components of Spark (i.e. just need
to query “big data” sources, need to interface with parquet, need to
combine disparate data source types — json, csv, parquet, rdbms - for
aggregation, etc). Drill also has support for spatial queries.

Using Drill SQL queries that reference parquet files on a local linux or
macOS workstation can often be more performant than doing the same data
ingestion & wrangling work with R (especially for large or disperate
data sets). Drill can often help further streamline workflows that
involve wrangling many tiny JSON files on a daily basis.

Drill can be obtained from <https://drill.apache.org/download/> (use
“Direct File Download”). Drill can also be installed via
[Docker](https://drill.apache.org/docs/running-drill-on-docker/). For
local installs on Unix-like systems, a common/suggestion location for
the Drill directory is `/usr/local/drill` as the install directory.

Drill embedded (started using the `$DRILL_BASE_DIR/bin/drill-embedded`
script) is a super-easy way to get started playing with Drill on a
single workstation and most of many workflows can “get by” using Drill
this way.

There are a few convenience wrappers for various informational SQL
queries (like `drill_version()`). Please file an PR if you add more.

Some of the more “controlling vs data ops” REST API functions aren’t
implemented. Please file a PR if you need those.

The following functions are implemented:

**`DBI`** (REST)

  - A “just enough” feature complete R `DBI` driver has been implemented
    using the Drill REST API, mostly to facilitate the `dplyr`
    interface. Use the `RJDBC` driver interface if you need more `DBI`
    functionality.
  - This also means that SQL functions unique to Drill have also been
    “implemented” (i.e. made accessible to the `dplyr` interface). If
    you have custom Drill SQL functions that need to be implemented
    please file an issue on GitHub. Many should work without it, but
    some may require a custom interface.

**`dplyr`**: (REST)

  - `src_drill`: Connect to Drill (using `dplyr`) + supporting functions

Note that a number of Drill SQL functions have been mapped to R
functions (e.g. `grepl`) to make it easier to transition from
non-database-backed SQL ops to Drill. See the help on
`drill_custom_functions` for more info on these helper Drill custom
function mappings.

**Drill APIs**:

  - `drill_connection`: Setup parameters for a Drill server/cluster
    connection
  - `drill_active`: Test whether Drill HTTP REST API server is up
  - `drill_cancel`: Cancel the query that has the given queryid
  - `drill_functions`: Show all the available Drill built-in functions &
    UDFs (Apache Drill 1.15.0+ required)
  - `drill_jdbc`: Connect to Drill using JDBC
  - `drill_metrics`: Get the current memory metrics
  - `drill_options`: List the name, default, and data type of the system
    and session options
  - `drill_popts`: Show all the available Drill options (1.15.0+)
  - `drill_rofile`: Get the profile of the query that has the given
    query id
  - `drill_profiles`: Get the profiles of running and completed queries
  - `drill_query`: Submit a query and return results
  - `drill_set`: Set Drill SYSTEM or SESSION options
  - `drill_settings_reset`: Changes (optionally, all) session settings
    back to system defaults
  - `drill_show_files`: Show files in a file system schema.
  - `drill_show_schemas`: Returns a list of available schemas.
  - `drill_stats`: Get Drillbit information, such as ports numbers
  - `drill_status`: Get the status of Drill
  - `drill_storage`: Get the list of storage plugin names and
    configurations
  - `drill_system_reset`: Changes (optionally, all) system settings back
    to system defaults
  - `drill_threads`: Get information about threads
  - `drill_uplift`: Turn a columnar query results into a type-converted
    tbl
  - `drill_use`: Change to a particular schema.
  - `drill_version`: Identify the version of Drill running

**Helpers**

  - `ctas_profile`: Generate a Drill CTAS Statement from a Query
  - `drill_up`: sart a Dockerized Drill Instance \# `sdrill_down`: stop
    a Dockerized Drill Instance by container id
  - `howall_drill`: Show all dead and running Drill Docker containers
  - `stopall_drill`: Prune all dead and running Drill Docker containers

# Installation

``` r
install.packages("sergeant", repos = "https://cinc.rud.is")
# or
devtools::install_git("https://git.rud.is/hrbrmstr/sergeant.git")
# or
devtools::install_git("https://git.sr.ht/~hrbrmstr/sergeant")
# or
devtools::install_gitlab("hrbrmstr/sergeant")
# or
devtools::install_bitbucket("hrbrmstr/sergeant")
# or
devtools::install_github("hrbrmstr/sergeant")
```

\`\`{r echo=FALSE, message=FALSE, warning=FALSE, error=FALSE}
options(width=120)

```` 

## Usage

### `dplyr` interface


```r
library(sergeant)
library(tidyverse)

# use localhost if running standalone on same system otherwise the host or IP of your Drill server
ds <- src_drill("localhost")  #ds
db <- tbl(ds, "cp.`employee.json`") 

# without `collect()`:
count(db, gender, marital_status)
##  # Source:   lazy query [?? x 3]
##  # Database: DrillConnection
##  # Groups:   gender
##    gender marital_status     n
##    <chr>  <chr>          <dbl>
##  1 F      S                297
##  2 M      M                278
##  3 M      S                276
##  4 F      M                304

count(db, gender, marital_status) %>% collect()
##  # A tibble: 4 x 3
##  # Groups:   gender [2]
##    gender marital_status     n
##    <chr>  <chr>          <dbl>
##  1 F      S                297
##  2 M      M                278
##  3 M      S                276
##  4 F      M                304

group_by(db, position_title) %>%
  count(gender) -> tmp2

group_by(db, position_title) %>%
  count(gender) %>%
  ungroup() %>%
  mutate(full_desc = ifelse(gender == "F", "Female", "Male")) %>%
  collect() %>%
  select(Title = position_title, Gender = full_desc, Count = n)
##  # A tibble: 30 x 3
##     Title                  Gender Count
##     <chr>                  <chr>  <dbl>
##   1 President              Female     1
##   2 VP Country Manager     Male       3
##   3 VP Country Manager     Female     3
##   4 VP Information Systems Female     1
##   5 VP Human Resources     Female     1
##   6 Store Manager          Female    13
##   7 VP Finance             Male       1
##   8 Store Manager          Male      11
##   9 HQ Marketing           Female     2
##  10 HQ Information Systems Female     4
##  # … with 20 more rows

arrange(db, desc(employee_id)) %>% print(n = 20)
##  # Source:     table<cp.`employee.json`> [?? x 20]
##  # Database:   DrillConnection
##  # Ordered by: desc(employee_id)
##     employee_id full_name first_name last_name position_id position_title
##     <chr>       <chr>     <chr>      <chr>     <chr>       <chr>         
##   1 999         Beverly … Beverly    Dittmar   17          Store Permane…
##   2 998         Elizabet… Elizabeth  Jantzer   17          Store Permane…
##   3 997         John Swe… John       Sweet     17          Store Permane…
##   4 996         William … William    Murphy    17          Store Permane…
##   5 995         Carol Li… Carol      Lindsay   17          Store Permane…
##   6 994         Richard … Richard    Burke     17          Store Permane…
##   7 993         Ethan Bu… Ethan      Bunosky   17          Store Permane…
##   8 992         Claudett… Claudette  Cabrera   17          Store Permane…
##   9 991         Maria Te… Maria      Terry     17          Store Permane…
##  10 990         Stacey C… Stacey     Case      17          Store Permane…
##  11 99          Elizabet… Elizabeth  Horne     18          Store Tempora…
##  12 989         Dominick… Dominick   Nutter    17          Store Permane…
##  13 988         Brian Wi… Brian      Willeford 17          Store Permane…
##  14 987         Margaret… Margaret   Clendenen 17          Store Permane…
##  15 986         Maeve Wa… Maeve      Wall      17          Store Permane…
##  16 985         Mildred … Mildred    Morrow    16          Store Tempora…
##  17 984         French W… French     Wilson    16          Store Tempora…
##  18 983         Elisabet… Elisabeth  Duncan    16          Store Tempora…
##  19 982         Linda An… Linda      Anderson  16          Store Tempora…
##  20 981         Selene W… Selene     Watson    16          Store Tempora…
##  # … with more rows, and 10 more variables: store_id <chr>,
##  #   department_id <chr>, birth_date <chr>, hire_date <chr>, salary <chr>,
##  #   supervisor_id <chr>, education_level <chr>, marital_status <chr>,
##  #   gender <chr>, management_role <chr>

mutate(db, position_title = tolower(position_title)) %>%
  mutate(salary = as.numeric(salary)) %>%
  mutate(gender = ifelse(gender == "F", "Female", "Male")) %>%
  mutate(marital_status = ifelse(marital_status == "S", "Single", "Married")) %>%
  group_by(supervisor_id) %>%
  summarise(underlings_count = n()) %>%
  collect()
##  # A tibble: 112 x 2
##     supervisor_id underlings_count
##     <chr>                    <dbl>
##   1 0                            1
##   2 1                            7
##   3 5                            9
##   4 4                            2
##   5 2                            3
##   6 20                           2
##   7 21                           4
##   8 22                           7
##   9 6                            4
##  10 36                           2
##  # … with 102 more rows
````

### REST API

``` r
dc <- drill_connection("localhost") 

drill_active(dc)
##  [1] TRUE

drill_version(dc)
##  [1] "1.15.0"

drill_storage(dc)$name
##   [1] "cp"       "dfs"      "drilldat" "hbase"    "hdfs"     "hive"    
##   [7] "kudu"     "mongo"    "my"       "s3"

drill_query(dc, "SELECT * FROM cp.`employee.json` limit 100")
##  # A tibble: 100 x 16
##     employee_id full_name first_name last_name position_id position_title
##     <chr>       <chr>     <chr>      <chr>     <chr>       <chr>         
##   1 1           Sheri No… Sheri      Nowmer    1           President     
##   2 2           Derrick … Derrick    Whelply   2           VP Country Ma…
##   3 4           Michael … Michael    Spence    2           VP Country Ma…
##   4 5           Maya Gut… Maya       Gutierrez 2           VP Country Ma…
##   5 6           Roberta … Roberta    Damstra   3           VP Informatio…
##   6 7           Rebecca … Rebecca    Kanagaki  4           VP Human Reso…
##   7 8           Kim Brun… Kim        Brunner   11          Store Manager 
##   8 9           Brenda B… Brenda     Blumberg  11          Store Manager 
##   9 10          Darren S… Darren     Stanz     5           VP Finance    
##  10 11          Jonathan… Jonathan   Murraiin  11          Store Manager 
##  # … with 90 more rows, and 10 more variables: store_id <chr>,
##  #   department_id <chr>, birth_date <chr>, hire_date <chr>, salary <chr>,
##  #   supervisor_id <chr>, education_level <chr>, marital_status <chr>,
##  #   gender <chr>, management_role <chr>

drill_query(dc, "SELECT COUNT(gender) AS gctFROM cp.`employee.json` GROUP BY gender")

drill_options(dc)
##  # A tibble: 179 x 6
##     name              value  defaultValue accessibleScopes kind  optionScope
##     <chr>             <chr>  <chr>        <chr>            <chr> <chr>      
##   1 debug.validate_i… FALSE  false        ALL              BOOL… BOOT       
##   2 debug.validate_v… FALSE  false        ALL              BOOL… BOOT       
##   3 drill.exec.funct… FALSE  false        ALL              BOOL… BOOT       
##   4 drill.exec.hasha… FALSE  false        ALL              BOOL… BOOT       
##   5 drill.exec.hashj… FALSE  false        ALL              BOOL… BOOT       
##   6 drill.exec.memor… 16777… 16777216     SYSTEM           LONG  BOOT       
##   7 drill.exec.memor… 0.1    0.1          SYSTEM           DOUB… BOOT       
##   8 drill.exec.stora… dir    dir          ALL              STRI… BOOT       
##   9 drill.exec.stora… filen… filename     ALL              STRI… BOOT       
##  10 drill.exec.stora… filep… filepath     ALL              STRI… BOOT       
##  # … with 169 more rows

drill_options(dc, "json")
##  # A tibble: 10 x 6
##     name               value defaultValue accessibleScopes kind  optionScope
##     <chr>              <chr> <chr>        <chr>            <chr> <chr>      
##   1 store.hive.maprdb… FALSE false        ALL              BOOL… BOOT       
##   2 store.json.all_te… TRUE  false        ALL              BOOL… SYSTEM     
##   3 store.json.extend… TRUE  false        ALL              BOOL… SYSTEM     
##   4 store.json.read_n… FALSE false        ALL              BOOL… BOOT       
##   5 store.json.reader… TRUE  true         ALL              BOOL… BOOT       
##   6 store.json.reader… TRUE  false        ALL              BOOL… SYSTEM     
##   7 store.json.reader… TRUE  false        ALL              BOOL… SYSTEM     
##   8 store.json.writer… TRUE  true         ALL              BOOL… BOOT       
##   9 store.json.writer… TRUE  true         ALL              BOOL… BOOT       
##  10 store.json.writer… TRUE  false        ALL              BOOL… SYSTEM
```

## Working with parquet files

``` r
drill_query(dc, "SELECT * FROM dfs.`/usr/local/drill/sample-data/nation.parquet` LIMIT 5")
##  # A tibble: 5 x 4
##    N_NATIONKEY N_NAME    N_REGIONKEY N_COMMENT           
##          <dbl> <chr>           <dbl> <chr>               
##  1           0 ALGERIA             0 haggle. carefully f 
##  2           1 ARGENTINA           1 al foxes promise sly
##  3           2 BRAZIL              1 y alongside of the p
##  4           3 CANADA              1 eas hang ironic, sil
##  5           4 EGYPT               4 y above the carefull
```

Including multiple parquet files in different directories (note the
wildcard support):

``` r
drill_query(dc, "SELECT * FROM dfs.`/usr/local/drill/sample-data/nations*/nations*.parquet` LIMIT 5")
##  # A tibble: 5 x 5
##    dir0      N_NATIONKEY N_NAME    N_REGIONKEY N_COMMENT           
##    <chr>           <dbl> <chr>           <dbl> <chr>               
##  1 nationsSF           0 ALGERIA             0 haggle. carefully f 
##  2 nationsSF           1 ARGENTINA           1 al foxes promise sly
##  3 nationsSF           2 BRAZIL              1 y alongside of the p
##  4 nationsSF           3 CANADA              1 eas hang ironic, sil
##  5 nationsSF           4 EGYPT               4 y above the carefull
```

### Drill has built-in support for spatial ops

Via: <https://github.com/k255/drill-gis>

A common use case is to select data within boundary of given polygon:

``` r
drill_query(dc, "
select columns[2] as city, columns[4] as lon, columns[3] as lat
    from cp.`sample-data/CA-cities.csv`
    where
        ST_Within(
            ST_Point(columns[4], columns[3]),
            ST_GeomFromText(
                'POLYGON((-121.95 37.28, -121.94 37.35, -121.84 37.35, -121.84 37.28, -121.95 37.28))'
                )
            )
")
##  # A tibble: 7 x 3
##    city        lon          lat       
##    <chr>       <chr>        <chr>     
##  1 Burbank     -121.9316233 37.3232752
##  2 San Jose    -121.8949555 37.3393857
##  3 Lick        -121.8457863 37.2871647
##  4 Willow Glen -121.8896771 37.3085532
##  5 Buena Vista -121.9166227 37.3213308
##  6 Parkmoor    -121.9307898 37.3210531
##  7 Fruitdale   -121.932746  37.31086
```

### sergeant Metrics

| Lang | \# Files | (%) | LoC | (%) | Blank lines | (%) | \# Lines | (%) |
| :--- | -------: | --: | --: | --: | ----------: | --: | -------: | --: |
| Rmd  |        1 |   1 |  56 |   1 |          55 |   1 |       90 |   1 |

## Code of Conduct

Please note that this project is released with a [Contributor Code of
Conduct](CONDUCT.md). By participating in this project you agree to
abide by its terms.
