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

(airflow_agent_example_usage)=
# Airflow agent example usage
[Apache Airflow](https://airflow.apache.org) is a widely used open source
platform for managing workflows with a robust ecosystem. Flyte provides an
Airflow plugin that allows you to run Airflow tasks as Flyte tasks.
This allows you to use the Airflow plugin ecosystem in conjunction with
Flyte's powerful task execution and orchestration capabilities.

```{code-cell}

from airflow.operators.bash import BashOperator
from airflow.sensors.filesystem import FileSensor
from flytekit import task, workflow


@task()
def t1():
    print("success")
```

+++ {"lines_to_next_cell": 0}

Use the Airflow `FileSensor` to wait for a file to appear before running the task.

```{code-cell}
@workflow
def file_sensor():
    sensor = FileSensor(task_id="id", filepath="/tmp/1234")
    sensor >> t1()
```

+++ {"lines_to_next_cell": 0}

Use the Airflow `BashOperator` to run a bash command.

```{code-cell}
@workflow
def bash_sensor():
    op = BashOperator(task_id="airflow_bash_operator", bash_command="echo hello")
    op >> t1()
```
