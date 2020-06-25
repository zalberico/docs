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
* `drum` (Hood library for...)
* `shoe` (library for CLI apps)
* `sole` (a more bare-bones library for CLI apps...?)

We will investigate how to write CLI apps by investigating how `chat-cli`
functions and building a simple CLI app of our own. This tutorial can be
considered to be a vast extension of the [Hoon school
lesson](@/docs/tutorials/hoon/generators.md#ask) on `sole` and `%ask`
generators, which only covers the bare minimum necessary to write generators
that take user input.

CLI apps may someday include text user interfaces (TUIs) and so if you are
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

#### The guts of `%chat-cli`

We will now investigate what happens when Gall `%deal`s `%chat-cli` a `%poke`
containing the letter `a`. Namely, where does this end up getting stored and
what libraries are being used here.
