# adapter-python-library
The adapter-python-library streamlines the development process for ClearBlade Adapters written in Python. Most of the interaction with the ClearBlade Edge/Platform has been abstracted out, including Authentication, Adapter Configuration Collection, and MQTT Connection.

## Usage
```python
import time
import json
from clearblade_adapter_library import AdapterLibrary

ADAPTER_NAME = "my-new-adapter"

def main():
    # Initialize the adapter library with the adapter name
    adapter = AdapterLibrary(ADAPTER_NAME)

    # Parse command line arguments and environment variables
    adapter.parse_arguments()

    # Initialize all things ClearBlade, includes authenticating if needed, and fetching the
    # relevant adapter_config collection entry
    adapter_config = adapter.initialize_clearblade()

    # if your adapter config includes custom adapter settings, parse/validate them here

    # Connect MQTT, if your adapter needs to subscribe to a topic, provide it as the first
    # parameter, and a callback for when messages are received. If no need to subscribe,
    # simply provide an empty string and None
    # connect_MQTT will establish the MQTT connection
    # Topic structure is entirely up to your adapter's requirements
    adapter.connect_MQTT(topic=adapter_config["topic_root"] + "/commands", cb_message_handler=cb_message_handler)

    # kick off adapter specific things here

    # keep adapter executing indefinitely
    while True:
        time.sleep(1)

def cb_message_handler(client, message):
    # process incoming MQTT messages as needed here
    payload = json.loads(message.payload.decode("utf-8"))
    print("Message Received: ", str(payload))

if __name__ == "__main__":
    main()
```

## Command Line Arguments & Environment Variables
All ClearBlade Adapters require a certain set of System specific variables to start and connect with the ClearBlade Platform/Edge. This library allows these to be passed in either by command line arguments, or environment variables. Note that command line arguments take precedence over environment variables.

| Name | CLI Flag | Environment Variable | Default |
| --- | --- | --- | --- |
| System Key | `--systemKey` | `CB_SYSTEM_KEY` | N/A |
| System Secret (**deprecated**) | `--systemSecret` | `CB_SYSTEM_SECRET` | N/A |
| Platform/Edge URL | `--platformURL` | N/A | `http://localhost:9000` |
| Platform/Edge Messaging URL | `--messagingURL` | N/A | `localhost:1883` |
| Device Name (**depreciated**) | `--deviceName` | N/A | `adapter_name` provided when initializing `AdapterLibrary` |
| Device Password/Active Key (**depreciated)** | `--password` | N/A | N/A |
| Device Service Account | N/A | `CB_SERVICE_ACCOUNT` | N/A |
| Device Service Account Token | N/A | `CB_SERVICE_ACCOUNT_TOKEN` | N/A |
| Log Level | `--logLevel` | N/A | `info` |
| Adapter Config Collection Name | `--adapterConfigCollection` | N/A | `adapter_config` |

A System Key will always be required to start the adapter, and it's recommended to always use a Device Service Account & Token for Adapters.
System Secret is **depreciated** and only provided for backwards compatibility. It should not be used for any new adapters. If provided its value will be ignored.

Device Name and Password for Adapters are **depreciated** and only provided for backwards compatibility and should not be used for any new adapters.

## Adapter Configuration & Settings
This library does require the use of an Adapter Configuration Collection. This allows you to easily provide configuration options to your adapter at runtime via this Collection, rather than command line arguments, environment variables, or hardcoded values in your adapter.

If your adapter does not have any specific settings it is still expected to have an entry in this collection, but adapter settings column can be left blank.

The default name for this Collection is `adapter_config`, and it's expected data structure is:

| Column Name | Column Type |
| --- | --- |
| `adapter_name` | `string` |
| `topic_root` | `string` |
| `adapter_settings` | `string` |

## Logging
This adapter introduces basic logging levels that your adapter can leverage. The `--logLevel` flag controls the logging output of the adapter.

The supported log levels are `debug`, `info`, `warn`, `error`, and `fatal`.

## MQTT Publishing
To publish MQTT messages from your adapter, use the `publish` method:

```python
adapter.publish("topic/name", "message payload")
```

## Fatal Errors
There are a few cases where this library will encounter a fatal error and cause your adapter to quit. A log with more information will be provided, but as a heads up these are the current cases that will cause the adapter to exit from within the library:

1. If required arguments (System Key, authentication credentials etc.) are not provided
2. If you are using a **depreciated** device name and password for authentication and MQTT disconnects with certain error codes
3. If your adapter subscribes to a topic and the device account used by the adapter does not have subscribe permissions on this topic (a warning will be logged)
