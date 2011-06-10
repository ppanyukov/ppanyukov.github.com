# Esoteric languages #

-   [brainfuck](http://en.wikipedia.org/wiki/Brainfuck)
-   [LOLCODE](http://en.wikipedia.org/wiki/LOLCODE)

## brainfuck ##

The following program prints "Hello World!" and a newline to the screen:

 
    +++++ +++++             initialize counter (cell #0) to 10
    [                       use loop to set the next four cells to 70/100/30/10
        > +++++ ++              add  7 to cell #1
        > +++++ +++++           add 10 to cell #2 
        > +++                   add  3 to cell #3
        > +                     add  1 to cell #4
        <<<< -                  decrement counter (cell #0)
    ]                   
    > ++ .                  print 'H'
    > + .                   print 'e'
    +++++ ++ .              print 'l'
    .                       print 'l'
    +++ .                   print 'o'
    > ++ .                  print ' '
    << +++++ +++++ +++++ .  print 'W'
    > .                     print 'o'
    +++ .                   print 'r'
    ----- - .               print 'l'
    ----- --- .             print 'd'
    > + .                   print '!'
    > .                     print '\n'


## LOLCODE ##

### Example 1 ###
    HAI
    CAN HAS STDIO?
    VISIBLE "HAI WORLD!"
    KTHXBYE

### Example 2 ###
     HAI CAN HAS STDIO?
     PLZ OPEN FILE "LOLCATS.TXT"?
         AWSUM THX
             VISIBLE FILE
         O NOES
             INVISIBLE "ERROR!"
     KTHXBYE

### Example 3 ###
    HAI
    CAN HAS STDIO?
    I HAS A VAR
    IM IN YR LOOP
       UP VAR!!1
       VISIBLE VAR
       IZ VAR BIGGER THAN 10? KTHX
    IM OUTTA YR LOOP
    KTHXBYE


