---
layout: post
title:      "String literal and the unescaped line break"
date:       2018-05-27 03:36:20 -0400
permalink:  string_literal_and_the_unescaped_line_break
---


I recently spent about 90 minutes chasing a JS error, 'SyntaxError string literal contains an unescaped line break'. <br>
<br>
Line 2 was throwing the error. Always a lot of problems with line 2...
<br>
<br>
`<lang="en" />`
<br>
*Well, that clears it up.*
<br><br>

The escape character was the issue because *somewhere* else I was rendering something incorrectly on the page, but there was nothing obvious.<br><br>

**A Little Background on the app 'Payam'** - *[github link](https://github.com/lawrend/payam-with-js)*<br>
<br>
A *Payam* is a kind of poem, usually made by eight players. I added a feature to my app to automatically generate a new payam, making the lines of the poem from random words taken from a word list.<br>
<br>
The list I had on hand was for making random pass phrases using 6-sided die:<br>
<br>
1. Roll five times;
2. Generate a five-digit number (each integer between 1-6);
3. Look up the corresponding word on the list.
 <br><br>

| ![random word list](https://i.imgur.com/BvBUzuB.png)  |<br>
*The List*
<br>
<br>
1.  I replicated five rolls to generate the number,
2.  searched the file for it,
3.  and returned the line, index of [6..-1] *(starting at the 7th character to just return the word and not the number)*
<br>
<br>
```
def random_word
        line_number = ""
        5.times do
            line_number << Random.new.rand(1..6).to_s
        end
        File.open("/Users/douglaslawrence/Development/code/rails-and-js/project-mode/payam-with-js/app/assets/other/eff_large_wordlist.txt") do |file|
            the_word = file.find {|line| line =~ /#{line_number}/ }
            return the_word[6..-1]
        end
    end
```
<br>	
I used this to generate a random *Payam*:<br>
* ten `random_word` formed each of eight lines
* one `random_word` was the title<br>
<br>
```  
def random
        @payam = Payam.new
        @payam.title = "Random-" + random_word
        @payam.decomp = false
        @payam.current_scribe = nil
        @payam.counter = 8
        @payam.style_id = 1
        @payam.save
        for i in 0..7
            line_text = []
            10.times do
                line_text << random_word
            end
            Line.create(:text => line_text.join(" "), :auth_id => current_user.id, :count => i+1, :payam_id => @payam.id)
        end
        flash[:notice] = "Random Payam Made!"
        redirect_to root_path
    end
```
<br>
<br>
This all worked *except* for one situation where I was dynamically adding the lines of the payam to the DOM. It worked for 'normal' payams, but not for the random. What gives?<br>
<br>
It took me a long time to hunt down this error and it came down to changing one number:<br>
<br>
*change*<br>
`return the_word[6..-1]`<br>
*to*<br>
`return the_word[6..-2]`<br>
<br>
`random_word` was returning the `/n` newline character that was at the end of every line of the word list. This newline was the unescaped line break showing up in my string literal. <br>
<br>
Figuring this out was about 10% testing things, 5% writing code, and 85% pushing boulders with my thoughts.<br>
<br>
![yoda moving rocks with his mind](https://i.imgur.com/r7ERB7y.jpg)<br>
*This is accurate*<br>
<br>
Don't judge coding work by how much typing you hear, unless you give equal weight to how much staring out the window you see. 

