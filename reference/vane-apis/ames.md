+++
title = "Ames Public API"
weight = 1
template = "doc.html"
+++

# Ames

In this document we describe the public interface for Ames.  Namely, we describe
each `task` that Ames can be `%pass`ed, and which `gift`(s) Ames can `%give` in return.

Ames `task`s can be naturally divided into two categories: messaging tasks and
system/lifecycle tasks.

## Messaging Tasks

### %hear

#### Accepts

#### Returns

#### Source


### %heed

#### Accepts

#### Returns

#### Source


### %hole

#### Accepts

#### Returns

#### Source


### %jilt

#### Accepts

#### Returns

#### Source


### %plea

#### Accepts

#### Returns

#### Source



## System and Lifecycle Tasks

### %born

Each time you start your Urbit, the Arvo kernel calls the `%born` task for Ames.

#### Accepts

`%born` takes no arguments.

#### Returns

In response to a `%born` `task`, Ames `%give`s Jael a `%turf` `gift`.

#### Source

```hoon
  ++  on-born
    ^+  event-core
    ::
    =.  unix-duct.ames-state  duct
    ::
    =/  turfs
      ;;  (list turf)
      =<  q.q  %-  need  %-  need
      (scry-gate [%141 %noun] ~ %j `beam`[[our %turf %da now] /])
    ::
    (emit unix-duct.ames-state %give %turf turfs)
```
    
### %crud

`%crud` is called whenever an error involving Ames occurs. It produces a crash
report in response.

#### Accepts

```hoon
=error
```

A `$error` is a `[tag=@tas =tang]`.

#### Returns

In response to a `%crud` `task`, Ames returns `event-core` with a new `%pass`
move to Dill to be performed instructing it to print the error.

#### Source

```hoon
  ++  on-crud
    |=  =error
    ^+  event-core
    (emit duct %pass /crud %d %flog %crud error)
```


## %init

`%init` is called a single time during the very first boot process, immediately
after the [larval stage](@/docs/tutorials/arvo/arvo.md#larval-stage-core)
is completed. This initializes the vane. Jael is initialized first, followed by
other vanes such as Ames.

In response to receiving the `%init` `task`, Ames subscribes to the information
contained by Jael.

#### Accepts

```hoon
our=ship
```

`%init` takes in the name of our ship, which is a `@p`. 

#### Returns

```hoon
    =~  (emit duct %pass /turf %j %turf ~)
        (emit duct %pass /private-keys %j %private-keys ~)
```

`%init` returns `event-core` with two new `move`s to be performed that subscribe
to `%turf` and `%private-keys` in Jael.

#### Source

```hoon
  ::  +on-init: first boot; subscribe to our info from jael
  ::
  ++  on-init
    |=  our=ship
    ^+  event-core
    ::
    =~  (emit duct %pass /turf %j %turf ~)
        (emit duct %pass /private-keys %j %private-keys ~)
    ==
```


### %sift

This `task` filters debug output by ship.

#### Accepts

```hoon
ships=(list ship)
```

The list of ships for which debug output is desired.

#### Returns

```hoon
    =.  ships.bug.ames-state  (sy ships)
    event-core
```

This `task` returns the `event-core` modified so that debug output is shown only
for `ships`.

#### Source

```hoon
  ::  +on-sift: handle request to filter debug output by ship
  ::
  ++  on-sift
    |=  ships=(list ship)
    ^+  event-core
    =.  ships.bug.ames-state  (sy ships)
    event-core
```


### %spew

Sets verbosity toggles on debug output. These toggles are as follows.

* `%snd` - sending packets
* `%rcv` - receiving packets
* `%odd` - unusual events
* `%msg` - message-level events
* `%ges` - congestion control
* `%for` - packet forwarding
* `%rot` - routing attempts

Each toggle is a flag set to `%.n` by default.

#### Accepts

```hoon
verbs=(list verb)
```

`%spew` takes in a `list` of `verb`, which are verbosity flags for Ames.

```hoon
+$  verb  ?(%snd %rcv %odd %msg %ges %for %rot)
```

`%spew` flips each toggle given in `verbs`.

#### Returns

`+on-spew` returns `event-core` with the changed toggles.

#### Source

```hoon
  ::  +on-spew: handle request to set verbosity toggles on debug output
  ::
  ++  on-spew
    |=  verbs=(list verb)
    ^+  event-core
    ::  start from all %.n's, then flip requested toggles
    ::
    =.  veb.bug.ames-state
      %+  roll  verbs
      |=  [=verb acc=_veb-all-off]
      ^+  veb.bug.ames-state
      ?-  verb
        %snd  acc(snd %.y)
        %rcv  acc(rcv %.y)
        %odd  acc(odd %.y)
        %msg  acc(msg %.y)
        %ges  acc(ges %.y)
        %for  acc(for %.y)
        %rot  acc(rot %.y)
      ==
    event-core
```

### %vega

`%vega` is called whenever the kernel is updated. Ames currently does not do
anything in response to this.

#### Accepts

`%vega` takes no arguments.

#### Returns

`%vega` returns `event-core`.

#### Source

```hoon
  ::  +on-vega: handle kernel reload
  ::
  ++  on-vega  event-core
```


### %wegh

This `task` is a request to Ames to produce a memory usage report.

#### Accepts

This `task` has no arguments.

#### Returns

In response to this `task,` Ames `%give`s a `%mass` `gift` containing Ames'
current memory usage.

#### Source

```hoon
  ++  on-wegh
    ^+  event-core
    ::
    =+  [known alien]=(skid ~(tap by peers.ames-state) |=(^ =(%known +<-)))
    ::
    %-  emit
    :^  duct  %give  %mass
    :+  %ames  %|
    :~  peers-known+&+known
        peers-alien+&+alien
        dot+&+ames-state
    ==
```

