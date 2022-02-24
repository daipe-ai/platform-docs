# Features orchestration

Writing features in every notebook is not recommended, because writing the features
is expensive operation (delta merge), also if you try to run those notebooks in
parallel its most likely you will get write conflict error.

Feature Store address this problem with its own orchestration of feature
notebooks.

You can define which feature notebooks to orchestrate in config.

```yaml
parameters:
  featurestorebundle:
    orchestration:
      num_parallel: 8
      stages:
        stage1:
          - '%databricksbundle.project_root.repo.path%/src/myproject/feature_store/features_ntb1'
          - '%databricksbundle.project_root.repo.path%/src/myproject/feature_store/features_ntb2'
        stage2:
          - '%databricksbundle.project_root.repo.path%/src/myproject/feature_store/features_ntb3'
```

This configuration means that you want to first run `features_ntb1` and
`features_ntb2`, write results to the feature store and then run
`features_ntb3`. So stages are executed sequentially to cover computational
dependency use cases. You can use `num_parallel` to control how many notebooks
will run in parallel in each stage.

You can now use orchestration notebook to run the orchestration.

```python
%run ./app/bootstrap
```

```python
import daipe as dp
from featurestorebundle.orchestration.DatabricksOrchestrator import DatabricksOrchestrator
```

```python
@dp.notebook_function()
def checkpointing_setup(spark: SparkSession):
    spark.sparkContext.setCheckpointDir("dbfs:/tmp/checkpoints")
```

```python
@dp.notebook_function()
def orchestrate(orchestrator: DatabricksOrchestrator):
    orchestrator.orchestrate()
```
