---
layout: default
title: SQL CLI
parent: SQL
nav_order: 5
---

# SQL CLI

SQL CLI is a stand-alone Python application that you can launch with the `odfesql` command.

Install the SQL plugin to your Elasticsearch instance, run the CLI using MacOS or Linux, and connect to any valid Elasticsearch end-point.

![SQL CLI](../../images/cli.gif)

## Features

SQL CLI has the following features:

- Multi-line input
- Autocomplete for SQL syntax and index names
- Syntax highlighting
- Formatted output:
  - Tabular format
  - Field names with color
  - Enabled horizontal display (by default) and vertical display when output is too wide for your terminal, for better visualization
  - Pagination for large output
- Works with or without security enabled
- Supports loading configuration files
- Supports all SQL plugin queries

## Install

Launch your local Elasticsearch instance and make sure you have the SQL plugin installed.

To install the SQL CLI:

1. We suggest you install and activate a python3 virtual environment to avoid changing your local environment:
```
pip install virtualenv
virtualenv venv
cd venv
source ./bin/activate
```

2. Install the CLI:
```
pip3 install odfe-sql-cli
```

The SQL CLI only works with Python 3.
{: .note }

3. To launch the CLI, run:
```
odfesql https://localhost:9200 --username admin --password admin
```
By default, the `odfesql` command connects to http://localhost:9200.

## Configure

When you first launch the SQL CLI, a configuration file is automatically created at `~/.config/odfesql-cli/config` (for MacOS and Linux), the configuration is auto-loaded thereafter.

You can configure the following connection properties:

- `endpoint`: You do not need to specify an option, anything that follows the launch command `odfesql` is considered as the endpoint. If you do not provide an endpoint, by default, the SQL CLI connects to http://localhost:9200.
- `-u/-w`: Supports username and password for HTTP basic authentication, such as with the security plugin or fine-grained access control for Amazon Elasticsearch Service.
- `--aws-auth`: Turns on AWS sigV4 authentication to connect to an Amazon Elasticsearch endpoint. Use with the AWS CLI (`aws configure`) to retrieve the local AWS configuration to authenticate and connect.

For a list of all available configurations, see [clirc](https://github.com/opendistro-for-elasticsearch/sql-cli/blob/master/src/conf/clirc).

## Using the CLI

1. Save the sample [accounts test data](https://github.com/opendistro-for-elasticsearch/sql/blob/master/src/test/resources/doctest/testdata/accounts.json) file.

1. Index the sample data.
```
curl -H "Content-Type: application/x-ndjson" -POST https://localhost:9200/data/_bulk -u admin:admin --insecure --data-binary "@accounts.json"
```

1. Run a sample SQL command:
```
SELECT * FROM accounts;
```

By default, you see a maximum output of 200 rows. To show more results, add a `LIMIT` clause with the desired value.

## Query options

Run a single query with the following options:

- `--help`: Help page for options
- `-q`: Follow by a single query
- `-f`: Specify JDBC or raw format output
- `-v`: Display data vertically
- `-e`: Translate SQL to DSL

## CLI options

- `-p`: Always use pager to display output
- `--clirc`: Provide path for the configuration file
