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

(mmcloud_plugin_example)=
# Memory Machine Cloud

This example shows how to use the MMCloud plugin to execute tasks on MemVerge Memory Machine Cloud.

```{code-cell}
:lines_to_next_cell: 1

from flytekit import Resources, task, workflow
from flytekitplugins.mmcloud import MMCloudConfig
```

`MMCloudConfig` configures `MMCloudTask`. Tasks specified with `MMCloudConfig` will be executed using MMCloud. Tasks will be executed with requests `cpu="1"` and `mem="1Gi"` by default.

```{code-cell}
@task(task_config=MMCloudConfig())
def to_str(i: int) -> str:
    return str(i)


@task(task_config=MMCloudConfig())
def to_int(s: str) -> int:
    return int(s)
```

[Resource](https://docs.flyte.org/en/latest/user_guide/productionizing/customizing_task_resources.html) (cpu and mem) requests and limits, [container](https://docs.flyte.org/en/latest/user_guide/customizing_dependencies/multiple_images_in_a_workflow.html) images, and [environment](https://docs.flyte.org/en/latest/api/flytekit/generated/flytekit.task.html) variable specifications are supported.

```{code-cell}
@task(
    task_config=MMCloudConfig(submit_extra="--migratePolicy [enable=true]"),
    requests=Resources(cpu="1", mem="1Gi"),
    limits=Resources(cpu="2", mem="4Gi"),
    environment={"KEY": "value"},
)
def concatenate_str(s1: str, s2: str) -> str:
    return s1 + s2


@workflow
def concatenate_int_wf(i1: int, i2: int) -> int:
    i1_str = to_str(i=i1)
    i2_str = to_str(i=i2)
    return to_int(s=concatenate_str(s1=i1_str, s2=i2_str))
```
