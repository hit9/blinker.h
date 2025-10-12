Blinker.h
---------

![](https://github.com/hit9/blinker.h/actions/workflows/tests.yml/badge.svg)
![](https://img.shields.io/badge/license-BSD3-brightgreen)

A lightweight signal/event library for C++, similar to Python's blinker, but designed to work with ticking loops.

中文博客: https://hit9.dev/post/blinker

## Code Example

1. Setup board, signals and connections:

   ```cpp
   // Creates a board.
   Blinker::Board board;

   // Creates signals, returns shared pointers.
   auto ab = board.NewSignal("a.b");
   auto ac = board.NewSignal("a.c");

   // Connects to some signals, returns unique pointers
   auto connection = board.Connect("a.*");
   ```

2. In your tick function:

   To emit signals (to backend buffer):

   ```cpp
   ab->Emit(123);
   ac->Emit("some string");
   ```

   To poll fired signals (from frontend buffer):

   ```cpp
   connection->Poll([&](const Blinker::SignalId id, std::any data)) {
     std::cout << std::any_cast<int>(data) << std::endl;
   };
   ```

   Finally, don't forget flip the double buffers before this tick end:

   ```cpp
   board.Flip();
   ```

Also checkout [example/main.cc](example/main.cc).

## Mechanism and Concepts

1. A signal board contains at most N signals.
   Where a signal is just composed of an id and name.

2. A signal's name should be delimited by dots, e.g. "movement.arrived".

   Signals are structured into a trie by names in the pre-process stage.

3. A subscriber is just a function.
   It can connect to one or multiple signals by providing signal names,
   or prefix patterns e.g. "movement.*".

4. Connections are owned by user, the board doesn't manage them.

5. Each connection has a signature, abstracts as a bitset, of which the n'th bit
   setting to true means that the signal of id n is subscribed by this connection.

6. Runtime signal dispatching is done by bitset's AND operation, instead of name matching,
   so it's fast enough. The string matching is only performed at pre-process stage, to generate signatures for connections.

7. Double buffers for a board under the hood:
   1. The frontend buffer is for subscribers to poll.
   2. The backend buffer is for new signal emittings.
   3. The two should be flipped on each tick.
   4. Each buffer owns a signature of fired signals.

## Instant Poll

In Blinker.h's typical design, subscribers handle signals delayed in the next tick,
if you wanna some signals (such as user's inputs) to be handled as quick as possible, instantly in the current tick,
just make a standalone board for those signals, and flip before logics `Update` function, for an example:

```cpp
// there're two boards:
// inputBoard: signals from user inputs.
// internalBoard: for internal signals emitting and handling
while (...) {

    handleInputEvents();  // transfer to signals
    inputBoard.Flip();  // flip fired signals to frontend.

    Update(); // main logics Update
    Draw(); // render stuffs.

    internalBoard.Flip();
}
```


License
-------

BSD.
