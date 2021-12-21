# voicegateway-load-tests
load tests for the cognigy voice gateway

## prerequisites
These load tests are generated using [SIPp](http://sipp.sourceforge.net/doc/), so the server acting as the load test generator must include SIPp.  Please refer to [this ansible role](https://github.com/davehorton/ansible-role-sipp) for details on building SIPp from scratch.  

## scenarios
Currently, there are two load test scenarios:

1. an inbound call that is answered by jambonz, lasts for 60 seconds, and is then hung up by the load generator.
1. an inbound call which is expected to result in an outbound (hairpinned) call back to the load generator.

### Running the inbound call scenario
You will need to be logged in as root, or execute the commands below with sudo priviledges.  For details on the various command line arguments available to SIPp please [refer here](http://sipp.sourceforge.net/doc/reference.html#Online+help+%28-h%29)

```bash
sipp -sf uac_pcap_60s_call.xml -r 10 -l 500 -m 300  -s 16173333456 18.185.52.215
```

The command above will generate calls towards the system at IP 18.185.52.215.  The various parameters have the following meaning:
- `-sf <scenario>`: the name of the scenario file
- `-r`: rate at which to generate calls, in calls per second
- `-l`: limit of concurrent calls; once the number of calls in progress hits this limit the sending rate will be throttled to keep it at or below this limit
- `-m`: total number of calls to send
- `'s`: the dialed number to put in the SIP INVITE

*Note:* You must configure the jambonz system under test with the phone number used in the `-s` parameter, and that phone number should route to an appropriate jambonz application that you want to load test.

### Running the hairpinned call scenario
You will need to open two terminal windows on the load test generator, since you will be running one scenario that generates calls and one that receives calls.  

In one window, start the receiving scenario (this must be done first):
```bash
sipp -sf uas.xml
```

In a second window, you can now start the sending application.  You should be able to use the same sending scenario as before, the difference is that on the jambonz side you must route the number to application that dials back to the load test generator.
