## Getting the Actual Resource Usage#
We saw some of the effects that can be caused by a discrepancy between resource usage and resource specification. It’s only natural that we should adjust our specification to reflect the actual memory and CPU usage better.

### The DB Container#
Let’s start with the database.

![image](https://user-images.githubusercontent.com/33947539/185898961-0350f67c-2f87-414e-84d8-8f7c29390434.png)

As expected, an api container uses even less resources than MongoDB. Its memory is somewhere between 3Mi and 6Mi. Its CPU usage is so low that Metrics Server rounded it to 0m.

>The memory usage resources may differ from the above mentioned limits.

💡 Things to Keep in Mind#
Equipped with this knowledge, we can proceed to update our YAML definition. Still, before we do that, we need to clarify a few things.

The metrics we collected are based on applications that do nothing. Once they start getting a real load and start hosting production size data, the metrics would change drastically.

What you need is a way to predict how much resources an application will use in production, not in a simple test environment. You might be inclined to run stress tests that would simulate a production setup. It’s significant, but it does not necessarily result in real production-like behavior.

Replicating the production and behavior of real users is tough. Stress tests will get you half-way. For the other half, you’ll have to monitor your applications in production and, among other things, adjust resources accordingly.

There are many additional things you should take into account but, for now, we wanted to stress that applications that do nothing are not a good measure of resource usage. Still, we’re going to imagine that the applications we’re currently running are under production-like load and that the metrics we retrieved represent how the applications would behave in production.

ℹ️ Simple test environments do not reflect production usage of resources. Stress tests are a good start, but not a complete solution. Only production provides real metrics.

