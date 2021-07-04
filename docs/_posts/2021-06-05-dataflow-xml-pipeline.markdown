---
layout: post
title:  "GCP Dataflow pipeline for writing XMLs into BigQuery"
categories: gcp gcs dataflow apache beam storage bigquery terraform
render_with_liquid: false
---

The source code that can be found on on my [GitHub Repo](https://github.com/almercolic/dataflow-xml-gcs-to-bq-pipeline) shows an example Apache Beam (Google Dataflow) pipeline that reads XML files from a GCS bucket and writes them into a BigQuery Table.

The demo processes two example `Company` objects, it parses the XML files and maps them into `TableRow` objects according a output schema.
A schema can be already attached to the BigQuery table itself or provided via the insert job. in this case it is defined directly in my Java code:
```
TableSchema schema =
    new TableSchema()
        .setFields(
            Arrays.asList(
                new TableFieldSchema()
                    .setName("name")
                    .setType("STRING"),
                new TableFieldSchema()
                    .setName("phone_numbers")
                    .setType("STRUCT")
                    .setMode("REPEATED")
                    .setFields(
                        Collections.singletonList(
                           new TableFieldSchema().setName("phone_number").setType("STRING"))),
                new TableFieldSchema()
                    .setName("address")
                    .setType("STRUCT")
                    .setMode("REPEATED")
                    .setFields(
                        Arrays.asList(
                            new TableFieldSchema().setName("street").setType("STRING"),
                            new TableFieldSchema().setName("house_number").setType("STRING"),
                            new TableFieldSchema().setName("city").setType("STRING"),
                            new TableFieldSchema().setName("zip_code").setType("STRING")))));
```

In the pipeline definition a provided parameter `BigQueryIO.Write.WriteDisposition.WRITE_TRUNCATE` leads to a truncation of the BigQuery table on every pipeline run.

In order to execute it, please follow the instructions in the repos' readme. It guides how to setup the GCP infrastructure via `terraform` and also how to execute the mvn command.