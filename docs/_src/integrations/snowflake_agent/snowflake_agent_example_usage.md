---
jupytext:
  cell_metadata_filter: all
  formats: md:myst
  main_language: python
  notebook_metadata_filter: all
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.1
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

+++ {"lines_to_next_cell": 0}

(snowflake_agent_example_usage)=
# Querying data in Snowflake

This example shows how to use the `SnowflakeTask` to execute a query in Snowflake.

To begin, import the required libraries.

```{code-cell}
from flytekit import kwtypes, workflow
from flytekitplugins.snowflake import SnowflakeConfig, SnowflakeTask
```

+++ {"lines_to_next_cell": 0}

Instantiate a {py:class}`~flytekitplugins.snowflake.SnowflakeTask` to execute a query.
Incorporate {py:class}`~flytekitplugins.snowflake.SnowflakeConfig` within the task to define the appropriate configuration.

```{code-cell}
snowflake_task_no_io = SnowflakeTask(
    name="sql.snowflake.no_io",
    inputs={},
    query_template="SELECT 1",
    output_schema_type=None,
    task_config=SnowflakeConfig(
        account="<SNOWFLAKE_ACCOUNT_ID>",
        database="SNOWFLAKE_SAMPLE_DATA",
        schema="TPCH_SF1000",
        warehouse="COMPUTE_WH",
    ),
)
```

+++ {"lines_to_next_cell": 0}

:::{note}
For successful registration, ensure that your Snowflake task is assigned a unique
name within your project/domain for your Flyte installation.
:::

In practical applications, our primary focus is often on utilizing Snowflake to query datasets.
Here, we employ the `SNOWFLAKE_SAMPLE_DATA`, a default dataset in the Snowflake service.
You can find more details about it [here](https://docs.snowflake.com/en/user-guide/sample-data.html).
The data adheres to the following schema:

```{eval-rst}
+----------------------------------------------+
| C_CUSTKEY (int)                              |
+----------------------------------------------+
| C_NAME (string)                              |
+----------------------------------------------+
| C_ADDRESS (string)                           |
+----------------------------------------------+
| C_NATIONKEY (int)                            |
+----------------------------------------------+
| C_PHONE (string)                             |
+----------------------------------------------+
| C_ACCTBAL (float)                            |
+----------------------------------------------+
| C_MKTSEGMENT (string)                        |
+----------------------------------------------+
| C_COMMENT (string)                           |
+----------------------------------------------+
```

Let us explore how we can parameterize our query to filter results for a specific country.
This country will be provided as user input, using a nation key to specify it.

```{code-cell}
snowflake_task_templatized_query = SnowflakeTask(
    name="sql.snowflake.w_io",
    # Define inputs as well as their types that can be used to customize the query.
    inputs=kwtypes(nation_key=int),
    task_config=SnowflakeConfig(
        account="<SNOWFLAKE_ACCOUNT_ID>",
        database="SNOWFLAKE_SAMPLE_DATA",
        schema="TPCH_SF1000",
        warehouse="COMPUTE_WH",
    ),
    query_template="SELECT * from CUSTOMER where C_NATIONKEY =  {{ .inputs.nation_key }} limit 100",
)


@workflow
def snowflake_wf(nation_key: int):
    return snowflake_task_templatized_query(nation_key=nation_key)
```

+++ {"lines_to_next_cell": 0}

To review the query results, access the Snowflake console at:
`https://<SNOWFLAKE_ACCOUNT_ID>.snowflakecomputing.com/console#/monitoring/queries/detail`.

You can also execute the task and workflow locally.

```{code-cell}
if __name__ == "__main__":
    print(snowflake_task_no_io())
    print(snowflake_wf(nation_key=10))
```
