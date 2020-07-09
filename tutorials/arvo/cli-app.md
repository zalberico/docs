+++
title = "Command line interface apps"
weight = 13
template = "doc.html"
+++

## Command line interface apps

In this tutorial we will go in-depth into how to build command line interface (CLI)
applications in Urbit, in the course of which we will encounter a number of
libraries and applications used to facilitate this. Even if you have no interest
in writing CLI applications, this tutorial should be useful exposure to
the guts of a number of internal systems, including:

* Dill (the console vane)
* Gall (the userspace vane)
* dojo (the shell)
* `%hood` (for Gall apps interacting with Dill)
* `drum` (Hood library that tracks console display and input state for apps)
* `shoe` (library for CLI apps that handles boilerplate code)
* `sole` (a more bare-bones library for CLI apps...?)

There are three CLI apps that currently ship with urbit - `%dojo`, `%chat-cli`,
and `%shoe`. You should be familiar with the former two, the latter is an example
app that shows off how the `shoe` library works. These are all Gall apps, and as
such their source can be found in the `app/` folder of your `%home` desk.

We will investigate how to write CLI apps by investigating how the 
example app `shoe`
functions and then building a simple CLI app of our own. This tutorial can be
considered to be an extension of the [Hoon school
lesson](@/docs/tutorials/hoon/generators.md#ask) on `sole` and `%ask`
generators, which only covers the bare minimum necessary to write generators
that take user input.

CLI apps will someday include text user interfaces (TUIs) and so if you are
interested in creating those, this tutorial is also for you.

## Seven components

In this section we briefly summarize each of the seven components mentioned
above and what purpose they perform in a CLI app. Then we look at a move trace
showing how a command send to a CLI app navigates through these components.

### Dill

Dill is the Arvo vane that handles keyboard input from the user as well as
drawing the text in the console. You can learn more about what `task`s Dill is
responsible for in its [API documentation](@/docs/reference/vane-apis/dill.md).

### Gall

Gall is the vane that handles userspace apps.

### dojo

dojo is the first CLI app you encounter when you boot your ship and one you
should already be quite familiar with if you are reading this tutorial.

### `%hood`

`%hood` is a Gall app that mediates interactions between other Gall apps and
Dill. This is an important functionality since we don't want multiple CLI apps
attempting to draw to the console at the same time. Thus all traffic between
Gall apps and Dill ultimately routes through `%hood` at some point.

### `drum`

`drum` is Hood library that is used for handling `|command`s that you may have
used before, such as `|sync`, `|verb`, or `|hi`. (i think? this is based on a
comment in hood.hoon but I haven't looked into it further)

### `shoe`

`shoe` is a library utilized by CLI apps. This is where most of our focus will
be for this tutorial.

### `sole`

`sole` is an old library that is also for CLI apps, but with much less
functionality than `shoe`. It is still appropriate to use for writing generators
that handle user input, but anything more complex should use `shoe`, which does
the messier low-level work with `sole` on your behalf.

### How they work together

Here we give an overview of how the above components work together as a whole to
implement CLI functionality.

#### Sending a single character

Let's first look at a move trace of what happens when a single character is
pressed on the keyboard. For a more in-depth explanation of what this means, check out
the [move trace tutorial](@/docs/tutorials/arvo/move-trace.md).

Starting from dojo, we enable verbose mode by entering `|verb` and then
switching to `%chat-cli` with `Ctrl-X`. Then we press a
single character, say `a`. The following move trace shows how that `a` ends up
being displayed on the screen and passed to `%chat-cli` for further handling.

```
["" %unix %belt //term/1 ~2020.6.25..18.27.29..cbd5]
["|" %pass [%d %g] [[%deal [~zod ~zod] %hood %poke] /] ~[//term/1]]
["||" %give %g [%unto %fact] i=/d t=~[//term/1]]
["||" %pass [%g %g] [[%deal [~zod ~zod] %chat-cli %poke] /use/hood/~zod/out/~zod/chat-cli/drum/phat/~zod/chat-cli] ~[/d //term/1]]
```
We've omitted two `%poke-ack`s for clarity here, as they are a distraction from our discussion.

In English, this represents the following sequence of `move`s:

1. Unix sends `%belt` `card` to Arvo, which then triggers the `%belt` `task` in
   Dill. This is the `task` that Dill uses to receive input from the keyboard.
2. Dill `%pass`es the input to Gall, which `%poke`s the `%hood` app, telling
   `%hood` that the `a` key was pressed.
3. Gall `%give`s back to Dill a `%fact`, telling it to display the key that was
   pressed `a` in the terminal.
4. Gall `%pass`es to itself, `%poke`ing `%chat-cli` to inform it that `a` has been
   pressed. This `%poke` to `%chat-cli` is along the `duct`
   `/use/hood/~zod/out/~zod/chat-cli/drum/phat/~zod/chat-cli]` which tells us that...
   `chat-cli` utilizes hood and drum somehow? I'm not sure what exactly occurs here.
   
This exercise has shown us how keyboard input in `%chat-cli` goes from Unix to Dill to
Gall to `%hood`. In the next section we will see what is going on with `%hood`,
`drum`, `shoe`, and `sole`.

### The guts of `%chat-cli`

We will now investigate what happens when Gall `%deal`s `%chat-cli` a `%poke`
containing the letter `a`. Namely, where does this end up getting stored and
what libraries are being used here.

### The guts of `/app/shoe.hoon`


### The `shoe` library

Here we describe the different cores of `/lib/shoe.hoon` and their purpose.

An app using the `shoe` library will have `sole-ids=(list @ta)` as part of its
state. These are typically called session ids, and they are identifiers not only
for ships, but for multiple sessions on a given ship. An app using the `shoe`
library may be connected to by a local or remote ship in order to send commands,
and each of these connections is assigned a unique `@ta` that identifies the
ship and which session on that ship if there are multiple.

#### `shoe` core

A Gall agent core with extra arms for working with the `shoe` library. Use this
core whenever you want to receive input and run a command. The input will get
put through the parser (`+command-parser`) and results in a noun of
`command-type` that the underlying application specifies, then the app calls
that command.

In addition to the ten arms that all Gall core apps possess, `+shoe` has a few
more.

##### `+command-parser`

Input parser for a specific session. If the head of the result is true,
instantly run the command.

##### `+tab-list`

Autocomplete options for the sessions (to match `+command-parser`).

##### `+on-command`

Called when a valid command is run.

##### `+can-connect`

Called to determine whether the session can be connected to. For example, you
may only want the local ship to be able to connect to the session.

##### `+on-connect`

Called when a connection to the session is made.

##### `+on-disconnect`

Called when a session is disconnected.

#### `default` core

This core contains the bare minimum implementation of shoe arms. It is used
analogously to how the `default-agent` core is used for Gall apps.

#### `agent` core

This is a wrapper core that takes in the specialized `shoe` core and turns it
into a standard Gall agent core.


### The `sole` library

In order to display text, `shoe` creates `$shoe-effect`s which for now are just ` [%sole effect=sole-effect]`s which are eventually broken down (by `drum?`) into Dill
calls.

From `sur/sole.hoon`:
```hoon
++  sole-effect                                         ::  app to sole
  $%  {$bel ~}                                          ::  beep
      {$blk p/@ud q/@c}                                 ::  blink+match char at
      {$clr ~}                                          ::  clear screen
      {$det sole-change}                                ::  edit command
      {$err p/@ud}                                      ::  error point
      {$klr p/styx}                                     ::  styled text line
      {$mor p/(list sole-effect)}                       ::  multiple effects
      {$nex ~}                                          ::  save clear command
      {$pro sole-prompt}                                ::  set prompt
      {$sag p/path q/*}                                 ::  save to jamfile
      {$sav p/path q/@}                                 ::  save to file
      {$tab p/(list {=cord =tank})}                     ::  tab-complete list
      {$tan p/(list tank)}                              ::  classic tank
  ::  {$taq p/tanq}                                     ::  modern tank
      {$txt p/tape}                                     ::  text line
      {$url p/@t}                                       ::  activate url
  ==
```

### `shoe` app walkthrough

Here we go through the code in the `shoe` example app, explaining what each line
does. This is in preparation for the next chapter of the tutorial in which we
modify the app.

```hoon
::  shoe: example usage of /lib/shoe
::
::    the app supports one command: "demo".
::    running this command renders some text on all sole clients.
::
/+  shoe, verb, dbug, default-agent
```

`/+` is the Ford rune wihich imports libraries from the `/lib` directory into
the subject.
 * `shoe` is the `shoe` library.
 * `verb` is...
 * `dbug` is a library of debugging tools. Why do we need this?
 * `default-agent` contains a Gall agent core with minimal implementations of
   required Gall arms.

```hoon
|%
+$  state-0  [%0 ~]
+$  command  ~
::
+$  card  card:shoe
--
```
The types used by the app.

`$state-0` stores the state of the app, which is null as there is no state to
keep track of. It is good practice (required?) to have a type for state anyways
in case the app is made stateful at a later time, and indeed we will do this
when we modify this app later in the tutorial.

`$command` is typically a set of tagged union types that are possible
commands that can be entered by the user. Since this app only supports one
command, it is unnecessary for it to have any associated data, thus the command
is represented by `~`.

In a non-trivial context, a `$command` is given by `[%name
data]`, where `%name` is the identifier for the type of command and `data` is
a type or list of types that contain data needed to execute the command. See
`app/chat-cli.hoon` for examples of commands, such as `[%say letter:store]` and
`[%delete path]`.

`$card` is either an ordinary Gall agent `card` or a `%shoe` `card`, which takes
the form `[%shoe sole-ids=(list @ta) effect=shoe-effect]`. A `%shoe` `card` is
sent to all `sole`s listed in `sole-ids`, instructing them to implement the
action specified by `effect`. Here we can
reference `card:shoe` because of `/+  shoe` at the beginning of the app.

```hoon
=|  state-0
=*  state  -
::
%+  verb  |
%-  agent:dbug
^-  agent:gall
%-  (agent:shoe command)
^-  (shoe:shoe command)
|_  =bowl:gall
+*  this  .
    def   ~(. (default-agent this %|) bowl)
    des   ~(. (default:shoe this command) bowl)
::
++  on-init   on-init:def
++  on-save   !>(state)
++  on-load
  |=  old=vase
  ^-  (quip card _this)
  [~ this]
::
++  on-poke   on-poke:def
++  on-watch  on-watch:def
++  on-leave  on-leave:def
++  on-peek   on-peek:def
++  on-agent  on-agent:def
++  on-arvo   on-arvo:def
++  on-fail   on-fail:def
::
++  command-parser
  |=  sole-id=@ta
  ^+  |~(nail *(like [? command]))
  (cold [& ~] (jest 'demo'))
::
++  tab-list
  |=  sole-id=@ta
  ^-  (list [@t tank])
  :~  ['demo' leaf+"run example command"]
  ==
::
++  on-command
  |=  [sole-id=@ta =command]
  ^-  (quip card _this)
  =-  [[%shoe ~ %sole -]~ this]
  =/  =tape  "{(scow %p src.bowl)} ran the command"
  ?.  =(src our):bowl
    [%txt tape]
  [%klr [[`%br ~ `%g] [(crip tape)]~]~]
::
++  can-connect
  |=  sole-id=@ta
  ^-  ?
  ?|  =(~zod src.bowl)
      (team:title [our src]:bowl)
  ==
::
++  on-connect      on-connect:des
++  on-disconnect   on-disconnect:des
--
```

### Example app idea

Able to be connected to from multiple ships and sessions. Each can run one
command, which displays the name of the ship and its session id to all ships. So
basically it should just be a modification of the demo app to allow for multiple
sessions... Maybe this is what it already does though? How do I connect to the
app from a remote ship?
