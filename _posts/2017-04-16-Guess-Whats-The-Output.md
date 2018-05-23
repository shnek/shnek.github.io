---
layout: post
cover: 'assets/images/programming.png'
title: Guess the Output of this Code
date:   2018-04-16 10:18:00
tags: platform
subclass: 'post tag-test tag-content'
categories: jakubsynowiec
navigation: True
author: 'Jakub Synowiec'
nickname: jakubsynowiec
bio: "Certified ServiceNow Application Developer"
image: 'assets/images/jsynowiec.jpg'
logo: 'assets/images/jg_logo_white.png'
---

Over the time that I have been working with ServiceNow I have discovered several interesting quirks about the behavior of the code. Some of them kept me wondering for a while, others made me giggle. I want to share one of them with you, that I have discovered recently with Dylan, while debugging one of our applications, that left us with one question: Why does it work like that?

## Let's play the guessing game

Let's say you are writing a server script, and you need to parse some integers from strings. You use the `parseInt` method to convert the types. Can you guess what will be the result of this execution?

```var result = parseInt("8");
```

That's easy, it's gonna be `8`. But what if you want to parse a number that has an unnecessary zero at the beginning. Let's see this code:

```var result = parseInt("08");
```

Now here's where things get a little bit more tricky. The result of this is Not a Number global property `NaN`.

## Why?

Why does it return `NaN`? 08 is clearly a number, with one unnecessary zero at the beginning. Is it a bug, or is it a feature? I had to know why this method has this behavior.

First thing I did is testing this with my Google Chrome debugger console. Executing this code in this environment gave me a result I was predicing from the beginning, it returned `8`. That was the first hint, it had to be related to the implementation of this method for the Mozilla Rhino JavaScript engine. Digging a little bit deeper I found the answer. The parseInt method has two parameters, `string` and `radix`, of which the second one is optional, and it's documentation states:

*radix - An integer between 2 and 36 that represents the radix (the base in mathematical numeral systems) of the above mentioned string. Specify 10 for the decimal numeral system commonly used by humans. **Always specify this parameter** to eliminate reader confusion and to guarantee predictable behavior. Different implementations produce different results when a radix is not specified, usually defaulting the value to 10.*

That was my 'bingo' moment. Numbers that start with zero usually represent the octal system. I tried different numbers, all the digits from 0 to 9 with the additional zero before them, and only `08` and `09` did return `NaN`. It seems that I was right. This discovery allowed us to solve the bug in our code, and the following execution of the parseInt gives us result we were expecting:

```var result = parseInt("08", 10);
```

##Is there more?
Fixing this bug resulted in more questions that I had related to the `parseInt` method behavior. What will happen if I add more zeros to the number? Is `008` treaded same as `08`? I tried it:

```var result = parseInt("008");
```
The result is `0`. **What?**

Alright, I know that `07` did work, because it's a real number in the octal system. So, what will happen if I run this code?

``` var result = parseInt("007");
```
The result is `7`. Okay, so the conversion still works, and we can easily check if it's still the octal system. If we convert `10` from octal to decimal, it should return `8`. So let's try it with one, and two zeros before the actual number:

```var result = parseInt("010");
var other  = parseInt("0010");
```Both of these results return `8`. So it's still converting from octal to decimal system. To be absolutely sure, I verified that parsing `"008"` with the second, radix argument of `8`. The result is again: `0`. Is the eight being omitted? Let's try parsing something different. Can you guess what is the result of:

```var result = parseInt("018");
```
It is `1`. Yes. **One.** The conversion from octal system, that has a character that does not belong to this system still returns the number, as if the foreign character wasn't even there.

So what happens if I put there a character that belong to other numeric system than the decimal, like `'a'`? Let's try that:

```var result = parseInt("01a");
```
Yup, still `1`. So if the character at the end is ommited, what about multiple characters in several places?

```var result = parseInt("0a1aa");
```
Result? `0`. Wait, we expected one! Does it skip all the rest of the string, if it encounters the first non-octal character? The result of:

```var result = parseInt("001818");
```
is `1`.

It looks like it does skip all the remaining characters in the string, once it encounters the first one that is a non-octal character.

## Taking it a step further

Alright, I wanted to take it to the next level. I can put hexadecimal characters in the string, and it stills return zero. What if I use more than just hexadecmal character? Let's try this:

```var result = parseInt("0This will return zero.");
```
The result is, as expected, `0`. It does make sense to return zero, since we acknowledged that it will skip the rest of the string once it encounters a character from non-octal character set. But let's go back to the beginning. I started this post with what will `parseInt("08")` return. Do you remember what was the answer? `NaN`. So

```"0This will return a zero."
```
is a number, but

```"08"
```
is not. **WHAT?**

## Conclusion

JavaScript is an interesting language indeed. It has some quirks, that vary depending on their implementation. But at the end it gives us one lesson; if we, the developers, want to write reliable code, we should not leave the optional parameter if it varies on different platforms, test our code and always have some doubt in the correctness of the methods, even if they are implemented by the engine the code is run on.