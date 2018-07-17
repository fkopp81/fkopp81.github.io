---
Date: '2018-07-11 15:49 +0200'
published: true
title: Regular expression 101 - writing a versatile number matcher
tags: regular-expressions
---
## Regular expressions

Regular expressions are a powerful tool for validating user input. For the uninitiated they also look like complete gibberish. Sites like [regexr](https://regexr.com) allow you to analyse existing expressions, but writing new ones from scratch that do exactly what you want them to can still be hard.

To help out colleagues looking to validate entries for a computer based math test, I set out to build a regular expression for detecting numbers.

The first baby steps are easy. To check if the user wrote exactly "1" the regular expression is:

> /^1$/

You have two slashes that surround your rule. ^ and $ are special characters: ^ means the beginning of the input and $ the end. And the 1 in the middle is the 1 you are looking for. The official lingo is that your regular expression "matches" an input of 1.

But what if you want your expression to match any digit?

> /^[0123456789]$/

The \[\] are used to surround a group of characters to match. Since regular expressions understand ranges you can shorten the above to

> /^[0-9]$/

Or even shorter:

> /^\\d$/

\\d is special regular expression shorthand for [0-9] (d for digit). Congratulation, you are now initiated on the ground level. The above expressions all match any one digit, so they are great if the user enters "1", "5" or "0", but they don't match longer numbers - or an empty input, if the user doesn't want to answer at all.

> /^\\d?$/

Enter the quantifiers: These special characters are used to signify how often the part directly in front of it is matched. That might sound complicated, so let's just go through a few examples: In the one above the  "?" is a quantifier that makes the part directly in front of it optional - so now the regular expression matches an empty input or a single digit. There's also

> /^\\d+$/

The + quantifiers means that the before part needs to be there once or multiple times, so it would match "01234" or "110", but not "". For that there's

> /^\\d*$/

With the * quantifiers the before part is matched any number of times including not at all, so "" is okay now.

Finally you can constrain how many digits you want with curly braces.

> /^\\d{0,3}$/

The first number in the curly braces is the least amount of digits we want to match (0 so empty inputs are okay), and the second number is the maximal amount (3 in the above example). So "", "1" and "999" are okay.

Of course there's more to numbers than digits, there are negative numbers and decimals. To cover all of our bases, let's include the necessary additional characters for them:

> /^[\\d,.-]*$/

The above expression matches "1.0", "-2,5" and ",7". Great! We are done now, right? Wrong! Because by this point we can do some very un-numerical shenanigans. Is "-.-" a number? Or "007"? How about "1,2.3,4"? Clearly we need to step up our game!

We need two more regular expression features to put it all together. The first one, "\|" signifies an "either or" match:

> /^a|b$/

This means the above matches "a" or "b", but not "" or "ab" (and "ba" is right out).

The other feature we need is the caption group, signified by "()". We use it to group a more complex sub-expression together, so we can write:

> /^(a|b)?c$/

Relax, you can get through this with what you've learned so far. The caption group matches either "a" or "b". It's followed by "?", so it's optional. The "c" following that is standing on its own, so it's required. Altogether it matches "ac", "bc" or just "c", but not "", "ca" or anything else.

Alright, back to numbers. First, let's stop 007 from ruining our evil plots:

> /^(0|[1-9]\d{0,3})?$/

What's this then? The whole capture group is optional, so "" is okay. Inside the group, it either matches "0" or the second part. That starts with [1-9], so all digits except 0, and is followed by \\d{0,3} - zero to three times any digit. So "0" is okay, just like "123" "9001" or "42" - but "0123" is not. (Wait - "9001"? But we got {0,3} in our expression, so shouldn't it match three digits at most? Yes, but the curly braces latch onto what's right in front of it - in this case the \\d. So the \[1-9\] matches any digit from 1 to 9 exactly once, followed by up to three more digits.)

Let's get real now - as in real numbers with decimal places.

> /^(0|[1-9]\d{0,3})?([,.]\d{0,2})?$/

I just copy-pasted the first capture group from above. The new part is "([,.]\\d{0,2})?" Easy, right? First we have \[,.\], so either a comma or full stop (but not both) followed by 0-2 digits. So ",5" is allowed (since the first capture group is optional because of the ? behind it), and so is "0,5", "2,0" and "5." Wait, "5."? It's not nice but necessary for my specific use case, because the input is validated and rejected on the fly by the software my colleagues use. If "5," was not valid, I couldn't enter the ",", even if I wanted to actually write "5,5". I could type in "55" and then insert a comma in-between the digits, but it's a math test, not a problem solving one.

Okay, we are almost done, and the last bit is anti-climactic, too:

> /^-?(0|[1-9]\d{0,3})?([,.]\d{0,2})?$/

Can you spot it? The "-?" at the beginning is new. All it says is that the whole input can optionally be prefaced by a "-", so it now also matches negative numbers! So "-9999", "-,5" "-0" all are allowed now. Well, "-0" is a bit silly (unless you are a Javascript developer, then it's the stuff of nightmares), but allowing it doesn't do much harm and comes with an advantage: We can view our regular expression as put together from three independably configurable segments.

## Configuring the regular expression
### Change the number of allowed digits
Change the second number in either of the curly braces. For predecimals keep in mind that one more digit is matched, because the curly braces don't constrain the initial \[1-9\].

> /^-?(0|[1-9]\d{0,1})?([,.]\d{0,1})?$/

So now any number between -99,9 and 99,9 is matched.

### Don't be negative
Just remove the "-?" part to only allow positive numbers

> /^(0|[1-9]\d{0,1})?([,.]\d{0,1})?$/

The above only matches 0 to 99,9 now.

### Natural numbers only

Remove the second capture group (including the ?) to stop matching decimals:

> /^(0|[1-9]\d{0,1})?$/

And all that's left is 0 to 99.

And of course you can also remove the "0\|" to stay positive.

> /^([1-9]\d{0,1})?$/

And voila, 1 to 99 is all that is still matched.

I hope you found this little introdcution to regular expressions helpful.
