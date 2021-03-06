# Logstash input plugin for data from Event Hubs 

## Summary
This plugin reads data from specified Azure Event Hubs.

## Installation
You can install this plugin using the Logstash "plugin" or "logstash-plugin" (for newer versions of Logstash) command:
```sh
logstash-plugin install logstash-input-azureeventhub
```
For more information, see Logstash reference [Working with plugins](https://www.elastic.co/guide/en/logstash/current/working-with-plugins.html).

## Configuration
### Required Parameters
__*key*__

The shared access key to the target event hub.

__*username*__

The name of the shared access policy.

__*namespace*__

Event Hub namespace.

__*eventhub*__

Event Hub name.

__*partitions*__

Partition count of the target event hub.

### Optional Parameters
__*domain*__

Domain of the target Event Hub. Default value is "servicebus.windows.net".

__*port*__

Port of the target Event Hub. Default value is 5671.

__*receive_credits*__

The credit number to limit the number of messages to receive in a processing cycle. Value must be between 10 and 999. Default is 999.

__*consumer_group*__

Name of the consumer group. Default value is "$default".

__*time_since_epoch_millis*__

Specifies the point of time after which the messages are received. Default value is the time when this plugin is initialized:
```ruby
Time.now.utc.to_i * 1000
```
__*thread_wait_sec*__

Specifies the time (in seconds) to wait before another try if no message was received.

__*partition_list*__

Specifies a partition array subset to fetch events from. Default value is range from 0 to the configured partition count.
```ruby
(0..(@partitions-1))
```

__*partition_receiver_epochs*__

A map from partition (string) to epoch (integer). By default each partition doesn't have an epoch defined. For more information read https://blogs.msdn.microsoft.com/gyan/2014/09/02/event-hubs-receiver-epoch/ .

__*decorate_events*__

Adds eventhub metadata to event to event. Default value is false. Example decoration:
```ruby
{
    "@metadata" => {
        "azure_eventhub" => {
            "partition_id" => "4",
            "enqueued_time" => "2018-05-10T05:27:25.196Z",
            "properties" => "{x-opt-sequence-number=2939844, x-opt-offset=627091016784, x-opt-partition-key=some_partition_key, x-opt-enqueued-time=Thu May 10 05:27:25 UTC 2018}"
        }
    }
}
```

### Examples
* Bare-bone settings
```
input
{
    azureeventhub
    {
        key => "VGhpcyBpcyBhIGZha2Uga2V5Lg=="
        username => "receivepolicy"
        namespace => "mysbns"
        eventhub => "myeventhub"
        partitions => 4
        partition_receiver_epochs => { '2' => 42 '0' => 15 }
    }
}
```

* Example for WAD (Azure Diagnostics)
```
input
{
    azureeventhub
    {
        key => "VGhpcyBpcyBhIGZha2Uga2V5Lg=="
        username => "receivepolicy"
        namespace => "mysbns"
        eventhub => "myeventhub"
        partitions => 4
        partition_receiver_epochs => { '2' => 42 '0' => 15 }
    }
}
filter {
    split {field => 'records'} #split the records array in individual events
}
output {
    stdout { 
        codec => rubydebug
    }
}
```

## More information
The source code of this plugin is hosted in GitHub repo [Microsoft Azure Diagnostics with ELK](https://github.com/Azure/azure-diagnostics-tools). We welcome you to provide feedback and/or contribute to the project.

Please also see [Analyze Diagnostics Data with ELK template](https://github.com/Azure/azure-quickstart-templates/tree/master/diagnostics-with-elk) for quick deployment of ELK to Azure.   
