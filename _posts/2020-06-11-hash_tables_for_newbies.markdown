---
layout: post
title:      "Hash Tables For Newbies"
date:       2020-06-11 19:05:31 -0400
permalink:  hash_tables_for_newbies
---


When I'm trying to learn a new concept, I'm often frustrated by explanations of the subject written by those who understand it because they often write to an audience of those who already understand it. 

One of those subjects is hash tables.

According to [wikipedia](https://en.wikipedia.org/wiki/Hash_table), Hash tables (or "hash maps") are data structures "...that implement an associative array abstract data type, a structure that can map keys to values."

That's exactly what I would have said!

Yeah, no. 

Let's break down what a hash table is, how it uses a "hash function", and what hash tables are generally good for. I'll do this with a simple, real-world hypothetical that (hopefully) gives an idea of why we have hash tables and how they compare to other data structures, as well as the most common problem with hash tables--collisions.

## Scenario - Daycare and Kids' Shoes
Let's say you work at a daycare center, and when the kids arrive they always take off their shoes and put them into a cubby. Normally you have 10 kids and 10 cubbies, so each kid has a cubby. It's your job to go get those shoes when each kid leaves. How might you manage it?  

## Method #1 -- Nametags
Into each cubby you place a little piece of paper for identification--a nametag. When it's time to go, the kid tells you their name and you have to go to the cubbies and get the matching shoes.  But if you don't remember where any particular kid's shoes are, how do you find them?  

The method seems straightforward; take the kid's name and start at the first cubby and check the nametag. If it's what you are looking for, yay! If not, move on to the next. Then the next, and so on until you found the matching name. In a worse case scenario, if the kid you are looking for is last, then you would check all 10 cubbies before returning the shoes. You would first search, then retrieve. Not very efficient, right?

## Method #2 -- Claim Ticket
You number the cubbies, 1-10. Just like a coat check, when each kid drops off their shoes, they get a claim ticket with a number on it corresponding to the numbered cubby. Then, when a kid hands in the claim ticket with #4 on it, you could go straight to the #4 cubby and retrieve those shoes. There is no searching. All scenarios have you looking at only one cubby. That's much faster! 

But this would never work. Kids are going to lose their claim tickets. And every day there would be a new round of handing out the tickets and expecting the kids to hold on to them all day. And what if one day there is a visitor, and we have 11 kids? Even if the cubbies are big enough to fit two pairs of shoes, there is no new ticket to hand out. What can we do??? 

## Method #3 -- Hash Table
What if we could take a kids name, run it through some process that converted it to a number, and then that number is the number on the cubby? Or better yet, what if you could make each name correspond to one of the numbers, 1-10?

That's exactly what a **hash function** does! It takes a value and turns it into a number, and the number tells you exactly where to go to find what you are looking for. 

For example, let's count the number of letters in a name and make that the cubby number. So you give me "Malcom", that's cubby 6. You tell me "Mavis" and I tell you cubby 5. "Danielle" is cubby 8. 

Now, a kid doesn't have to hand you a number for a claim ticket, the kid just says their name and you can take it from there. A small run through the name-to-number hash function and you have the number of the cubby! Brilliant! 

## Collisions! 
Ah, but wait. I'm sure you've seen the problem. This cool hash function of mine that turns names into numbers turns a bunch of different names into the same number? "Saba", "Eric", "Nico", and "Rosa" all end up in cubby 4. 

This means our hash functions results in a large number of **collisions**. Every hash function has the possibility of collisions, but they can range to very likely (like my example) to very unlikely (like hashes used in cryptography).

## A Better Hash Function
I could give each letter in the alphabet a value, 1-26, corresponding to its place among the 26 letters. Then, when a kid gives me their name, it's much more likely that the cubby is unique to that kid. Now, "Saba" is equal to 19+1+2+1= 23, and "Eric" is equal to 5+18+9+3=35. Better! 

But there are still problems. We may have to have a few hundred cubbies ready to account for long names with lots of Zs and Ys. Plus, what if we had names with letters, ("Reza"=18+5+26+1=50) and another name with different but equal values ("Oto"=20+15+15=50)? And of course if two names had the same letters we would have a collision.

## Hash Function Problem Solved! 
There are hash functions that work tremendously well and luckily we don't have to make our own. So let's say we used one and now each kid's name results in a unique number. We could use the number of cubbies (10) and take the [modulo operator](https://en.wikipedia.org/wiki/Modulo_operation) (the remainder after dividing by 10) and that would give us one of the cubbies 1-10 (a zero remainder would be number 10). 

## Still one problem, though
We still may have collisions! Even though we get a unique number, we don't want to have some huge number of cubbies so when we do the modulo operation we could end up with more than one pair of shoes in a cubby.

In that case we'd need to further separate that particular cubby into smaller cubbies, for example if cubby 7 had two pairs of shoes we may designate a cubby 7a and a cubby 7b. Then, we would still go to the correct cubby rather quickly, but we'd still have a tiny bit of searching left to do. 

## How This Example Relates
A hash table is a data structure that maps *keys* (the number we get from the kid's name) and *values* (the shoes) and places those values into *buckets* (the cubbies). These can be called dictionaries, hashes, and/or associative arrays, among other terms. A hash table uses a *hash function* (the function including the modulo operation) to generate the keys, which allows for very fast retrieval. Using Big O notation, a hash table both gets and sets with a Big O value of O(1), whereas our nametag example gets with a Big O value of O(n). 

## Conclusion
Hopefully this gives you a solid fundemental understanding of what a hash table is and how hash functions are used. For next steps, I'd recommend [this post](https://www.mattzeunert.com/2017/02/01/implementing-a-hash-table-in-javascript.html) by Matt Zeunert and [this one](https://logicmason.com/2013/how-to-implement-a-hash-table/) by Mark Wilbur.
