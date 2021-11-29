# Reading Features

Accessing Feature Store is pretty straightforward.

```python
from datalakebundle.imports import transformation
from featurestorebundle.feature.FeatureStore import FeatureStore


@transformation(display=True)
def load_feature_store(feature_store: FeatureStore):
    return feature_store.get_latest("client")
```
