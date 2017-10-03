---
layout: post
title:  "Developer Conversation: Functional Programming, Message-Passing, and Objects"
author: louis_jackman
comments: true
---

Alice and Bob take time away from sending each other encrypted messages in
envelopes to discuss programming esoteria.

---

**Alice**

Hey, a showerthought: isn't immutability incompatible with the original purpose
of OOP?

Because as I understand it, OOP is basically "objects handle their own state
through encapsulation".


**Bob**

OOP originally meant messages being passed around to self-contained objects.
Whether those objects used mutable or immutable implementations would
theoretically be encapsulated away. Alan Kay, who coined the term OOP, later
claimed regretting the over emphasis on classes, inheritance, etc.
Message-passing is the only thing OOP really does.

Consider these two programs. Note they do both message passing, but one is
immutable and the other is mutable:

```erlang
counter(N) ->
    receive
        inc -> counter(N + 1);
        reset -> counter(0);
        display ->
            io:format("~A", N),
            counter(N)
    end.
```

```java
class Counter {

    private int n;

    Counter(int n) {
        this.n = n;
    }

    void inc() {
        ++n;
    }

    void reset() {
        n = 0;
    }

    void display() {
        System.out.println(n);
    }

}
```

Note that neither is better nor worse; mutability and immutability are both
valid approaches to handling these message-passing objects.

Ofc, mutability has complexities with concurrency that the Erlang example
doesn't have, but that's another topic.

**Alice**

What do you mean by *messages*? For example, in the Java snippet, what are
messages?

**Bob**

Invoking a method == sending a message.

The first is Java/C++ terminology, the latter is Smalltalk and Objective-C
terminology.

In "real" OOP, everything is a message. The original idea of objects by Alan Kay
was closer to Erlang processes than modern Gang of Four-inspired class-based
OOP.

Smalltalk is class-based ofc, but it's not a critical requirement.

OOP is message-passing; whether languages call it "invoking a method" or
"sending a message" doesn't really matter.

**Alice**

But isn't everything message-based then? What are other languages that are *not*
message-based?

**Bob**

That's a hard question, because there are divergences in the answer from
different areas of CS.

GoF-style C#/Java programmers might argue that any function that "belongs" to an
object or class is a method, and that invoking it is basically passing a
message. They'd probably agree that virtual methods (i.e. non-final methods in
Java) are more qualified to be called "message passing" that non-virtual ones.

Smalltalk and Obj-C programmers would argue that message passing must allow
message interception, proxying, and handling all unknown messages. They'd argue
that Java and C#'s methods aren't "real message-passing" because they don't
support those properties. (Actually Java supports proxying these days.) In fact,
they'd say that OOP and static typing cannot coexist because you should be able
to pass any message to any object.

Some Erlang programmers might argue that message-passing's ultimate conclusion
is actors and mailboxes, and that subroutines were a first step in that
direction, Smalltalk-style methods were another step, and now their actors are
the next step after that. They'd say that message-passing doesn't make sense
unless they are all asynchronous, which Erlang's/BEAMs mailboxes are but
Smalltalk and Java's methods are not.

Then you have the actor/procedure distinction made by Guy Steele et al, which is
actually tied to tail call optimisation. They'd argue that any tail call is
effectively an actor message since there is no call stack and execution is
jumping to a new operation. I disagree though, as I'd think a message passing
system would have an execution flow for each object/actor, and not transferred
ones.

**Alice**

But then, are OOP and FP eventually compatible, right?

**Bob**

Depends on whether you ask an Erlang programmer or a Haskell programmer ;)

If an actor sends different messages back over time, like a counter actor, that
wouldn't be referentially-transparent, which is arguably the most important
property for functional programming.

Referential transparency is really what functional programming is about; the
stuff about tuples and lambdas is a bit of a distraction.

**Alice**

I don't get it... Ah, wiki seems to be quite clear about it:

> In functional code, the output value of a function depends only on the
> arguments that are passed to the function, so calling a function f twice with
> the same value for an argument x will produce the same result f(x) each time.

