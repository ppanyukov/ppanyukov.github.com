---
layout: post
title: Le C# Grit
---

# Le C# Grit #

This is a growing list of my annoyances with C# which emerged after my exploration of other languages, in particular F#.

# Functions are not first class citizens #

There are many, many times when I want to have a nice function which does
something but I need or want to break it into several smaller functions.
In C# it often goes like this:

    class Foo_Traditional
    {
        public object GetStuff(object input)
        {
            object temp_A = this.GetStuff_Helper_A(input);
            object temp_B = this.GetStuff_Helper_B(temp_A);
            return temp_B;
        }

        private object GetStuff_Helper_A(object input)
        {
            return input.ToString() + "/GetStuff_Helper_A";
        }

        private object GetStuff_Helper_B(object input)
        {
            return input.ToString() + "/GetStuff_Helper_B";
        }
    }

This is not nice because `GetStuff_Helper_A` and `GetStuff_Helper_B` are used
only by one method but are now exposed as the class-level methods. What I want
is to make those helper methods private to the implementation of `GetStuff`.

OK, let's rewrite this as lambdas.

    class Foo_Lambdas_attempt_1
    {
        public object GetStuff(object input)
        {
            // Error here: can't declare variable "input" as it's already declared!
            Func<object, object> get_stuff_helper_a = (input) =>
                {
                    return input.ToString() + "/GetStuff_Helper_A (lambda)";
                };

            // Error here: can't declare variable "input" as it's already declared!
            Func<object, object> get_stuff_helper_b = (input) =>
            {
                return input.ToString() + "/GetStuff_Helper_B (lambda)";
            };

            object temp_A = get_stuff_helper_a(input);
            object temp_B = get_stuff_helper_b(temp_A);
            return temp_B;
        }
    }

Two things to notice here:

1.  It's not just a matter of copy/paste of class-level members. We must do
    some work to convert them to lambdas. And this involves declaring things
    like `Func<object,object>`.

2.  We can't use the same variable name `input` as the parameter for the lambda
    because it's already delcared in outer scope. But what do we do? What if it
    makes perfect sense to call that variable `input`?

There are two solutions to the latter, none of which make things any easier:

1.  Rename `input` to something else like `input1`; or

2.  Remove the parameter from the lambda and use the one that's already
    declared. This will mean re-writing the `Func<object,object>` declaration
    into `Fun<object>` and removing the parameters. More work!

OK, I prefer to rename the parameters and end up with this:

    class Foo_Lambdas_attempt_2_which_compiles
    {
        public object GetStuff(object input)
        {
            // OK now, renamed input to input2
            Func<object, object> get_stuff_helper_a = (input2) =>
            {
                return input2.ToString() + "/GetStuff_Helper_A (lambda)";
            };

            // OK now, renamed input to input2
            Func<object, object> get_stuff_helper_b = (input2) =>
            {
                return input2.ToString() + "/GetStuff_Helper_B (lambda)";
            };

            object temp_A = get_stuff_helper_a(input);
            object temp_B = get_stuff_helper_b(temp_A);
            return temp_B;
        }
    }


Now lets say I need a helper function which returns a `IEnumerable<T>`. I go an
try to enhance my function with this helper lambda:

    class Foo_with_enumerables
    {
        public object GetStuff(object input)
        {
            Func<IEnumerable<string>> get_stuff_helper_c = () =>
                {
                    // ERROR! Only methods, operators and accessors could contain 'yield' statement.
                    yield return "this sucks";
                };

            ...
        }
    }

And why the hell can't I have the lambda which returns an `IEnumerable<T>`???
What do I do now? Back to the class-level method I guess. 

THIS SUCKS.

### The F# way ###

Here is the same starting point, with main function and helpers being separate:

    // Starting point
    let get_stuff_helper_a input = 
        input.ToString() + "/GetStuff_Helper_A (F#)"

    let get_stuff_helper_b input = 
        input.ToString() + "/GetStuff_Helper_A (F#)"

    let DoStuff input = 
        let temp_A = get_stuff_helper_a input
        let temp_B = get_stuff_helper_b temp_A
        temp_B

Notice one thing immediately: there is much less clutter to start with.

Now let's hide those helper functions.

    // Hide the helper functions
    let DoStuff input = 
        let get_stuff_helper_a input = 
            input.ToString() + "/GetStuff_Helper_A (F#)"

        let get_stuff_helper_b input = 
            input.ToString() + "/GetStuff_Helper_A (F#)"

        let temp_A = get_stuff_helper_a input
        let temp_B = get_stuff_helper_b temp_A
        temp_B


Wow, just cut and paste them to be inside `DoStuff`, no change required!

And now lets see if we can add helper function which returns `IEnumerable<T>`.

    // Add helper function returning IEnumerable<T>
    let DoStuff input = 
        let get_stuff_helper_a input = 
            input.ToString() + "/GetStuff_Helper_A (F#)"

        let get_stuff_helper_b input = 
            input.ToString() + "/GetStuff_Helper_A (F#)"

        let get_stuff_helper_c () = seq {
            yield "this rocks!"
        }

        let temp_A = get_stuff_helper_a input
        let temp_B = get_stuff_helper_b temp_A
        temp_B

It works! And, incidentally, the function `DoStuff` is automatically generic:

    'a -> string

which translates in C# terms as

    string GetStuff<T>(T input)

In C# I would have to explicitly make the function generic (which is another story).

Lets summarise. In F# we have:

1.  Cut/paste functions without changes. No can do in C#.
2.  Re-bind different value to same name (aka no need to rename parameter). No can do in C#.
3.  Can have _any_ function within function, including those returning `IEnumuerable<T>`. No can do in C#.
4.  Functions are generic by default. Not in C#.
5.  Code seems to be much more concise and readable. C# has too much clutter, it's too verbose.


------------------------------------------------

# Can't assign lambdas to implicit variables #

Scanario. I want to write something like this:

    // C# -- This doesn't work
    var someFun = (x) => 
    {
        if (x > 0)
            return true;
        else
            return false;
    };

No can do! Gives me a compiler error: _error CS0815: Cannot assign lambda expression to an implicitly-typed local variable_.

Instead I have to do the full type specification:

    // C# -- This works
    Func<int,bool> somFun = (x) =>
    {
        if (x > 0)
            return true;
        else
            return false;
    };

It might be OK if the types in use are simple like above, but what if they are not? For example:

    // C# -- really bad
    Fun<Map<string,DisposableWrapper<_MailItem>>, Tuple<DisposableWrapper<_MailItem>>> someFun2 = (stuffIn) =>
    {
        ...
    }


    
THIS SUCKS.

### The F# way ###

And here is F#:

    // F# -- this works
    let someFun x = 
        if x > 0 then 
            true 
        else 
            false



