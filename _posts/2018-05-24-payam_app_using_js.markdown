---
layout: post
title:      "Payam app using JS"
date:       2018-05-24 22:19:10 +0000
permalink:  payam_app_using_js
---

*Github: https://github.com/lawrend/payam-with-js* 

I made a Rails app (which you can read about [here](http://printsamongleaves.com/2017/07/08/rails_portfolio_project_-_payam/)), and this project was supposed to be a quick addition of some JS and AJAX calls. 

Well, the previous app had some issues with overgrown complexity so I stripped this thing down and rebuilt a lot from scratch.

My goal with this project was just to learn as much as I could, but that meant I ended up spending time on things that weren't important to overall functionality, but I just wanted to see if I could make it work. And that got me learning things I didn't intend to, and taking a lot of time I didn't need to. 

## Technical Notes
### Payam - What is It? 

* #### Players and Gameplay

Players sign up with an email, username, and password. Alternatively, they can use Github. 

A Player can make a new Payam, pick its title and style, and write the first line of 10-20 words. The Payam is then sent to a random Player to write the second Line, only able to read the last 5 words of the previous Line. 

After eight Players write Lines the Payam is completed and all Lines can be read. The Payam is listed with all other completed Payams of its Style, and anyone can read them.

* #### Associations

A Payam does not belong to a Player directly (there is not a player_id indexed in the Payams table). Instead, a Line belongs to both a Player (as the author) and a Payam. A Player has many Lines and has many Payams through those Lines. 

Each Payam belongs to a Style and has many Lines, and has many Players through those Lines.

* #### Decompositions

Once a Payam is completed and can be displayed it can be decomposed--one random word is removed from each line. Another press of the button and another word from each line is gone, and a Player can press until only one word on each Line remains. If at any time the Player can save the current decomposed Payam as a special kind of Payam--a "Decomposition". A decomposition's Lines are all by the same Player, and it is associated with the original Payam through an integer "orig" attribute. 

* #### AJAX calls and JS

A Player's homepage shows a Player's completed Params one at a time, adding them to the DOM with JS and an AJAX call. 

When a list of Payams are displayed, only their titles are listed. A button calls a JS function and dynamically adds the Lines to the DOM to display them. Another push of the button hides them again.

When a Player makes a decomposition and presses the "Save" button, a new Payam is instantiated and saved and 8 new Lines are instantiated and saved. The Payam is dynamically added to the page immediately and can be previewed like all other decompositions associated with that Payam.

## Takeaway: One Small Bit of Laziness Now Can Result In Large Bits of Headache in a Later State

When I first started coding, nearly every attempt at functionality had some error to break it. It was slow going, but by the time I was ready to move from one line to the next, everything had been thouroughly thought through. 

I've gotten marginally more experienced with coding so that occasionally I throw a few lines together and it works on the first go. Hey, I'm as surprised about it as you are. But that means my idiotic variable names that sounded funny to me at the time are still in there. As are let-me-see-how-this-looks CSS class names and random line height adjustments because I bit some html and css from a site I liked the look of. 

As soon as I'd noticed evidence of my carelessness and that there was an annoying and/or embarrassing lack of consistency in naming conventions, I had to go back and change it.

Time and errors and complexity, oh my. 

It was the butterfly effect on display, as a small bit of carelessness increased in hassle in unforeseeable ways.

Among the things I went back to change in my code: 

* CSS Naming Conventions
* CSS Syntax and Spacing
* JS Naming Conventions
* Inconsistent Styling
* Class and Function Names

All in the name of learning, I suppose. 

