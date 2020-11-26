---
layout: post
title: connecting to postgres with standard sql package in golang
description: Postgres package pq in golang connection with certificate
author: bitsmonkey
tags: golang, database, certificate
---

![postgres-golang](/img/golang-postgres.jpg){:.coverimage}

Quick steps connecting to postgres DB using the stand library with postgres driver,

Import the packages

```go
import (
    "database/sql"
    _ "github.com/lib/pq" //postgres drivers for initialization
)
```

A simple get method to run a select statement. In example we will the connection string from the environment variables.

```go
var connStr = os.Getenv("CONNSTR")

//GetSomeDummyData ...
func (r *Repo) GetSomeDummyData() {

	db, _ := sql.Open("postgres", connStr)
	defer db.Close()

	id := "34940574"
	row := db.QueryRow("SELECT column1, column2 FROM dummytable where id=$1", id)

	rows, err := db.Query(query, qteNum, limit)
	defer rows.Close()
	if err != nil {
		log.Println(err)
	}

	var result []*model.DummyData

	for rows.Next() {
		d := &model.DummyData{}
		err = rows.Scan(&d.Prop1, &d.Prop2)

		if err != nil {
			log.Println(err)
			return nil, err
		}
		result = append(result, q)
	}
	return result, nil

}
```

Now the connection string looks something like this in the environment variable

```bash
EXPORT CONNSTR="user=username password=password host=dbname.host.com port=5433 dbname=dbname connect_timeout=20 sslmode=disable"
```

Lets now enable ssl mode with the ca cert in your project. In which case we need to set the path of the PEM file. Can be relative path of the absolute path. Postgres expects this in a [environment variable](https://www.postgresql.org/docs/9.3/libpq-envars.html) PGSSLROOTCERT=pathof.pem and connection string will specify `sslmode=verify-full`

```bash
EXPORT PGSSLROOTCERT=pathof.pem
EXPORT CONNSTR="user=username password=password host=dbname.host.com port=5433 dbname=dbname connect_timeout=20 sslmode=verify-full"
```

Code On!

Photo by [Ron Whitaker](https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText)