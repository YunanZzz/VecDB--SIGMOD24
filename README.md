# Finding Root Causes of Performance Gap Between the State-of-the-Art Generalized and Specialized Vector Database

By comparing the state-of-the-art generalized vector database (i.e., Alibaba PASE) and the state-of-the-art specialized vector database (i.e., Facebook Faiss). We instrument the codebase of each system (as they are open-sourced fortunately) and drill down the underlying reasons that cause the performance gap and analyze whether the gap is bridgeable. We present a comprehensive experimental study and in-depth analysis to compare the two systems in terms of index construction, index size, and query processing on a variety of real-world datasets. Based on the results, we provide lessons and directions for building a generalized vector database in the future that achieves similar performance to the state-of-the-art specialized vector database.

# Repository Contents

## **postgresql-11.0** 
This directory contains the source code distribution of the PostgreSQL
database management system.


## **faiss** 
This directory contains the source code of Faiss, a library for efficient similarity search and clustering of dense vectors

* **faiss/IndexIVFFlat.cpp:** Faiss index IVF_FLAT implementation

* **faiss/IndexIVFPQ.cpp:** Faiss index IVF_PQ implementation

* **faiss/IndexHNSW.cpp:** Faiss index HNSW implementation


## **pase**

Code of PASE is in the directory **postgresql-11.0/contrib/pase**. 

* **ivfflat:** PASE index IVF_FLAT implementation

* **ivfpq:** PASE index IVF_PQ implementation

* **hnsw:** PASE index HNSW implementation

* **sql:** Sample SQL file for PASE

* **type:** Data types used in PASE

* **utils:** Util functions used in PASE

We implemented index IVF_PQ in PASE and the code is in **postgresql-11.0/contrib/pase/ivfpq**.

# Prerequisite

OpenMP 4.0.1

# Getting the Source
`git clone https://github.com/Anonymous-Vec/Vec-Exp.git`

## How to use PASE

### Start PostgreSQL

#### Checking the Required Environment

`sudo apt-get install build-essential libreadline-dev zlib1g-dev flex bison libxml2-dev libxslt-dev libssl-dev libxml2-utils xsltproc ccache`

`wget https://ftp.postgresql.org/pub/source/v11.0/postgresql-11.0.tar.gz`

`tar -zxvf postgresql-11.0.tar.gz`

Code of PG11 here can not be used directly since some importance files are not uploaded by git.

`cd postgresql-11.0`

#### Configure

`mkdir build`

`./configure --prefix=$/absolute/path/build CFLAGS="-O3" LDFLAGS="-fPIC -fopenmp" `

#### Compile
`make`

`make install`


#### Initial new cluster named "data" on a folder:

`build/bin/initdb -D data`

#### Set the size of shared buffer in postgresql-11.0/build/data/postgresql.conf/:
`shared_buffers = 160GB`

### Start PASE

`cd contrib/pase`

#### May need to change the Makefile:
`PG_CONFIG=postgresql-11.0/build/bin/pg_config`
#### Compile the PASE
`make USE_PGXS=1`

`make install`
	
#### Start the cluster:
`cd ../..`

`build/bin/pg_ctl -D data start` 
#### Create a database named "pasetest"
`build/bin/createdb -p 5432 pasetest`
#### Connect the database
`build/bin/psql -p 5432 pasetest`

### EXAMPLE CODE Used in Psql Command Line
#### Create Extension

`create extension pase;`

#### Create Table

`CREATE TABLE vectors_ivfflat_test ( id serial, vector float4[]);`

```
INSERT INTO vectors_ivfflat_test SELECT id, ARRAY[id
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1,1,1,1,1,1
       ,1,1,1,1,1
       ]::float4[] FROM generate_series(1, 50000) id;
```

#### Build Index

```
CREATE INDEX v_ivfflat_idx ON vectors_ivfflat_test
       USING
         pase_ivfflat(vector)
  WITH
    (clustering_type = 1, distance_type = 0, dimension = 256, clustering_params = "10,100");
```

#### Search Index

```
SELECT vector <#> '31111,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1'::pase as distance
    FROM vectors_ivfflat_test
    ORDER BY
    vector <#> '31111,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1'::pase
     ASC LIMIT 10;
```


### Original PASE Code:

- [Pase: PostgreSQL Ultra-High Dimensional Approximate Nearest Neighbor Search Extension](https://github.com/alipay/PASE)

## How to use Faiss

### Prerequisite

C++11 compiler (with support for OpenMP support version 2 or higher)


BLAS implementation (we strongly recommend using Intel MKL for best performance).

`sudo apt install intel-mkl`

`sudo apt-get install -y libopenblas-dev` 


CMake minimum required(VERSION 3.17)


### Compile and Build:
`cd faiss`

`cmake -B build . -DFAISS_ENABLE_GPU=OFF -DFAISS_ENABLE_PYTHON=OFF -DCMAKE_BUILD_TYPE=Release -DFAISS_OPT_LEVEL=generic `

`make -C build -j faiss`  

`sudo make -C build install` 

`make -C build 2-IVFFlat`

### Run the Example Code:
`./build/tutorial/cpp/2-IVFFlat` 

### Original Faiss Code:

- [Faiss: A Library for Efficient Similarity Search and Clustering of Dense Vectors](https://github.com/facebookresearch/faiss)

# Comparison Results

## Evaluating Index Construction

### IVF_FLAT
![plot](results/ivfflat_build.png)
### IVF_PQ
![plot](results/ivfpq_build.png)
### HNSW
![plot](results/HNSW_build.png)


## Evaluating Index Size

### IVF_FLAT
![plot](results/ivfflat_indexSize.png)
### IVF_PQ
![plot](results/ivfpq_indexSize.png)
### HNSW
![plot](results/HNSW_indexSize.png)

## Evaluating Search Performance

### IVF_FLAT
![plot](results/ivfflat_SearchTime.png)
### IVF_PQ
![plot](results/ivfpq_SearchTime.png)
### HNSW
![plot](results/HNSW_SearchTime.png)


# Summary

This work investigates the performance difference between specialized vector databases and generalized vector databases. Based on the experimental results, there is still a significant performance gap between the two approaches. In particular, specialized vector databases (e.g., Faiss) can outperform generalized vector databases (e.g., PASE) by an order of magnitude or even more depending on different scenarios, indexes, and datasets.

We summarize the root causes of the performance gap as follows and discuss how to bridge the gap:

* **RC#1: Data Ingestion Model.**

* **RC#2: Tuple Accesses.**

* **RC#3: Parallel Execution.**

* **RC#4: Space Utilization.**

* **RC#5: Clustering Implementation.**

* **RC#6: SQL Evaluation Model.**

* **RC#7: Precomputed Table Implementation.**


Overall Message. The overall conclusion of this work is that, for
The overall conclusion of the work is that, with a careful implementation, it is feasible to support vector data management inside a relational database that achieves comparable performance to the state-of-the-art specialized vector database. We do not see a fundamental limitation in using a relational database to support efficient vector data management. In this way, we can use a single relational database to support more applications that involve tables and vectors. The paper lays out seven useful root causes with actionable guidelines to build a generalized vector database step by step to achieve both high performance and generality in the future.