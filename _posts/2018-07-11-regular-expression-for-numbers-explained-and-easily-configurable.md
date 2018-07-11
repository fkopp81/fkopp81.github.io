---
Date: '2018-07-11 15:49 +0200'
published: false
title: Regular expression for numbers explained and easily configurable
---
## Regular expressions

Regular expressions are a powerful tool for validating user input. For the uninitiated they also look like complete gibberish. Sites like [regexr](https://regexr.com) allow you to analyse existing expressions, but writing new ones from scratch that do exactly what you want them to can still be hard.

To help out colleagues looking to validate entries for a computer based math test, I set out to build a regular expression for detecting numbers.

The first baby steps are easy. To check if the user wrote exactly "1" the regular expression is:

> /^1$/

You have two slashes that surround your rule. ^ and $ are special characters: ^ means the beginning of the input and $ the end. And the 1 in the middle is the 1 I am looking for. But what if I want to look for any number?

> /^[0123456789]$/

The \[\] are used to surround a group of characters I am looking for. Since regular expression understands ranges I can shorten the above to

> /^[0-9]$/

Or I can shorten it even more to

> /^\\d$/

\\d is a special regular expression shorthand for [0-9] (d for digit). Congratulation, you are now initiated on the ground level. The above expressions validate any one number, so they are great if the user enters "1", "5" or "0", but it doesn't validate if the user doesn't want to answer at all or if they want to enter more than a single digit.

> /^\\d?$/

Enter the "?": This is a special character meaning that the part directly in front of it is optional - so now the regular expression validates an empty input or a single digit. There's also

> /^\\d+$/

+ means that the before part needs to be there once or multiple times, so "01234" or "110" would be fine, but not "". For that there's

> /^\\d*$/

With the * the before part can be there any number of times, so "" is okay now.

Finally we can constrain how many digits we want with curly braces.

> /^\\d{0,3}$/

The first number inside is the least amount of digits we accept (0 so empty inputs are okay), up to 3. So "", "1" and "999" are okay.

Of course there's more to numbers than digits, there are negative numbers and decimals. To cover all of our bases, let's include them:

> /^[\\d,.-]*$/

Great! We are done now, right? Wrong! Because by this point we can do some very un-numerical shenanigans. Is "-.-" a number? Or "007"? How about "1,2.3,4"? Clearly we need to up our game here!

We need two more regular expression features to put it all together. The first one, "|" signifies an "either or":

> /^a|b$/

This means "a" or "b" is valid, but not "" or "ab" (and "ba" is right out).

The other feature we need is the caption group, signified by "()". We use it to group a more complex sub-expression together, so we can write:

> /^(a|b)?c$/

Relax, you can get through this with what you learned so far. The caption group includes either "a" or "b" and is followed by "?", so it's optional. The "c" following that is just standing there, so it's expected. So the above matches "ac", "bc" or just "c", but not "", "ca" or anything else.

Alright, back to numbers. First, let's stop 007 from ruining our evil plots:

> /^(0|[1-9]\d{0,3})?$/

So, what's this then? The whole capture group is optional, so "" is okay. Inside the group, it can either be "0", or that second part. First there's [1-9], for the range of digits excluding 0, followed by \\d* or any number of any digit. So "0" is okay, just like "123" "9001" or "42" - but "0123" is not. (Wait - "9001"? But we got {0,3}, so shouldn't only three digits be allowed? Yes, but the {0,3} latches onto what's right in front of it - the \\d. So first we can have any number from 1 to 9 because of the \[1-9\] followed by up to three more digits.)

Let's get real now - as in real numbers with decimal places.

> /^(0|[1-9]\d{0,3})?([,.]\d{0,2})?$/

I just copy-pasted the first capture group from above. The new part is "([,.]\\d{0,2})?" Easy, right? First we have \[,.\], so either a comma or full stop, but not both, followed by 0-2 digits. So ",5" is allowed (since the first capture group is optional because of the ? following it), and so is "0,5", "2,0" and "5." Wait, "5." It's not nice but necessary for my specific use case, because the input is validated and rejected on the fly by the authoring software my colleagues use for the test that rejects input as soon as an invalid character is typed. If "5," was not valid, I couldn't enter the ",", even if I wanted to actually write "5,5". I could type in "55" and then insert a comma in-between the digits, but it's a math test, not a problem solving one.

Okay, we are almost done, and the last bit is anti-climactic, too:

> /^-?(0|[1-9]\d{0,3})?([,.]\d{0,2})?$/

Can you spot it? The "-?" at the beginning is new. All it says is that the whole input can optionally be prefaced by a "-", so we can now also have negative numbers! So "-9999", "-,5" "-0" all are allowed now. Well, "-0" is a bit silly (unless you are a Javascript developer, then it's the stuff of nightmares), but allowing it doesn't do much harm and comes with a big of advantage: We can view our regular expression as put together from three independably configurable segments.

## Configuring a regular expression
### Change the number of allowed digits
Change the second number in the curly braces. For predecimals keep in mind that one more digit is allowed, because the curly braces don't constrain the initial \[1-9\].

> /^-?(0|[1-9]\d{0,1})?([,.]\d{0,1})?$/

So now any number between -99,9 and 99,9 is allowed.

### Don't be negative
Just remove the "-?" part to only allow positive numbers

> /^(0|[1-9]\d{0,1})?([,.]\d{0,1})?$/

0 to 99,9 are valid now.

### Natural numbers only

Remove the second capture group (including the ?) to keep decimals out of the user input:

> /^(0|[1-9]\d{0,1})?$/

And all that's left is 0 to 99

Of course you can just remove the "0|" to stay positive.

> /^([1-9]\d{0,1})?$/

And voila, 1 to 99.