# Developing Features daipe way

First you need to initialize the feature decorator for entity that you want,
e.g. "client".

```python
from pyspark.sql import types as t
from daipecore.decorator.DecoratedDecorator import DecoratedDecorator
from featurestorebundle.entity.Entity import Entity
from featurestorebundle.feature.FeaturesStorage import FeaturesStorage
from featurestorebundle.notebook.decorator.feature import feature

entity = Entity(
    name="client",
    id_column="UserName",
    id_column_type=t.StringType(),
    time_column="run_date",
    time_column_type=t.DateType(),
)

if not "client_feature" in globals():  # client_feature and features_storage are only initialized once
    features_storage = FeaturesStorage(entity)  # features_storage stores DataFrames from all feature notebooks

    @DecoratedDecorator
    class client_feature(feature):  # your custom decorator for a specific entity
        def __init__(self, *args, category=None):
            super().__init__(*args, entity=entity, category=category, features_storage=features_storage)
```

Now you can develop the features same way as you were used to,
except you wrap the code in daipe decorators. Note that features
defined in `client_feature` decorator must be mapped to the actual
features returned by transformation.

```python
from pyspark.sql import functions as f
from pyspark.sql import DataFrame
from datalakebundle.imports import transformation, read_table


@transformation(read_table("silver.tbl_loans"), display=True)
@client_feature(
    ("Age", "Client's age"),
    ("Gender", "Client's gender"),
    ("WorkExperience", "Client's work experience"),
    category="personal",
)
def client_personal_features(df: DataFrame):
    return (
        df.select("UserName", "Age", "Gender", "WorkExperience")
        .groupBy("UserName")
        .agg(
            f.max("Age").alias("Age"),
            f.first("Gender").alias("Gender"),
            f.first("WorkExperience").alias("WorkExperience"),
        )
        .withColumn("run_date", f.lit(today))
    )
```

Features can be written to Feature Store using Features Writer. Note that
`features_storage` was initialized in the first step. The write step is here
for completeness, but it is not recommended writing features in every notebook,
instead use write only in orchestration notebook (see section [orchestration](orchestration.md)).

```python
from datalakebundle.imports import notebook_function
from featurestorebundle.delta.DeltaWriter import DeltaWriter


@notebook_function()
def write_features(writer: DeltaWriter):
    writer.write_latest(features_storage)
```
