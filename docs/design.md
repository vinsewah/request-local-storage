# Why request-local-storage?

Request local storage handles a particular problem, that of
[high arity functions](https://en.wikipedia.org/wiki/Arity) in request/response
objects and their associated callbacks.  As the author of an express middleware,
your middleware is called off once on the way in with a request object, then you
lose context, and then you are called again with the response object.  
Generally, you start many asynchronous calls, that might in turn make function
calls that take callbacks, and accumulate very long chains of callbacks, such
that adding a new parameter to the last callback in the chain requires modifying
many functions along the way.  Request local storage fixes this by implementing
a [continuation](https://en.wikipedia.org/wiki/Continuation) which
[reifies](https://en.wikipedia.org/wiki/Reification_(computer_science)) the
"thread" of execution.


# How does it work?

Under the covers, request local storage uses
[continuation local storage](https://github.com/othiym23/node-continuation-local-storage),
and adds [namespaces](https://en.wikipedia.org/wiki/Namespace) on top of it to
prevent requests from colliding with each other.  CLS keeps track of the number
of listeners that have access to a given continuation by using a proposed node
api called [asyncWrap](https://github.com/nodejs/tracing-wg/tree/master/docs/AsyncWrap)
(implemented with a [polyfill](https://github.com/othiym23/async-listener)),
which is intended to part of the story for how [domains](https://nodejs.org/api/domain.html)
will ultimately be deprecated from [node.js](https://nodejs.org/en/).  Request
state, then, is stored on `process.namespaces`, so that you can access your
request state from any module by `require`ing request local storage, which will
then handle registering and deregistering "listeners", or callbacks that should
have access to your namespace, and maintains and retrieves state from its
backing continuation local storage.


# Prior Art

[Flask](http://flask.pocoo.org/docs/0.10/api/#flask.g)
[Thread local storage](https://en.wikipedia.org/wiki/Thread-local_storage)
