---
title: "Of functions and design patterns"
date: 2024-05-24T18:45:01+02:00
---

Dear reader,

I have been thinking about functions and design patterns and how they resemble one another. They emerge similarly and they can be powerful communication tools. What differentiates them and why should we care?

## Stumbling upon a function

Let's start by examining some code written in Rust. With it, a chipmunk is keeping track of the nuts surplus it gathers every month.

``` rs
// January
let nuts_found = 17;
let nuts_needed_this_month = 62;
let nuts_surplus = {
    if nuts_found > nuts_needed_this_month {
        nuts_found - nuts_needed_this_month
    } else {
        0
    }
};
println!("On January, we have a surplus of {nuts_surplus} nuts.");

// February
let nuts_found = 72;
let nuts_needed_this_month = 56;
let nuts_surplus = {
    if nuts_found > nuts_needed_this_month {
        nuts_found - nuts_needed_this_month
    } else {
        0
    }
};
println!("On February, we have a surplus of {nuts_surplus} nuts.");
```

We see some repetition; a pattern stands out. The same logic is used for each month. The lines that are harder to read are the nuts surplus computation. Let's help the chipmunk out and make its code a bit clearer. Let's extract the nuts surplus computation into a *function*. 

``` rs
// January
let nuts_found = 17;
let nuts_needed_this_month = 62;
let nuts_surplus = compute_nuts_surplus(nuts_found, nuts_needed_this_month);
println!("On January, we have a surplus of {nuts_surplus} nuts.");

// February
let nuts_found = 72;
let nuts_needed_this_month = 56;
let nuts_surplus = compute_nuts_surplus(nuts_found, nuts_needed_this_month);
println!("On February, we have a surplus of {nuts_surplus} nuts.");

fn compute_nuts_surplus(nuts_found: i32, nuts_needed_this_month: i32) -> i32 {
    if nuts_found > nuts_needed_this_month {
        nuts_found - nuts_needed_this_month
    } else {
        0
    }
}
```

A useful definition of a function might be: a function is a code section contained in a reusable unit. We could write a function from scratch with a bit of foresight, but in our case, we discovered a pattern and extracted it into a function.

Sure, it could still use some improvement, but the code is now a lot easier to inspect. Let's focus on the nuts surplus computation:

``` rs
fn compute_nuts_surplus(nuts_found: i32, nuts_needed_this_month: i32) -> i32 {
    if nuts_found > nuts_needed_this_month {
        nuts_found - nuts_needed_this_month
    } else {
        0
    }
}
```

What is this function doing? From the point of view of the caller, it is computing the nuts surplus. The ingredients for the computation are the number of nuts that have been found and the number of nuts needed in the current month.

How is this function implemented? If the number of nuts found is greater than the number of nuts needed, their difference is returned. Otherwise, zero is returned. This sort of rings a bell... This is equivalent to taking the maximum between zero and the difference between nuts found and nuts needed!

We recognized this because it is part of our vocabulary. There is probably a `max` function in every programming language or standard library and after coding for a while, we are almost guaranteed to run into it. Let's exploit the popularity of the `max` function:

``` rs
fn compute_nuts_surplus_max(nuts_found: i32, nuts_needed_this_month: i32) -> i32 {
    std::cmp::max(0, nuts_found - nuts_needed_this_month)
}
```

By using the `max` function of Rust's standard library, our implementation became more concise and easier to understand! It is easier to understand *because* `max` is so well-known and widespread.

## Stumbling upon a design pattern

The chipmunk comes back to us with a new version of its code. The chipmunk realized that the variables `nuts_found`, `nuts_needed_this_month`, and `nuts_surplus` are related. The new structure `MonthlyNutsInventory` expresses this relationship naturally, the chipmunk argues. It goes on to elaborate on how this better communicates the intention of the code to the reader. Well, all that sounds very convincing to me, let's take a look at the code!

The new code is presented in two sections. First, a structure definition:

``` rs
struct MonthlyNutsInventory {
    pub found: i32,
    pub needed: i32,
}

impl MonthlyNutsInventory {
    pub fn surplus(&self) -> i32 {
        std::cmp::max(0, self.found - self.needed)
    }

    pub fn add_nuts(&mut self, new_nuts: i32) {
        self.found += new_nuts;
    }
}
```

We see the structure `MonthlyNutsInventory` with two associated functions, `surplus` and `add_nuts`. The `surplus` function calculates the current nuts surplus. The `add_nuts` function helps the chipmunk log newly found nuts into the monthly inventory.
Let's see how the chipmunk used this structure in the second section of the code:

