# Features orchestration

Writing features in every notebook is not recommended, because writing the features
is expensive operation (delta merge), also if you try to run those notebooks in
parallel its most likely you will get write conflict error.

Because of that it's considered as a best practice to create features orchestration
notebook and write all features at once.

```python
%run ./app/install_master_package
```

```python
%run ./client_feature_writer_init
```

```python
from datalakebundle.imports import notebook_function
from featurestorebundle.delta.DeltaWriter import DeltaWriter
```
```python
%run ./features_notebook1
```

```python
%run ./features_notebook2
```

```python
# write all features at once
@notebook_function()
def write_features(writer: DeltaWriter):
    writer.write_latest(features_storage)
```
