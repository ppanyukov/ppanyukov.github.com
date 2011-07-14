# Why C# sucks #

This is a growing list of my annoyances with C# which emerged after my exploration of other languages, in particular F#.


## Can't assign lambdas to implicit variables ##

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

And here is F#:

        // F# -- this works
        let someFun x = 
            if x > 0 then 
                true 
            else 
                false