``` rs
let mut january = MonthlyNutsInventory {
    found: 0,
    needed: 62,
};

january.add_nuts(10);
january.add_nuts(17);

println!(
    "On January, we have a surplus of {0} nuts.",
    january.surplus()
);

let mut february = MonthlyNutsInventory {
    found: 0,
    needed: 56,
};

february.add_nuts(17);
february.add_nuts(10);
february.add_nuts(17);
february.add_nuts(8);
february.add_nuts(20);
println!(
    "On February, we have a surplus of {0} nuts.",
    february.surplus()
);
```

For a given month, an inventory is initialized, nuts are logged when they are found and the `surplus` function is used to compute the nuts surplus at the end of the month.

We can happily report to the chipmunk that its code does communicate its intention more easily. However, two things could see some improvement. 

- There is no reason for the `found` field to be public. 
- It should not be the responsibility of the user of the `MonthlyNutsInventory` structure to initialize the `found` field when initializing the structure. The user is forced to provide zero as the initial value for the field `found` for each month. This is error-prone.

To solve these two issues, let's make the `found` field private and create a function that will only take the value of the `needed` field as input. Since this function will need to initialize the value of the private `found` field, it cannot be a free function, it needs to be associated with the `MonthlyNutsInventory` structure.

``` rs
struct MonthlyNutsInventory {
    found: i32,
    needed: i32,
}

impl MonthlyNutsInventory {
    pub fn surplus(&self) -> i32 {
        std::cmp::max(0, self.found - self.needed)
    }

    pub fn add_nuts(&mut self, new_nuts: i32) {
        self.found += new_nuts;
    }

    pub fn create(needed: i32) -> Self {
        Self { found: 0, needed }
    }
}
```

The only modification required in the second section of the code is the initialization of the MonthlyNutsInventory instances with our new `create` function.

``` rs
let mut january = MonthlyNutsInventory::create(62);

january.add_nuts(10);
january.add_nuts(17);

println!(
    "On January, we have a surplus of {0} nuts.",
    january.surplus()
);

let mut february = MonthlyNutsInventory::create(56);

february.add_nuts(17);
february.add_nuts(10);
february.add_nuts(17);
february.add_nuts(8);
february.add_nuts(20);
println!(
    "On February, we have a surplus of {0} nuts.",
    february.surplus()
);
```

The initialization looks cleaner now since we don't need to worry about the `found` field of the `MonthlyNutsInventory` structure. This avoids dangerous miscalculations of nuts surplus during winter.

Let's take a step back and see what happened. Adding the `create` associated function allowed us to protect the member variables of the `MonthlyNutsInventory` structure; The structure `MonthlyNutsInventory` no longer exposes its internal representation. This corresponds to a design pattern called the **Static Factory Method**. It is **static** because it is invoked by the structure, no instance of the structure is required. It is called a **factory** because it creates a new instance of the structure and returns it. A **method** or associated function is used so that the private internal representation of the structure can be initialized.

Now that we are familiar with the **Static Factory Method**, we may use it if we encounter the same situation again. A useful definition of a design pattern might be: a design pattern is a solution to a software design problem. In this case, the problem was a needlessly public attribute and the need to initialize it. The solution was using a static factory method.

The name `create` is perfectly fine. It describes clearly and concisely what the function is doing. However, Rust programmers might be scratching their heads over this choice. There is a very well-known alternative: `new`. It is a simple change of name, but it will help Rust developers understand the intention more quickly because we are now adopting standard vocabulary.

## Functions and design patterns

We proposed two main modifications to the chipmunk's code. In the first modification, we proposed to use a function, and in the second one, a design pattern. Part of the implementation of the design pattern used a function. So, are design patterns equivalent to functions?

Design patterns are implemented with functions or other mechanisms. A few of the simpler design patterns might be implemented with a single function, but they are often more complex.

Functions serve a very general purpose: code reuse. They have important positive consequences. They improve readability and maintainability. They also make code easier to test.

Design patterns are more specific. Each design pattern offers a solution to a **particular problem** in a given **context**. In our example, the context is set by a characteristic of the programming language: the existence of public and private data. The problem was the exposure of the internal representation of a structure.

Functions and design patterns are useful abstractions. Adopting the associated vocabulary improves communication between developers. It helps us share and reuse knowledge. Functions and design patterns allow us to be more efficient and build better software more quickly when used adequately.
