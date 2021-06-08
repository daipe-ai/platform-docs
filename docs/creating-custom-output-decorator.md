# Creating a custom output decorator

We are going to create a `@send_to_api` decorator which appends DataFrame to a table while simultaneously sends the data to an API.

Inside our project's `src/__project_name__/lib` we create a file called `send_to_api.py`

We must adhere to this interface. 

```python
@DecoratedDecorator
class send_to_api(OutputDecorator):
    def __init__(self, *args, **kwargs):
        # init
        
        def process_result(self, result: DataFrame, container: ContainerInterface):
            # the decorator logic
```

The input arguments of the class are completely arbitrary. We are using an `url` variable to specify where to send the data.

The `process_result()` function has a fixed interface. It recieves the `result` DataFrame and a `container` with Daipe services and configuration.

All the objects necessary to process the DataFrame can be obtained from the container e. g. Logger.

We define a custom method for sending data to an API.

```python
def __send_to_api(self, df):
    df_json = df.toPandas().to_json()
    conn = http.client.HTTPSConnection(self.__url)
    conn.request("POST", "/", df_json, {'Content-Type': 'application/json'})
```

The complete output decorator class can look like this

```python
from logging import Logger

import http.client
from daipecore.decorator.DecoratedDecorator import DecoratedDecorator
from daipecore.decorator.OutputDecorator import OutputDecorator
from injecta.container.ContainerInterface import ContainerInterface
from pyspark.sql import DataFrame


@DecoratedDecorator
class send_to_api(OutputDecorator):
    def __init__(self, url: str):
        self.__url = url

    def process_result(self, result: DataFrame, container: ContainerInterface):
        logger: Logger = container.get("datalakebundle.logger")
        logger.info(f"Sending {result.count()} records to API")

        self.__send_to_api(result)

    def __send_to_api(self, df):
        df_json = df.toPandas().to_json()
        conn = http.client.HTTPSConnection(self.__url)
        conn.request("POST", "/", df_json, {'Content-Type': 'application/json'})

```

In our project we simple import the decorator using

```python
from __myproject__.lib.send_to_api import send_to_api
```
