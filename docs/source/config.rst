====================
Plugin Configuration
====================

Metlog provides some plugins to ease integration with logstash.

Input plugins provided:

    * logstash.inputs.zeromq_hs

Filter plugins provided:

    * logstash.filters.tagger

Output plugins provided:

    * logstash.outputs.metlog_file
    * logstash.outputs.metlog_statsd
    * logstash.outputs.simple_statsd

Input plugins
=============

zeromq_hs configuration
-----------------------

The zeromq_hs input plugin is provides a simple handshake service
which exposes a ZMQ::REP socket that is used to synchronize the
PUB/SUB 0mq input plugin.  This plugin is only required if Metlog has
declared a sender type of ZmqHandshakePubSender.

Using the zeromq_hs plugin requires setting a 0mq address so to bind a
socket.  All other required keys are inherited from the base input
plugin from logstash. ::

    input {
        zeromq_hs {
           type => "metlog"
           mode => "server"
           address => "tcp://*:5180"
        }
    }

In the above example, the `type` is required by logstash.inputs.base.
It is not used by the zeromq_hs plugin.

`mode` must always be set as 'server' for the socket to bind properly.
`address` is any valid URL recognized by 0mq.

Filter plugins
==============

tagger configuration
--------------------

The tagger filter lets you define a pattern keypath into an event.
Each keypath is applied in order.  On the first match - all tags will
be applied to the event.

Keypaths are defined using a '/' notation.

One common case is to match the `type` of an event so that events are
routed to a final destination.  In the following example, we want to
route all `timer` type events to the statsd output plugin by adding 
the tag 'output_statsd' to the event. ::

    filter {
        tagger {
            # all timer messages go to statsd
            type => "metlog"
            pattern => [ "fields/type", "timer"]
            add_tag => [ "output_statsd" ]
        }
    }

If a keypath does not exist within an event, it is ignored.

Multiple keypaths can be defined as shown in the following example.
If the file type is either a 'timer' or 'counter', the 'output_statsd'
tag will be applied. ::

    filter {
        tagger {
            # all timer messages go to statsd
            type => "metlog"
            pattern => [ "fields/type", "timer", "fields/type", "counter" ]
            add_tag => [ "output_statsd" ]
        }
    }
