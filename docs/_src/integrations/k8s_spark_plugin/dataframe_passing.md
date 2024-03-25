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

(intermediate_spark_dataframes_passing)=

# Converting a Spark DataFrame to a Pandas DataFrame

This example shows the process of returning a Spark dataset from a Flyte task
and then utilizing it as a Pandas DataFrame.

+++ {"lines_to_next_cell": 0}

To begin, import the libraries.

```{code-cell}
import flytekit
import pandas
from flytekit import ImageSpec, Resources, kwtypes, task, workflow
from flytekit.types.structured.structured_dataset import StructuredDataset
from flytekitplugins.spark import Spark

try:
    from typing import Annotated
except ImportError:
    from typing_extensions import Annotated
```

+++ {"lines_to_next_cell": 0}

Create an `ImageSpec` to automate the retrieval of a prebuilt Spark image.

```{code-cell}
custom_image = ImageSpec(name="flyte-spark-plugin", registry="ghcr.io/flyteorg")
```

+++ {"lines_to_next_cell": 0}

:::{important}
Replace `ghcr.io/flyteorg` with a container registry you've access to publish to.
To upload the image to the local registry in the demo cluster, indicate the registry as `localhost:30000`.
:::

In this particular example,
we specify two column types: `name: str` and `age: int`
that we extract from the Spark DataFrame.

```{code-cell}
columns = kwtypes(name=str, age=int)
```

+++ {"lines_to_next_cell": 0}

To create a Spark task, add {py:class}`~flytekitplugins.spark.Spark` config to the Flyte task.

The `spark_conf` parameter can encompass configuration choices commonly employed when setting up a Spark cluster.
Additionally, if necessary, you can provide `hadoop_conf` as an input.

Create a task that yields a Spark DataFrame.

```{code-cell}
:lines_to_next_cell: 2

@task(
    task_config=Spark(
        spark_conf={
            "spark.driver.memory": "1000M",
            "spark.executor.memory": "1000M",
            "spark.executor.cores": "1",
            "spark.executor.instances": "2",
            "spark.driver.cores": "1",
        }
    ),
    limits=Resources(mem="2000M"),
    container_image=custom_image,
)
def spark_df() -> Annotated[StructuredDataset, columns]:
    """
    This task returns a Spark dataset that conforms to the defined schema.
    """
    sess = flytekit.current_context().spark_session
    return StructuredDataset(
        dataframe=sess.createDataFrame(
            [
                ("Alice", 5),
                ("Bob", 10),
                ("Charlie", 15),
            ],
            ["name", "age"],
        )
    )
```

+++ {"lines_to_next_cell": 2}

`spark_df` represents a Spark task executed within a Spark context, leveraging an active Spark cluster.

This task yields a `pyspark.DataFrame` object, even though the return type is specified as
{ref}`StructuredDataset <structured_dataset>`.
The Flytekit type system handles the automatic conversion of the `pyspark.DataFrame` into a `StructuredDataset` object.
The `StructuredDataset` object serves as an abstract representation of a DataFrame, adaptable to various DataFrame formats.

+++ {"lines_to_next_cell": 0}

Create a task to consume the Spark DataFrame.

```{code-cell}
:lines_to_next_cell: 2

@task
def sum_of_all_ages(sd: Annotated[StructuredDataset, columns]) -> int:
    df: pandas.DataFrame = sd.open(pandas.DataFrame).all()
    return int(df["age"].sum())
```

+++ {"lines_to_next_cell": 2}

The `sum_of_all_ages` task accepts a parameter of type `StructuredDataset`.
By utilizing the `open` method, you can designate the DataFrame format, which, in our scenario, is `pandas.DataFrame`.
When `all` is invoked on the structured dataset, the executor will load the data into memory (or download it if executed remotely).

+++ {"lines_to_next_cell": 0}

Lastly, define a workflow.

```{code-cell}
@workflow
def spark_to_pandas_wf() -> int:
    df = spark_df()
    return sum_of_all_ages(sd=df)
```

+++ {"lines_to_next_cell": 0}

You can execute the code locally.

```{code-cell}
if __name__ == "__main__":
    print(f"Running {__file__} main...")
    print(f"Running my_smart_schema()-> {spark_to_pandas_wf()}")
```

New DataFrames can be dynamically loaded through the type engine.
To register a custom DataFrame type, you can define an encoder and decoder for `StructuredDataset` as outlined in the {ref}`structured_dataset` example.

Existing DataFrame plugins include:

- {ref}`Modin <Modin>`
- [Vaex](https://github.com/flyteorg/flytekit/blob/master/plugins/flytekit-vaex/README.md)
- [Polars](https://github.com/flyteorg/flytekit/blob/master/plugins/flytekit-polars/README.md)
