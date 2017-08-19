---
layout: post
title:  "Async IO in Today's Programming Languages"
author: louis_jackman
comments: true
---

Async IO has become a must-have feature on the checklist of trendy new
technology stacks, yet they often disagree on what passes as 'asynchronous' or
even 'non-blocking' features.  Similar disagreements exist about the precise
definition of other concurrency constructs like semaphores, mutexes, and
critical regions.

In contrast to other concurrency constructs, asynchronous language constructs
like `await/async` in
[JavaScript](https://github.com/tc39/ecmascript-asyncawait),
[C#](https://msdn.microsoft.com/en-us/library/hh191443(v=vs.120).aspx), and
[Python](https://www.python.org/dev/peps/pep-0492/) exist to deal with
operations that 'block', registering an action to run at some point in the
future when the blocking finishes. As these blocks manifest outside of the
language, like in the operating system's network stack, even languages like
JavaScript that lack fine-grained parallelism or concurrency must deal with
events that occur not only later on, but can happen in any order relative to
other events, as determined by unforeseeable events. When the blocking finishes,
the system must be able to resume from where if left off; it needs to track the
context.

```javascript
// JavaScript (ECMAScript 6+)

const processRecords = async (fileName) => {
    const file = openFile(fileName);

    // This can take a while if it's multiple gigabytes; remember the context
    // and pick up later once it is finished.
    const content = await readFileContents(file);

    return process(content);
};
```

## The Problem

As blocking comes from external factors, like when a user's browser half way
across the world chooses to finish its HTTP requests, or an old IDE harddrive
manages to finally write that large file onto its platter, code must be able to
hold onto the context with as few resources as possible. Poor reliability on a
network, a nefarious DoS attempt, or simply too many users will force the system
to juggle so many of these contexts that resource consumption spirals out of
control.

Enter asynchronous IO.

```python
# Python 3.5+

async def handle_http_request(user, pending_request):
    handle = user.make_http_request_handler()

    # Loading the entire request could take as long as the user decides; what if
    # they stall midway through sending a single HTTP request? The context
    # idles, wasting system resources until it is resumes. Fortunately, we're
    # using a non-blocking API.
    request = await pending_request.load_all()

    return handle(request)
```

## Async IO's Solution

The premise is this: the old way of remembering execution contexts, by simply
blocking 'kernel threads' until an operation resumes, requires too much memory
and incurs heavy processing costs due to context switching. So we instead give
the blocking event a piece of code to run when it's finished; more specifically,
we throw the code into an event loop, in which it lies dormant until a blocking
operation completes and runs the pieces of code waiting for it.

Kernel threads are tracked by the operating system, whether it be Linux, macOS,
or Windows. Those costs come from the lack of assumptions a thread can make; it
assumes that all CPU state like registers must be stashed away and restored
between switches. It must also track the whole callstack of every thread, even
if the small piece of code responding to the event needs only a fraction of it.
Furthermore, kernel threads must give roughly equal time to different threads,
incurring the overhead of a scheduler that is constantly trying to slice up
resources across competing threads.

Instead, adding a response to the event loop 'unwinds' the stack, freeing
memory, and lets the blocking operations trigger events rather than trying to
preemptively schedule with the overhead it incurs. This is one of the reasons
Nginx's event-driven architecture was renown for being faster than old-school
Apache, which dedicated a thread per HTTP request.

```javascript
// JavaScript (ECMAScript 6+)

// Don't get the content straight from the file; instead, give the file reader a
// function, which is thrown into the event loop and triggered when the file has
// finished reading. As we're not waiting to read the file _right now_, the
// `readFile` function returns immediately.
fs.readFile('a/very/large/file.txt', (err, content) => {
    if (!err) {
        doSomethingToTheFile(content);
    }
});
```

## Synchronosity and Blocking

What's the difference between a synchronous function and a blocking one?
Synchronisity covers how the code's execution flows by using the function,
whereas the actually performance characteristics are determined by whether the
function blocks.

_These two traits are separate and do not depend on one another._

It's possible for an API to be non-blocking and asynchronous, but also to be
non-blocking and _synchronous_. Asynchronosity is whether the stack unwinds back
to an event loop or whether it just waits around until the response, after which
it continues running from where it stopped.

As asynchronous code is more complex than synchronous code, and is only needed
for non-blocking code. There is no use case for blocking asynchronous code, but
all other combinations exist.

```erlang
% Erlang

F = fun() -> do_expensive_network_bound_operation() end.

% 9999 processes are spawned with each process doing expensive network IO
% processing with synchronous APIs, yet there aren't 9999 threads idling on when
% the network suffers a short period of downtime. There aren't any thread pools
% being starved even with large amounts of processes waiting for IO.
[spawn(F) || _ <- lists:seq(1, 9999].


% An example of a synchronous non-blocking API.
```

## The State of Async IO in Today's Languages

Using the asynchronous style has varying support across modern languages. There
are a few common ones today.

Node.js is non-blocking with an asynchronous API, as it requires the programmer
to distinguish blocking operations by usage: dealing with promises or passing
along callback functions. The non-blocking features ensure the stack is
immediately unwound upon being invoked. All in all, Node.js handles non-blocking
IO well, as it has very few synchronous blocking APIs, except require and
functions suffixed with `Sync`, making it clear and obvious when blocking
operations occur.

C, C++, Java, C#, and Rust have two modes: blocking synchronous and non-blocking
asynchronous. C, C++, and Rust can access non-blocking APIs provided by the
operating system like epoll or kqueue, which expose non-blocking asynchronous
APIs. C# and Java both support async; C# has await and async syntax for tasks,
which are basically promises, while Java has a vaguely monadic-looking API. The
point is, all these APIs require the programmer to distinguish ['green and red'
functions](http://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/),
i.e. functions that are asynchronous and those that are not. The original .NET
and JVM use blocking synchronous APIs for their first versions; they added
asynchronous APIs later on.

Go, Haskell and the BEAM languages Erlang and Elixir, are non-blocking with
synchronous APIs. The programmer doesn't distinguish between blocking and
non-blocking operations and leaves it entirely to the runtime to work out.
Expecting the programmer to always remember to unwind the stack rather than hit
a blocking API, à la Java or C#, is less acceptable for the domains Go and
Erlang target, such as large networking services with Go at Google or telecoms
utilising Erlang with very frequent context switching.

There are also languages like Scheme and Lua which support undelimited
continuations or asymmetric coroutines. These allow Go/Erlang style IO to
implemented at the library level rather than in the language, but it comes as
the expense of preemptive scheduling. Without preemptive scheduling, run away
processes can starve out responses to other events; a good example is the DoS
attacks against Node.js using overly permissive regular expressions.

```lua
-- Lua 5+

-- You have no idea whether this will context switch or not; it's the API
-- designer's decision. The API decides when to switch, and unlike await/async,
-- it doesn't need to define it as part of the API. It can just choose to do it
-- in the bowels of its implementation. The advantage is that APIs can add
-- transparent non-blocking support without changing APIs; the disadvantage is a
-- not being able to make assertions about the order of events.
write_request_payload()
```

## To Summarise

* Out of order execution and unpredictable ordering of event responses are not
  restricted to languages with extensive parallelism and concurrency support;
  even JavaScript must worry about this with asynchronosity.
* Non-blocking APIs do _not_ need to be asynchronous; this is just an
  implementation artefact of some contemporary language runtimes.
* Blocking features aren't viable for code expected to span across a large
  amount of concurrent users.
* [Concurrency is not
  Parallelism.](https://blog.golang.org/concurrency-is-not-parallelism)
