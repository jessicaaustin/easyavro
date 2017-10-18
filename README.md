# EasyAvro  [![Build Status](https://travis-ci.org/axiom-data-science/easyavro.svg?branch=master)](https://travis-ci.org/axiom-data-science/easyavro)

A python helper for producing and consuming `avro` schema'd Kafka topics. Simplicity and the ability to execute a function for each message consumed is the top priority. This is not designed for high throughput.


## Installation

```bash
conda install -c axiom-data-science EasyAvro
```

There is py PyPi package. I know, I know. Feel free to submit a PR to integrate it into Travis.


## Usage

#### Producer

The schemas `my-topic-key` and `my-topic-value` must be available in the schema registry.

```python
from easyavro import EasyAvroProducer

bp = EasyAvroProducer(
    schema_registry_url='http://localhost:50002',
    kafka_brokers=['localhost:50001'],
    kafka_topic='my-topic'
)

# Records are (key, value) tuples
records = [
    ('foo', 'foo'),
    ('bar', 'bar'),
]
bp.produce(records)
```

Or pass in your own schemas.

```python
from easyavro import EasyAvroProducer

bp = EasyAvroProducer(
    schema_registry_url='http://localhost:50002',
    kafka_brokers=['localhost:50001'],
    kafka_topic='my-topic',
    value_schema=SomeAvroSchemaObject,
    key_schema=SomeAvroSchemaObject,
)

# Records are (key, value) tuples
records = [
    ('foo', 'foo'),
    ('bar', 'bar'),
]
bp.produce(records)
```

If you don't have a key schema, just pass anything other than `None` to the
constructor and use `None` as the value of the key.

```python
from easyavro import EasyAvroProducer

bp = EasyAvroProducer(
    schema_registry_url='http://localhost:50002',
    kafka_brokers=['localhost:50001'],
    kafka_topic='my-topic',
    value_schema=SomeAvroSchemaObject,
    key_schema='not_used_because_no_keys_in_the_records',
)

# Records are (key, value) tuples
records = [
    (None, 'foo'),
    (None, 'bar'),
]
bp.produce(records)
```


#### Consumer

The defaults are sane. They will pull offsets from the broker and set the topic offset to `largest`. This will pull all new messages that haven't been acknowledged by a consumer with the same `consumer_group` (which translates to the `librdkafka` `group.id` setting).

```python
from easyavro import EasyAvroConsumer

def on_recieve(key: str, value: str) -> None:
    print("Got Key:{}\nValue:{}\n".format(key, value))

bc = EasyAvroConsumer(
    schema_registry_url='http://localhost:50002',
    kafka_brokers=['localhost:50001'],
    consumer_group='easyavro.testing',
    kafka_topic='my-topic'
)
bc.consume(on_recieve=on_recieve)
```

Or pass in your own [topic config](https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md#topic-configuration-properties) dict.

```python
from easyavro import EasyAvroConsumer

def on_recieve(key: str, value: str) -> None:
    print("Got Key:{}\nValue:{}\n".format(key, value))

bc = EasyAvroConsumer(
    schema_registry_url='http://localhost:50002',
    kafka_brokers=['localhost:50001'],
    consumer_group='easyavro.testing',
    kafka_topic='my-topic',
    topic_config={'enable.auto.commit': False, 'offset.store.method': 'file'}
)
bc.consume(on_recieve=on_recieve)
```

Or pass in a value to use for the `auto.offset.reset` [topic config](https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md#topic-configuration-properties) setting.

```python
from easyavro import EasyAvroConsumer

def on_recieve(key: str, value: str) -> None:
    print("Got Key:{}\nValue:{}\n".format(key, value))

bc = EasyAvroConsumer(
    schema_registry_url='http://localhost:50002',
    kafka_brokers=['localhost:50001'],
    consumer_group='easyavro.testing',
    kafka_topic='my-topic',
    offset='earliest'
)
bc.consume(on_recieve=on_recieve)
```

## Testing

There are only integration tests.

**Start a confluent kafka ecosystem**

```
docker run -d --net=host \
        -e ZK_PORT=50000 \
        -e BROKER_PORT=50001 \
        -e REGISTRY_PORT=50002 \
        -e REST_PORT=50003 \
        -e CONNECT_PORT=50004 \
        -e WEB_PORT=50005 \
        -e RUNTESTS=0 \
        -e DISABLE=elastic,hbase \
        -e DISABLE_JMX=1 \
        -e RUNTESTS=0 \
        -e FORWARDLOGS=0 \
        -e SAMPLEDATA=0 \
        --name easyavro-testing \
      landoop/fast-data-dev
```

#### Docker

```
docker build -t easyavro .
docker run --net="host" easyavro
```

#### No Docker

```
py.test -s -rxs -v
```
