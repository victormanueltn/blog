---
title: "Design patterns you know (but might not be aware of)"
date: 2025-07-16T18:45:01+02:00
draft: false
---

Dear reader,

As software grows complex and care is given to its structure, [design patterns](https://victormanueltn.github.io/codingletters.github.io/posts/functions_and_design_patterns/) will appear. Sometimes they will be used intentionally and sometimes inadvertently. We might intuitively reach for patterns in our mental repertoire, or we might reinvent a long-known solution for a seemingly new problem. Regardless of our awareness of them, design patterns will show up.

If we strive to be intentional about the structure of our software, we will be better equipped by being able to recognize design patterns. A good familiarity with design patterns will allow us to willingly invoke them when required. When designing software, we will have a richer vocabulary, more colors in our palette. 

To train ourselves in this direction, I would like to mention three design patterns you have probably seen but might have not noticed. They are hiding in plain sight in the C++ standard library.


## The iterator pattern

I had known the iterators in the C++ standard library long before I started learning about design patterns. I was so used to iterators, that I experienced disbelief when I first heard that they were a design pattern. They are.

Thanks to the iterator pattern, different collections can be traversed with the same syntax. It doesn't matter if the collection is a hash map or a dynamic array, if there is an iterator available, the iterator will be used in the same way.

There are many ways to iterate over a collection in C++. One of these makes the use of iterators most explicit: 

``` cpp
std::vector<int> items{1, 2, 3, 4, 5};
for(auto iterator = items.begin(); iterator != items.end(); iterator++) {
    std::cout << *iterator << '\n';
}
```

## The adapter pattern

Did you know that the stack and the queue data structures in the C++ standard library are implemented with the same underlying data structure? If you did, you already knew two examples of the adapter pattern.

The stack and the queue are implemented with a double-ended queue (or deque). This becomes evident in their second template argument when comparing their declaration side by side:

``` cpp
template<
    class T,
    class Container = std::deque<T>
> class stack;

template<
    class T,
    class Container = std::deque<T>
> class queue;
```

The adapter pattern is used to expose the simpler and well-known interfaces of the stack and the queue data structures. This makes them easier to reason with when solving problems that are elegantly expressed with stacks or queues.

## The strategy pattern

The `std::accumulate` function from the `numeric` header iterates over a range applying a binary operation to subsequent elements to ultimately return a single value. Which binary operation, you ask? Thanks to the strategy pattern, you decide:

``` cpp
template<class InputIterator, class T, class BinaryOperation>
T accumulate(
    InputIterator first,
    InputIterator last,
    T init,
    BinaryOperation operation
);
```

The use of the strategy pattern makes `std::accumulate` versatile. It allows reusing the iteration logic for any binary operation.

## Are there more?

There are many more in the standard C++ library. Can you spot some of them? Klaus Iglberger went through some of them in his talk [Design Patterns: Facts and Misconceptions](https://www.youtube.com/watch?v=OvO2NR7pXjg). Be sure to check it out!
