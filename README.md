condir-lasp
===========

The aim of condir-lasp (convergent directory distribution) to try lasp as a
framework for CRDT.

Fixes:
------

The partisan module of lasp is outdated some fixes had to be made to work on erlang/OTP 21:
 * ssl:ssl_accept -> ssl:handshake
 * net_kernel:connect -> net_kernel:connect_node

Build
-----

    $ rebar3 compile

Usage
-----

I tried the AWMap CRDT example from the [blog-post]:

```erl
Key1 = <<"key1">>.
Key2 = <<"key2">>.
Timestamp = fun () -> erlang:unique_integer([monotonic, positive]) end.

AwMapType = {state_awmap, [state_mvregister]}.
AwMapVarName = <<"awmap">>.
AwMapVal1 = foo.
AwMapVal2 = bar.

% register
{ok, {AwMap, _, _, _}} = lasp:declare({AwMapVarName, AwMapType}, AwMapType).
% update
{ok, {AwMapNew, _, _, _}} = lasp:update(AwMap, {apply, Key1, {set, Timestamp(), AwMapVal1}}, self()).

%read
{ok, AwMapRes} = lasp:query(AwMap).
% The AwMapRes will contain Key1 because lasp:query follows the changes
[{K, sets:to_list(V)} || {K, V} <- AwMapRes].
```

Callback functions can be registered to the registered container:

```erl
Function = fun(AwMapRes) -> [{K, sets:to_list(V)} || {K, V} <- AwMapRes] end,
lasp:stream({<<"set">>, state_orset}, Function).
```

Nodes can be connected, however the api is broken and partisan function has to
be calle directly:

```
(mynode@localhost)> lasp_peer_service:join(othernode@localhost).
Here comes the synchronized  data manipulation...
(mynode@localhost)> partisan_peer_service:leave(othernode@localhost).
```

I could not setup lasp_pg, however the dynamic process group would be nicer
than the peer to peer node connection.

 [blog-post]: http://marianoguerra.org/posts/playing-with-lasp-and-crdts.html
 [tutorial-read]: https://lasp-lang.readme.io/docs/reading-variables
 [tutorial-pg]: https://lasp-lang.readme.io/docs/what-is-lasp-pg
