A simple Erlang client for the [Quantum Random Bit Generator](http://random.irb.hr/) service.  Example usage:
```
Eshell V5.5.4  (abort with ^G)
1> c(qrbg).
{ok,qrbg}
2> {ok, Socket} = qrbg:connect().
{ok,#Port<0.127>}
3> Response = qrbg:get_response(Socket, "username", "password").
<<0,0,0,0,16,0,230,132,193,235,0,254,163,8,239,180,51,164,169,160,170,248,94,
  132,220,79,234,4,117,...>>
4> {ok, _Response, _Reason, _Length, Data} = qrbg:extract_data(Response).
{ok,0,
    0,
    4096,
    <<230,132,193,235,0,254,163,8,239,180,51,164,169,160,170,248,94,132,220,
      79,234,4,117,248,...>>}
5> {Int, RestData} = qrbg:extract_int(Data).
{-427507221,
 <<0,254,163,8,239,180,51,164,169,160,170,248,94,132,220,79,234,4,117,248,174,
   59,167,49,165,170,154,...>>}
6> Int.
-427507221
7> {Int2, MoreRestData} = qrbg:extract_int(RestData).
{16687880,
 <<239,180,51,164,169,160,170,248,94,132,220,79,234,4,117,248,174,59,167,49,
   165,170,154,146,102,164,89,...>>}
8> Int2.
16687880
```

As of [r4](https://code.google.com/p/qrbgerl/source/detail?r=4), I've also included the beginnings of a drop-in replacement for the random number generation in the [Crypto API](http://erlang.org/doc/man/crypto.html):
```
Eshell V5.5.4  (abort with ^G)
1> c(qrbg).
{ok,qrbg}
2> crypto:start().
ok
3> crypto:rand_bytes(8).
<<53,61,2,251,238,32,128,17>>
4> qrbg:rand_bytes(8).
<<82,180,139,199,18,246,153,132>>
```

This requires you to add your username and password to  [qrbg.hrl](http://qrbgerl.googlecode.com/svn/trunk/qrbg.hrl) and is extremely inefficient if you're only looking for a few bytes.  A fresh request is made for each call to qrbg:get\_bytes/1, so be gentle.  If you need several sets of random bytes you'll want to cache the data and extract bytes from it.

It goes without saying that in its current form, this client is susceptible to a man in the middle attack, so don't use it for anything sensitive.  It's an excellent source for truly random numbers though, and the QRBG service developers would like to support SSL connections in the future.