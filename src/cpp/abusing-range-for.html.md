---
title: (Ab)using range-based for
---

Range-based for loops are syntactic sugar over a traditional loop with iterators.
They make the previous idiom simpler and also enable some new ones.

## Loop through integers, Python-style

No one would dare use `boost::irange` in a traditional loop. But with the new loop, it's not that unwieldy:

```
for(int i : boost::irange(0, 10)) {
    std::cout << i << '\n';
}
// prints: 0 1 2 3 4 5 6 7 8 9 
```

... or in reverse:

```
for(int i : boost::irange(0, 10) | boost::adaptors::reversed) {
    std::cout << i << ' ';
}
// prints: 9 8 7 6 5 4 3 2 1 0 
```

## To loop or not to loop

`boost::optional` can be seen as a container that holds at most one element.
So, why not loop over it? Just provide appropriate iterators and `begin` and `end` functions.

```
boost::optional<T> o = 42;
for(auto&& t : o) {
   frob(t);
   schizzle(t);
}
```

The code in the loop would run only when the optional holds a value (and only once), and `t` is automatically bound to it.

## RAII

One interesting property of the range-based loop is that, if the range is a temporary,
its lifetime is the whole loop (all iterations). This allows us to throw RAII semantics into mix.
And what interesting things can we do with this? [Naturally enforced and properly scoped locks](http://ideone.com/HK4Kw)!

The idea is to, instead of keeping a mutex and the shared data it protects both visible and separate,
we hide the data away and only reveal it when the mutex is acquired. We write a class with the mutex
and data as private members and add an `open` function that acquires the mutex and returns an object that releases it upon destruction.

Add the appropriate iterators to that, and we can write the following:

```
locker_box<racy> box; // shared data is not accessible without locking
try {
    for(auto&& x : box.open()) {  // lock is acquired, and is held only while the loop runs
                                  // and the shared data becomes available through x
        x.do_racy_stuff();
        x.do_more_racy_stuff();
        x.do_really_raunchy_stuff();
    }
} catch(not_suitable_for_this_audience const&) {}
```
