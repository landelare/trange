# Trange (working title)

This repo contains the **beta** pre-release version of a single-file,
header-only library that adapts Unreal Engine containers and views to conform to
the C++ standard [range concept](https://en.cppreference.com/w/cpp/ranges/range).

Wrap your containers in the very cheap (`sizeof(void*)` and a very good inlining
candidate) `trange::view` and enjoy all the functionality of STL ranges,
the Ranges-v3 library, and anything else relying on the standard concepts.
Both the function call and "pipe" syntax are available.

`elements<N>`, `keys`, `values` are also provided to work around TTuples
(returned by TMap, etc.) not being tuple-like.
Use these as drop-in replacements instead of, e.g., `std::views::keys`.

## Examples

Iterate over only valid entries in a TArray:
```c++
TArray<UObject*> Array;
for (UObject* Obj : trange::view(Array) | std::views::filter(IsValid))
    check(IsValid(Obj)); // ✅
```

Add up all the elements of an array (also using Ranges-v3 and Boost.Lambda2 for
the short lambda syntax):
```c++
TArray<float> Array;
float Sum = ranges::accumulate(Array | trange::view, 0, _1 + _2);
```

Iterate over an array with the other providing indices:
```c++
TArray<int> Indices{2, 1, 0, 4, 3};
TArray<int> Values{10, 20, 30, 40, 50};
for (auto Value : Indices | trange::view
                          | std::views::transform([&](int i){return Values[i];}))
    ; // Value will be 30, 20, 10, 50, 40
```

Give support to all custom actors that need it:
```c++
for (AMyActor* Actor : trange::view(TActorRange<AMyActor>(GetWorld()))
                     | std::views::filter(&AMyActor::NeedsSupport))
    Actor->ReceiveSupport();
```
