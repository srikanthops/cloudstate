# Forwarding and effects

A command may only act on one entity at a time. Sometimes however, you would like to update multiple entities in a single command. An example of this might be wanting to publish a message to a topic entity after an event sourced entity processes a command, to inform any watchers of that entity.

## Forwarding control to another entity

An entity may, rather than sending a reply to a command, forward it to another entity. This is done by sending a forward message back to the proxy, instructing the proxy which call on which entity should be invoked, and passing the message to invoke it with.

The command won't be forwarded until any state actions request by the command handler have successfully completed. It is the responsibility of the forwarded action to return a reply that matches the type of the original command handler. Forwards can be chained arbitrarily long.

## Emitting effects on another entity

An entity may also emit one or more effects. An effect is something whose result has no impact on the result of the current command - if it fails, the current command still succeeds. The result of the effect is therefore ignored. Effects are only performed after the successful completion of any state actions requested by the command handler.

Effects may be declared as synchronous or asynchronous. Asynchronous commands run in a "fire and forget" fashion. The code flow of the caller (the command handler of the entity which emitted the async command) continues while the command is being asynchronously processed. Meanwhile, synchronous commands run in "blocking" mode, ie. the commands are processed in order, one at a time. The final result of the command handler, either a reply or a forward, is not sent until all synchronous commands are completed.

## Transactional concerns

It's important to note that forwarded commands, and commands emitted as side effects, are **non**-atomic, ie. there is no guarantee that the transactions either all succeeded or none. If the user function or store fails while a forwarded command is executing, the original command that triggered it will not be rolled back.

Hence, forwarding and effects should not be used to update multiple entities at once if partial updates is a problem. In such cases, it's best to initiate commands from an event log consumer (future Cloudstate functionality), which can guarantee that a sequence of operations will run through to completion.
