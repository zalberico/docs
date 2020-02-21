+++
title = "Common Tasks"
weight = 2
template = "doc.html"
+++

# Common Tasks

There is a set of `task`s defined in `+vane-task` in `zuse` that are common to
most vanes. Each vane has a different implementation for these `task`s, but they
each perform the same basic function.

## +vane-task

For convenience, we copy here the code for `+vane-task` in `zuse`, which gives
the structure of the `card`s associated with these `task`s.

```hoon
+$  vane-task
  $~  [%born ~]
  $%  ::  i/o device replaced (reset state)
      ::
      [%born ~]
      ::  error report
      ::
      [%crud p=@tas q=(list tank)]
      ::  boot completed (XX legacy)
      ::
      [%init p=ship]
      ::  trim state (in response to memory pressure)
      ::
      [%trim p=@ud]
      ::  kernel upgraded
      ::
      [%vega ~]
      ::  produce labeled state (for memory measurement)
      ::
      [%wegh ~]
      ::  receive message via %ames
      ::
      [%plea =ship =plea:ames]
  ==
```

### `%born`

The Arvo kernel `%pass`es a `%born` `task` to each vane after Arvo is reset. For
example, Behn resets its timer to be in sync with the Unix timer when it is
`%pass`ed a `%born` `task`.

### `%crud`

`%crud` `task`s are for error reporting. Whenever an event crashes, the vane
that was originally handling that event is `%pass`ed a `%crud` `task` with an
error report that is generated by the interpreter and injected into the kernel.
What the vane does with the error report at that point is left up to the vane.

### `%init`

The `%init` `task` is called once as a part of the initial boot process during the [larval
stage](@/docs/tutorials/arvo/arvo.md#larval-stage-core) of the Arvo kernel and
is never used again. It
occurs after the entropy and identity is added but before the metamorphosis into
the [adult stage](@/docs/tutorials/arvo/arvo.md#structural-interface-core). 

Jael is the first vane to be initialized, finalizing the ship's registration
with Azimuth, which then goes on to trigger an `%init` `task` for each of
the other vanes.

### `%trim`

The `%trim` `task` is sent to a vane by the interpreter when memory is getting
low. The vane then attempts to do some garbage collection to clear up memory.

Behn's `%trim` `task` is empty because it is utilized for scheduling and so cannot
afford to forget any timers. Ford, on the other hand, has an extensive and
nontrivial `%trim` `task`.

### `%vega`

After the kernel is upgraded it calls the `%vega` `task` for each vane, which
contains code that tells the vane what to do in the case of a kernel upgrade,
such as migrating to a new type of state.

### `%wegh`

`%pass`ing a vane a `%wegh` `task` tells the vane to calculate how much memory
it is using. This is done by gathering the entire state, labeling the different
parts, and returning it as nouns to the interpreter, which can then easily
determine the memory usage as nouns are acyclic data structures.

A `%mass` `gift` is returned in response to a `%wegh` `task`.

### `%plea`

`%plea`s are how vanes utilize Ames to communicate with vanes on other ships.

Say that Vane X on ship A wants to communicate with vane Y on ship B. Vane X
makes a `%plea` to Ames on ship A, which then makes a `%plea` to Ames on ship B,
which then makes a `%plea` to Vane Y on ship B.

`%plea`s are the `%pass` portion of communication via Ames, and `%memo`s are the
`gift` that are `give`n in return.