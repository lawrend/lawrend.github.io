---
layout: post
title:      "total number of integers in an array with an even number of digits WOO!"
date:       2020-06-18 20:46:12 -0400
permalink:  count_the_digits_in_each_number_in_an_array
---


How many times does it happen? You are outsomewhere, and someone says, "I have all these number on this list but how many of them have an even number of digits? Who can help me??"

Soon, you can help. That will be AMAZING.

## The Problem

Given an array nums of integers, return how many of them contain an even number of digits.
 

### Example
```
arr = [12, 23, 1, 4223, 123, 64, 2]
# number of values with an even number of digits = 4; 
```

## Method #1--make that a string, please

You may think to take each value and:
* Convert it into string
* Determine the value of the `length` property
*  Check to see if that number is even
*  If so, add 1 to the tally


```
let evenCount = 0;
    for(n in nums) {
        let currentCount = 0;
        let digit = nums[n]; 
        currentCount = digit.toString().split('').length
        
        if(currentCount % 2 == 0) {
            evenCount += 1; 
        }
    }
    return evenCount;
```

That's certainly going to work. But there is a cooler way I found that makes total sense once you think about it...


## Method #2--divide by ten, again, and again
This is what we do:
* Create a new variable `currentTally`
* Divide the integer value by 10, rounding down
* Check if the value is 0. 
* If it is NOT, add 1 to the currentTally and keep repeating this process until it is 0.
* If it IS 0, check to see if your current tally is even.
* If so, add 1 ot the tally of even numbers
* Return the tally

```
const findNumbers = function(arr) {

    let evenCount = 0;
		
    for(let i = 0; i<arr.length; i++) {
        let currentCount = 0;
        let digit = arr[i];
        while(digit != 0){
            digit = Math.floor(digit/10);
            currentCount += 1;
        } 
        
       if(currentCount % 2 === 0) {
           evenCount += 1;
       }
    }
  return evenCount;
}
```

I have to admit, the second way didn't occur to me at first, but it's a great way to do it, especially if you are coding in language like Java where integer division is already rounded to an integer, unlike JavaScript, which is why we have to use `Math.floor()` in the example above.

