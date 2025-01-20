# ![veld chain](https://raw.githubusercontent.com/veldhub/.github/refs/heads/main/images/symbol_V_letter.png) veld_chain__automatic_tei-ification_of_gutenberg

This repo contains [chain velds](https://zenodo.org/records/13322913) encapsulating several
processing workflows:
- download of entire [project gutenberg](https://www.gutenberg.org/) metadata
- ingestion into a local triplestore for complex sparql queryies
- query and download of all german books from gutenberg that don't have a TEI representation but a
  txt one
- running a veldified version of [teitok tools](https://github.com/ufal/teitok-tools) + 
  [udpipe](https://lindat.mff.cuni.cz/services/udpipe/) to automatically generate TEI files of them.

## requirements

- git
- docker compose (note: older docker compose versions require running `docker-compose` instead of 
  `docker compose`)

Clone this repo with all its submodules
```
git clone --recurse-submodules https://github.com/veldhub/veld_chain__automatic_tei-ification_of_gutenberg.git
```

## how to reproduce

There are two ways to reproduce the entirety of all chains: 
- individually: going through each following step sequentially. See 
[individual chains](#individual-chains)
- multichain: aggregates all individual chains into one multichain. See [multichain](#multichain)

### individual chains

All details of the following chains can be inspected within their respective veld_* yaml file:

**[./veld_step1_download_gutenberg_metadata.yaml](./veld_step1_download_gutenberg_metadata.yaml)**

Since project gutenberg doesn't offer an API, it's not programmatically possible to query its data.
It does however offer a download of its entire metadata as rdf-xml ( 
https://gutenberg.org/cache/epub/feeds/rdf-files.tar.bz2 ), which will be used for querying. This 
metadata is downloaded into [./data/gutenberg_rdf/](./data/gutenberg_rdf/)

```
docker compose -f veld_step1_download_gutenberg_metadata.yaml up
```

**[./veld_step2_run_server.yaml](./veld_step2_run_server.yaml)**

Querying the data is done via a apache fuseki triplestore, since sparql can adapt to the data's 
complexity and also allows for high flexibility should query requirements change at some point. For 
this, a triplestore is started in this step. Configuration for the server can be found in
[./data/fuseki_config/](./data/fuseki_config/). The server can be reached at 
[http://localhost:3030](http://localhost:3030). Note: this service needs to be kept running for step 
3 (ingestion) and 4 (querying). After step 4, it can be shut down.

```
docker compose -f veld_step2_run_server.yaml up
```

**[./veld_step3_import_rdf.yaml](./veld_step3_import_rdf.yaml)**

Once the server is running via step 2, the metadata downloaded by step 1 can be ingested. Note that 
this step can take a long time (on a AMD Ryzen 7 4800H it took 11 hours). 

```
docker compose -f veld_step3_import_rdf.yaml up
```

**[./veld_step4_query_books_urls.yaml](./veld_step4_query_books_urls.yaml)**

After the metadata is ingested, the triplestore can be queried for german books that have no TEI but
txt files. The query for this can be found at 
[./data/queries/german_books_txt_no_tei.rq](./data/queries/german_books_txt_no_tei.rq), and the 
output is saved as csv file in [./data/fuseki_export/](./data/fuseki_export/)

```
docker compose -f veld_step4_query_books_urls.yaml up
```

**[./veld_step5_download_gutenberg_books.yaml](./veld_step5_download_gutenberg_books.yaml)**

The csv file from step 4 contains book download links and their designated file names. This csv is
used as input for downloading the book's txt files within this step. The books can be found at 
[./data/gutenberg_books/](./data/gutenberg_books/).

```
docker compose -f veld_step5_download_gutenberg_books.yaml up
```

**[./veld_step6_convert_books_to_teitok.yaml](./veld_step6_convert_books_to_teitok.yaml)**

TODO: yet to implement.

## multichain

[./veld_multichain_all.yaml](./veld_multichain_all.yaml)

All of the individual chains above are also aggregated into one multichain which keeps the order of
the steps above. Note that the multichain simply references the indvidual chains by loading them
from their respective veld_* yaml file. This means that any change to any such file will be also
reflected in this multichain. For more details, see 
[./veld_multichain_all.yaml](./veld_multichain_all.yaml) 

```
docker compose -f veld_multichain_all.yaml up
```

