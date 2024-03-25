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

(spark_on_databricks_agent)=

# Running Spark on Databricks

To begin, import the required dependencies.

```{code-cell}
import datetime
import random
from operator import add

import flytekit
from flytekit import Resources, task, workflow
from flytekitplugins.spark import Databricks
```

+++ {"lines_to_next_cell": 0}

To run a Spark job on the Databricks platform, simply include Databricks configuration in the task config.
The Databricks config is the same as the Databricks job request. For more details, please refer to the
[Databricks job request](https://docs.databricks.com/dev-tools/api/2.0/jobs.html#request-structure) documentation.

```{code-cell}
@task(
    task_config=Databricks(
        spark_conf={
            "spark.driver.memory": "1000M",
            "spark.executor.memory": "1000M",
            "spark.executor.cores": "1",
            "spark.executor.instances": "2",
            "spark.driver.cores": "1",
        },
        databricks_conf={
            "run_name": "flytekit databricks plugin example",
            "new_cluster": {
                "spark_version": "11.0.x-scala2.12",
                "node_type_id": "r3.xlarge",
                "aws_attributes": {
                    "availability": "ON_DEMAND",
                    "instance_profile_arn": "arn:aws:iam::<AWS_ACCOUNT_ID_DATABRICKS>:instance-profile/databricks-flyte-integration",
                },
                "num_workers": 4,
            },
            "timeout_seconds": 3600,
            "max_retries": 1,
        },
    ),
    limits=Resources(mem="2000M"),
    cache_version="1",
)
def hello_spark(partitions: int) -> float:
    print(f"Starting Spark with {partitions} partitions.")
    n = 100000 * partitions
    sess = flytekit.current_context().spark_session
    count = sess.sparkContext.parallelize(range(1, n + 1), partitions).map(f).reduce(add)
    pi_val = 4.0 * count / n
    print("Pi val is :{}".format(pi_val))
    return pi_val
```

+++ {"lines_to_next_cell": 0}

For this particular example,
we define a function that executes the map-reduce operation within the Spark cluster.

```{code-cell}
def f(_):
    x = random.random() * 2 - 1
    y = random.random() * 2 - 1
    return 1 if x**2 + y**2 <= 1 else 0
```

+++ {"lines_to_next_cell": 0}

Additionally, we define a standard Flyte task that won't be executed on the Spark cluster.

```{code-cell}
@task(cache_version="1")
def print_every_time(value_to_print: float, date_triggered: datetime.datetime) -> int:
    print("My printed value: {} @ {}".format(value_to_print, date_triggered))
    return 1
```

+++ {"lines_to_next_cell": 0}

Finally, define a workflow that connects your tasks in a sequence.
Remember, Spark and non-Spark tasks can be chained together as long as their parameter specifications match.

```{code-cell}
@workflow
def my_databricks_job(triggered_date: datetime.datetime = datetime.datetime.now()) -> float:
    pi = hello_spark(partitions=1)
    print_every_time(value_to_print=pi, date_triggered=triggered_date)
    return pi
```

+++ {"lines_to_next_cell": 0}

You can execute the workflow locally.

```{code-cell}
if __name__ == "__main__":
    print(f"Running {__file__} main...")
    print(
        f"Running my_databricks_job(triggered_date=datetime.datetime.now()) {my_databricks_job(triggered_date=datetime.datetime.now())}"
    )
```
