# Developing Features daipe way

First you need to define an entity, and it's `id column`, e.g. `client` with
primary key `client_id`.

```yaml
parameters:
  featurestorebundle:
    entities:
      client:
        id_column: "client_id"
        id_column_type: "string"
```

Then you can get the entity, and its feature decorator in notebook. 

```python
import daipe as dp

entity = dp.fs.get_entity()
feature = dp.fs.feature_decorator_factory.create(entity)
```

Now you can develop the features same way as you were used to,
except you wrap the code in daipe decorators. Note that features
defined in `feature` decorator must be mapped to the actual
features returned by transformation.

```python
import daipe as dp
from pyspark.sql import functions as f
from pyspark.sql import DataFrame


@dp.transformation(dp.read_table("silver.tbl_loans"), display=True)
@feature(
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
