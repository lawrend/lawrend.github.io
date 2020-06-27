---
layout: post
title:      "What's the Difference? "
date:       2020-06-25 20:17:57 -0400
permalink:  whats_the_difference
---

## Minimum absolute difference between two elements in an array 

## Greedy Algorithms 

Some alogrithm challenges involved something called [*greedy algorithms*](https://en.wikipedia.org/wiki/Greedy_algorithm).  These are algorithms that may not solve for the *globally* optimal solution, but at each stage makes a *locally* optimal solution. 

Here is one problem that can be solved using a greedy algorithm: 
Given an array, find the minimum absolute difference between any two values in the array. 
A *absolute difference* here is just the difference between two values expressed as a positive integer. So for example, the absolute difference between -4 and -8 is 4, as is the absolute difference between 8 and 4. However, the absolute difference between 4 and -4 is 8. 
```
[-4, -8] = 4
[4, 8] = 4
[-4, 4] = 8
```

It would be very easy if we had only two values. However, we will have many values in the array, and with each new value added we add complexity. 

The first thing to do is to sort the array. This allows us to compare values that are close to one another. If we find a consecutive pair, then we know we have our smallest possible difference. 

In JavaScript we have to add our own callback function to the `sort` method:

```
function minimumAbsoluteDifference(arr) {
    arr = arr.sort((a, b) => a - b);
}
```

We need an initial minimum difference so that we have something to which we can compare our calculated values. So, let's use the difference between the first and second values:

```
function minimumAbsoluteDifference(arr) {
    arr = arr.sort((a, b) => a - b);
		let minDiff = arr[1] - arr[0];
}
```

I'm subracting the larger value from the smaller, which will always give us our absolute difference. 

Then I just iterate through the values, comparing the adjacent values to one another, then comparing that value to the minimum value. 

```
function minimumAbsoluteDifference(arr) {
    arr = arr.sort((a, b) => a-b) 
    let minDiff = arr[1] - arr[0]
   for(let a =0; a < arr.length; a++) {
       let diff = arr[(a+1)] - arr[a];
       if(diff < minDiff) {
           minDiff = diff;
       }
   }
   return minDiff;
}
```

This isn't the most efficient solution, but it works. I'm still slowly progressing through my understanding of algorithms, so perhaps I'll update this with a more efficient solution in the future. 

One quick fix would be to check if minDiff is 1 at each iteration. If so, the difference can get no smaller (there are no identical values--if there were the value we'd check is 0). And if that's the case, return `minDiff` and exit the program! 

