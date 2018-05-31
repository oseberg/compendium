# Logstash

- [intro](https://www.elastic.co/guide/en/logstash/current/introduction.html)
- [running logstash](https://www.elastic.co/guide/en/logstash/current/setup-logstash.html)
- [running with docker](https://www.elastic.co/guide/en/logstash/current/docker.html) note, use the -oss image

## intro

logstash is a useful tool at Oseberg to take data from kafka, and then send it
to elasticsearch. We also use logstash for stateless kafka to kafka
transformations, as logstash has a kafka input and output plugin, and a simple
api called filter, for transforming events

#### plugins we use
here are the input and output plugin pages
- [input](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)
- [output](https://www.elastic.co/guide/en/logstash/current/ouput-plugins.html)

here are the ones we use mostly
- [kafka input](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-kafka.html)
- [kafka output](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-kafka.html)
- [elasticsearch output](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html)

### the filter component

a better name for this piece would be map, as we are usually changing events by
adding or modifying fields in the event, and rarely actually filter anything
out

- [filter](https://www.elastic.co/guide/en/logstash/current/core-operations.html)
- [filter plugins](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html)
- [ruby](https://www.elastic.co/guide/en/logstash/current/plugins-filters-ruby.html)

### the logstash.conf

In the end, we will deploy a logstash container with one logstash.conf that will
have the below structure

input {}
filter {}
output {}

One can have more than one input, and more than one output, but usually we are
doing 1:1

### kafka to kafka example

logstash.conf file:
```
input {
  kafka {
    client_id => 'logstash'
    group_id => 'logstash'
    codec => 'json'
    auto_offset_reset => 'earliest'
    bootstrap_servers => '<kafka server>'
    topics_pattern => '<testing>'
    max_partition_fetch_bytes => '5242880'
    metadata_max_age_ms => 1000
  }
}

filter {
  ruby {
    code => "
      require 'json'
      event.cancel
      event_type = event.get('[payload][op]')
      source = event.get('[payload][before][source]') || event.get('[payload][after][source]')
      natural_key = event.get('[payload][before][natural_key]') || event.get('[payload][after][natural_key]')

      new_event = LogStash::Event.new({
        'source' => source,
        'naturalKey' => natural_key,
      })
      if natural_key != nil
        if event_type == 'd'
          new_event.set('action', 'delete')
          new_event_block.call(new_event)
        else
          new_event.set('action', 'upsert')
          new_event.set('payload', JSON.parse(event.get('[payload][after][payload]')))
          new_event_block.call(new_event)
        end
      end
    "
  }
}

output {
  kafka {
    message_key => '%{action}'
    codec => 'json'
    topic_id => 'testing'
    bootstrap_servers => '<kafka server>'
  }
}

```

```
docker run --rm -it \
   -v "${PWD}/logstash.conf:/config-dir/logstash.conf" \
   docker.elastic.co/logstash/logstash-oss:6.2.2 \
   logstash -f /config-dir/logstash.conf
```

