Here are two tests for checking if a number is divisible by 7

### Method 1

I read this method in a school textbook several years ago and just used it
without caring about a proof.

> Double the last digit of the number and subtract it from the remaining number.
If the resulting number is divisible by 7, the original number was divisible by
7 too.

Example: to check if 343 is divisible by 7
343 -> 34 - 2 * 3 = 28
Since 28 is divisible by 7, 343 is divisible too.

### Method 2

Came across the 2nd method on Twitter and this prompted me to try
and come up with a proof. The test goes like this:

> Take the last digit of the number and multiply it with 5. Add it
to the rest of the number. If the resulting number is divisible by 7,
then the original number was divisible by 7 too.

Example: 343 -> 34 + 3 * 5 = 49
Since 49 is divisible by 7, 343 is divisible by 7.

### Proof

The proof is simple and relies on the fact that the divisibility
does not change when the number is multiplied by another number
that is not a multiple of 7.

* Let the number be (10 x + y). If it is divisible by 7 then
* 5 * (10 x + y) is divisible by 7
* (50 x + 5 y) is divisible by 7
* 49 x + (x + 5 y) is divisible by 7
* (x + 5 y) is divisible by 7
