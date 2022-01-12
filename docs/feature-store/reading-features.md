# Reading Features

Accessing Feature Store is pretty straightforward.

```python
import daipe as dp
from featurestorebundle.feature.FeatureStore import FeatureStore


@dp.transformation(display=True)
def load_feature_store(feature_store: FeatureStore):
    return feature_store.get_latest("client")
```
