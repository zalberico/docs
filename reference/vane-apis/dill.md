+++
title = "Dill"
weight = 3
template = "doc.html"
+++

# Dill

In this document we describe the public interface for Dill. Namely, we describe
each `task` that Dill can be `pass`ed, and which `gift`(s) Dill can `give` in return.


## Tasks


### `%belt`

Every keystroke entered into the console triggers a `%belt` `task` for Dill,
which contains information about the keystroke, such as which key was pressed
and from which terminal. 

#### Accepts

`%belt` `task`s come in a number of different types, corresponding to different
keystrokes. You can find this list in `+dill-belt` in `zuse.hoon`.

```hoon
++  dill-belt                                         ::  new belt
    $%  {$aro p/?($d $l $r $u)}                         ::  arrow key
        {$bac ~}                                        ::  true backspace
        {$cru p/@tas q/(list tank)}                     ::  echo error
        {$ctl p/@}                                      ::  control-key
        {$del ~}                                        ::  true delete
        {$hey ~}                                        ::  refresh
        {$met p/@}                                      ::  meta-key
        {$ret ~}                                        ::  return
        {$rez p/@ud q/@ud}                              ::  resize, cols, rows
        {$txt p/(list @c)}                              ::  utf32 text
        {$yow p/gill:gall}                              ::  connect to app
    ==                                                  ::
```

#### Returns

Dill returns no `gift` in response to a `%belt` `task`.

#### Source

```hoon
$belt  (send `dill-belt`p.kyz)
```


### `%blew`

This `task` is called whenever the terminal emulator is resized, so that Dill can adjust
the text output to fit the new window size.

#### Accepts

```hoon
[p=@ud q=@ud]
```

`p` is the number of columns and `q` is the number of rows.

#### Returns

Dill returns no `gift` in response to a `%blew` `task`.

#### Source

```hoon
$blew  (send %rez p.p.kyz q.p.kyz)
```


### `%boot`

"Weird Dill boot"

#### Accepts

#### Returns

#### Source

```hoon
  ::  the boot event passes thru %dill for initial duct distribution
  ::
  ?:  ?=(%boot -.task)
    ?>  ?=(?(%dawn %fake) -.p.task)
    ?>  =(~ hey.all)
    =.  hey.all  `hen
    =/  boot
      ((soft $>($?(%dawn %fake) task:able:jael)) p.task)
    ?~  boot
      ~&  %dill-no-boot
      ~&  p.task
      ~|  invalid-boot-event+hen  !!
    =.  lit.all  lit.task
    [[hen %pass / %j u.boot]~ ..^$]
```


### `%crud`

#### Accepts

#### Returns

#### Source


### `%flog`

#### Accepts

#### Returns

#### Source

```hoon
  ::  %flog tasks are unwrapped and sent back to us on our default duct
  ::
  ?:  ?=(%flog -.task)
    ?~  hey.all
      [~ ..^$]
    ::  this lets lib/helm send %heft a la |mass
    ::
    =?  p.task  ?=([%crud %hax-heft ~] p.task)  [%heft ~]
    ::
    $(hen u.hey.all, wrapped-task p.task)
```


### `%flow`
 
#### Accepts

#### Returns

#### Source


### `%hail`

#### Accepts

#### Returns

#### Source


### `%heft`

#### Accepts

#### Returns

#### Source


### `%hook`

#### Accepts

#### Returns

#### Source


### `%harm`

#### Accepts

#### Returns

#### Source


### `%init`

#### Accepts

#### Returns

#### Source

```hoon
  ?:  ?=(%init -.task)
    ?>  =(~ dug.all)
    ::  configure new terminal, setup :hood and %clay
    ::
    =*  duc  (need hey.all)
    =/  app  %hood
    =/  see  (tuba "<awaiting {(trip app)}, this may take a minute>")
    =/  zon=axon  [app input=[~ ~] width=80 cursor=(lent see) see]
    ::
    =^  moz  all  abet:(~(into as duc zon) ~)
    [moz ..^$]

```

### `%knob`

#### Accepts

#### Returns

#### Source


### `%lyra`

#### Accepts

#### Returns

#### Source


### `%noop`

#### Accepts

#### Returns

#### Source


### `%pack`

#### Accepts

#### Returns

#### Source


### `%talk`

#### Accepts

#### Returns

#### Source


### `%text`

#### Accepts

#### Returns

#### Source


### `%trim`

#### Accepts

#### Returns

#### Source

```hoon
  ?:  ?=(?(%trim %vega) -.task)
    [~ ..^$]

```


### `%veer`

#### Accepts

#### Returns

#### Source



### `%vega`

#### Accepts

#### Returns

#### Source

```hoon
  ?:  ?=(?(%trim %vega) -.task)
    [~ ..^$]
```


### `%verb`

#### Accepts

#### Returns

#### Source
