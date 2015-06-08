# transducifer
Fatally flawed attempt to maintain sanity in a program running in a chaotic world.

Code is traditionally written with the assumption that all mutations are the result of the program itself.
```
x = 5
x += 1
x -= 1
assert(x == 5) // and all is well in the world
```
Modern compilers and hardware conspire to keep us unaware of the lie that programs run as declared.  We don't care that our instructions are often being reordered at several layers.  We don't care that multiple cores maintain separate copies of shared mutable data, because our cache coherence mechanisms are pretty awesome.  We can just throw a lock around it and expect that it will work.
```
l = lock
s = 0
nThreads = 50
f = func() {
  l.lock()
  s += 1
  l.unlock()
}
for i in nThreads:
  spawn f()
assert(s == nThreads)  // and all is well in the world
```
As our machines exist across separate [address/physical] spaces, synchronization pains become more evident.
```
s = socket(AF_INET, SOCK_STREAM).connect("host-f5b4e420:5876")
assert(s.connected()) // and all is well in the world

s.send(`{"call": "add", "args": [%d, 1]}`.format(s))
assert(s.recv() == `{"return": %d}`.format(s + 1))  // FAILURE OF SOME SORT!
```
In static configurations, a retry using an exponential backoff will often be all you need.  Today, we are often not running code in static configurations.  Maybe the node referenced by the address host-f5b4e420 doesn't even exist anymore.  Maybe another node has taken its place.  When we retry this code, will another DNS query occur, or will we attempt to connect to the node that host-f5b4e420 resolved to the first time?

Anyway, we can generally expect our infrastructure to change while some of our programs continue to run.  Our programs may gradually learn about new events, and the correct way foward may change.

We have a sequence of inputs, an overall state and a sequnce of outputs.  There is no guarantee of commutativity: a->b->c may produce a different output sequence than b->a->c.  Inputs of {cluster_healthy,cluster_unhealthy} will result in different sequences of outputs depending on their ordering in the input sequence.  Maybe we only want to take corrective action when a particular health check has been unhealthy for the past 3 minutes.

We may want to compile several FST's into a single FST.