Is that referential transparency?

**Bob**

Yeah, that Wiki summary sounds solid to me.

Using the actor above, but assuming we have a `get` message that sends the count
back to the sender:

```elixir
counter = spawn(fn() -> counter() end)
counter.send(:inc)
counter.send(:get)
counter.send(:inc)
counter.send(:get)

receive do
    n -> IO.puts n
end

receive
    n -> IO.puts n
end
```

Those two `receive`s display different results, despite being identical. That's
a violation of referential transparency.

The idea is that there are no side-effects, so a function's domain always maps
to the same range.

So you can always be absolutely sure what something is doing.

You don't have a magic flag that gets changed somewhere that subtly changes some
behaviour somewhere else.

**Alice**

OK

Sounds like the obviously right approach, weird we're still dealing with states
everywhere...

**Bob**

OOP is about controlling state through encapsulation, FP is about eliminating
state altogether.

**Alice**

Wait, so OOP = encapsulation, FP = no state?

**Bob**

Well, not really actually, the Erlang example above has state but is immutable.

It defines state as the execution of processes over time, whereas most languages
define state as modifying a memory location.

Erlang correctly realised that only state as processes scales, and state as
memory addresses does not.

Developers sometimes don't encapsulate state properly, so eliminating it from
the language works pretty well.

**Alice**

True

**Bob**

Notice that counter receives the next message by calling itself with a new
argument.

That is why locking isn't needed with the actor model.

It doesn't move onto the next message until finished with the current one.

**Alice**

OK. This kind of what closure are in JS right? Higher scopes variables available
in "sub functions"?

**Bob**

Right, with a very important point: everything is immutable. So *bindings* can
be captured, but values themselves are not. So you never get into a problem
where the wrong value is captured.

That's also why Elixir allows rebinding the same name over and over again in one
function without introducing mutability.

**Alice**

Right, got it. Like in JS, you could actually modify the content of an array
without changing its reference, so the closure would actually have the modified
array.

While in FP, this is not possible.

**Bob**

Yep.

**Alice**

Sounds great!

**Bob**

Which has major consequences in the larger picture.

**Alice**

Yeah, I can imagine!

**Bob**

Ok, for example, say you send a message with a massive data structure. In Java
you'd need to defensively copy it to ensure it wasn't changed. In Erlang, you
just copy it by reference, which is very cheap, and they'd just *know* the
receiver can't change its own behaviour by tampering with anything.

Another consequence of immutability: value-vs-reference becomes meaningless.
There is no such thing in Erlang or Haskell.

Whether things are copied by value or reference becomes a mere optimisation
detail of the underlying VM.

And once you know local memory isn't changing, sending a message over the
network becomes semantically identical to sending a message to a local process.

**Alice**

Yeah, true.

But something I'm not sure I understand: some guys one day said "hey, here is
what I propose: we use encapsulation, we call it OOP".

If they said instead "we use message-passing, we'll call it OOP", then people
would have said "guys, we're already doing this for years"!

**Bob**

Well, it is about message passing. Encapsulation, it turns out, is needed for
maintainable message passing systems. Encapsulation can be closures holding
exclusive access to a variable, `private` fields, whatever. Note that
encapsulation isn't tied to mutability or immutability; Erlang and Java above
both encapsulate their counts.

Encapsulation is just a useful building block for the central idea of OOP:
message-passing.

**Alice**

OK...

**Bob**

Ofc, as you correctly point out, OOP became a fad term and then completely
misunderstood. We have the "Pillars of OOP" nonsense to thank for that.

**Alice**

Well, I don't get how OOP could say "now, use this new message-passing
paradigm", while everything was message passing already...

But anyway, clearer already :)

**Bob**

Tbf, Smalltalk was around in the early '80s, and their definition of
message-passing (see above) was new at the time.

**Alice**

Alright!

Well, this conversation could deserve a blog post actually ;)

**Bob**

Done!
