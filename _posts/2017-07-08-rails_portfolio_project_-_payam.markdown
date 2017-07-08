---
layout: post
title:  "Rails Portfolio Project - Payam"
date:   2017-07-08 03:24:15 -0400
---

*Note: the four model classes--User, Corpse(aka Payam), Style, and Line--are capitalized throughout this post.*

### <a href="#overview">I. Overview</a>
### <a href="#models">II. Models and Associations</a>
### <a href="#methods">III. Validations and Methods</a>
### <a href="#walkthrough">IV. Walkthrough</a> 

<a id="overview"> **I. OVERVIEW** </a>

The code for this app can be found on my github [here.](https://github.com/lawrend/payam)

The app is based on the game ["Exquisite Corpse"](https://en.wikipedia.org/wiki/Exquisite_corpse). Eight Users each add one Line to a kind of collaborative poem called a "Payam". The first User creates a Payam but the other seven Users are chosen at random. Currently, a User can randomly be chosen to add a Line to a particular Payam more than once, although never back-to-back. 

When each User writes their Line, they can only see a small part of the whole--the title, style, and last five words of the previous User's Line.

The whole Payam can be read only after all eight Users have added their Lines. When it is viewed, the authorship of each Line is hidden from the viewer, although each Line is rendered in a different color. 

The final odd twist to the game is that once the Payam is public, *anyone* can press a Payam's "Decompose" button, and one word will be permanently deleted from each Line. The button can be pushed until only one word per Line remains.

### *'Payam' to the User but 'Corpse' to the Coder - Why?*

Yeah, so about that name thing...I was originally going to call this app *Exquisite Corpse*  and a finished round of the game would be a *Corpse*, but that sounded way too creepy.

I decided to call each round a *Payam* but I'd already written a lot of the code, controllers, views, migrations, etc. Rather than go back and change all the code and generate new controllers, I made all routes read as "payams" when it would otherwise have read "corpses", and could then refer to payams for the User and Corpses for me.

In a real-world situation I would have gone in and re-written the code, especially if this was going to be maintained by future developers.  However, I used it as an opportunity to learn a bit more about routing. Read about it [here](#).

<h2 id="models"> **II. MODELS AND ASSOCIATIONS**</h2>
 
1. **User** - has many Lines they have written, and has many Corpses through those Lines
2. **Corpse** - has many (8) Lines, has many (8) Users through those Lines, and belongs to a Style
3. **Line** - belongs to a Corpse and belongs to the User who wrote the Line
4. **Style** - has many Corpses

### 1. User

#### *Attributes*

A User has fields for:
* **username**
* **email**
* **password**

#### *Associations*

* Each ***User has many Lines***. The `:auth_id` of a Line is the User's id so the foreign key is `:auth_id`.

* Each ***User has many Corpses through Lines***.

#### *Model*

```
class User < ApplicationRecord
  has_many :lines, :foreign_key => "auth_id"
  has_many :corpses, through: :lines
end
```

### 2. Corpse

#### *Attributes*

A Corpse has fields for:
* **title**
* **style_id**
* **current_scribe** (the current User)
* **counter** (the line number of the Line that the current_scribe creates)

#### *Associations*

* Each ***Corpse belongs to a Style***.

* Each ***Corpse has many Lines***. 

* Each ***Corpse has many Users through Lines***. The author of each Line is a User so the source is `:auth`.

#### *Scopes*

When eight Users have written Lines, the `:current_scribe` is set to `nil`. The scope `:completed` is a Corpse with a `current_scribe` set to `nil`.

#### *Model*

```
class Corpse < ApplicationRecord
  has_many :lines, dependent: :destroy
  belongs_to :style
  has_many :users, through: :lines, source: :auth
	accepts_nested_attributes_for :lines
  accepts_nested_attributes_for :style, reject_if: proc { |attributes| attributes['name'].blank? }
	scope :completed, -> { where(:current_scribe => nil) }
end
```

### 3. Line

#### *Attributes*

A Line has fiedls for:
* **text**
* **count** (the line number)
* **author id**
* **corpse id**

#### *Associations*

* Each ***Line belongs to an "auth"***. The author ("auth") of a Line is a User so the `:class_name` is set to `User`. 

* Each ***Line belongs to a Corpse***. Because both a new Corpse and a new Line are instantiated when the form is submitted, this association has an option of `optional: true` to allow a Line to be created simultaneously with the Corpse.  

#### *Model*

```
class Line < ApplicationRecord
  belongs_to :auth, :class_name => "User"
  belongs_to :corpse, optional: true
end
```

### 4. Style

#### *Attributes*

A Style has a field for: 
* **name**

#### *Associations*

* Each ***Style has many Corpses***.

#### *Model*

```
class Style < ApplicationRecord
  has_many :corpses
end
```

<h2 id="methods"> **III. VALIDATIONS AND METHODS** </h2>

### 1. User

#### *Validations*
I let Devise handles most of the validations but add a uniqueness validation to `:username`.

#### *Class Methods - Oauth*
* **self.find_for_oauth
* self.new_for_session
* self.dummy_email**

#### *Instance Methods*
* **waiting:** Displays all Corpses awaiting the current User's addition of a Line.

```
class User < ApplicationRecord

  validates_uniqueness_of :username
	
	def waiting
    Corpse.where(:current_scribe => self.id)
  end
	
end
```

### 2. Corpse

#### *Validations*

A Corpse is just a collections of associations with a unique, one-word title. The uniqueness validation is an obvious addition but I also add a character maximum to avoid buffer overflow.

For the one-word title validation I created a `TitleValidator` class and put it in the `app/models/concerns` folder. It wasn't necessary but I want to use at least one concern and it is at least plausible that another classs might have a `:title` field with a one-word restriction.

#### *Methods*

* **previous_five:** Displays the last five words of the last Line added to the Corpse.
* **send_to_next:** I tried to DRY up my controller but this is a weak attempt. It just bumps up the counter of the Corpse by one and saves the Corpse. The current_scribe is picked at random using logic in the controller so when this method is run the Corpse is saved with a new current_scribe and counter and is accessible to the newly-selected current_scribe.
* **style=(name):** A Corpse's Style is selected from either a drop-down menu of existing Styles or a new User-generated Style, so this custom setter is used for the nested form on the new Corpse view.

```
class Corpse < ApplicationRecord
  validates :title, presence: true, length: {maximum: 40}
  validates_with TitleValidator
	
  def previous_five
    newln = Line.where(:corpse_id => self.id, :count => self.counter-1).first
    lstln = newln.text
    llstln = lstln.split
    llstln[-5..-1].join(" ")
  end

  def send_to_next
    self.counter += 1
    self.save
  end

  private

  def style=(name)
    self.style = Style.find_or_create_by(name)
  end
	
end
```

```
class TitleValidator < ActiveModel::Validator

  def validate(record)
    if record.title.split.size > 1
      record.errors.add(:title, "Your title is too long. Remember--One Word.")
    end
  end
		
end
```

### 3. Line

#### *Validations*
Validates the presence of `:text` and sets the character maximum to 200 to prevent buffer overflow.

I use a custom validation to validate a word count of between 10-20 words for a Line. The word count validation is not run after a Corpse is completed to allow for a post-completion method (`lose_word`, discussed below). A completed Line has a `:current_scribe` set to `nil`, but because a new Line is formed contemporaneously with a new Corpse, using just `current_scribe == nil` as an indicator of a completed Line would keep the validation from running for the first Line. So, the validation does not run when both the Line is associated with a Corpse and `current_scribe == nil`. 

#### *Methods*
* **word_count:** Splits the text into an array and adds an error if the word count is either less than 10 or greater than 20.
* **lose_word:** The button on the show view of a completed Corpse, when pressed, permanently deletes a word from each Line of the Corpse, so long as the Line has at least one word. Because the word_count custom validation is thrown when a Line's word count drops below 10, it does not run on completed Corpse's so that the lose_word method can run properly.

```
class Line < ApplicationRecord
  validates :text, presence: true, length: {maximum: 200}
  validate :word_count, unless: Proc.new {|a| a.corpse != nil && a.corpse.current_scribe.nil? }
	
	def lose_word
    if self.text.split.length > 1
      prev_line = self.text.split
      prev_line.delete_at(rand(prev_line.length))
      self.text = prev_line.join(" ")
      self.save
    end
  end

  private
  def word_count
    @count = text.scan(/[[:alpha:]]+/).count
    if @count < 10
      errors.add(:text, "That's #{10 - @count} too few words for the line.")
    elsif @count > 20
      errors.add(:text, "That's #{@count - 20} too many words for the line.")
    end
  end

end
```
### 4. Style

#### *Validations*
The uniqueness of the name is validated before the first save of a Style that is not yet in the database.

#### *Methods*
* **newest_style?:** Checks to see if a Style has yet to be saved to the database.

```
class Style < ApplicationRecord
  before_save validates :name, uniqueness: {:case_sensitive => false}, if:  :newest_style?

  def newest_style?
    self.id.nil?
  end

end
```

<h2 id="walkthrough"> **IV. WALKTHROUGH** </h2>

### **Part 1: New User**

A User can sign-up/sign-in using Devise or Github (thanks to Omniauth). A User enters an **email** and **password**, with standard validations provided by Devise.

A User also adds a **username** with a `:uniqueness` validation. Adding a username to Devise was tricky, and I wrote up a small blog post about it [here](#).

When a User signs in they can see:

* Any Corpse they have worked on that can now be viewed; 
* Any Corpse for which they are the next author and thus need to provide a Line for; and
* A link to create a new Corpse. 

Also, the header for a signed-in User has links to view all viewable Corpses.
![](http://i.imgur.com/EfArogrl.png)
### ** Part 2: Creating a New Corpse** 

First, yes this sounds like a horror movie. But moving on...

![](http://i.imgur.com/LV6cfVbl.png)

The creator of a new Corpse provides:

* a one-word title;
* a one-word Style (either from a list of pre-existing or by creating a new one); and
* a first Line of 10-20 words. 

The form has nested attributes for both a Style and a Line.

When the new Corpse form is submitted:

* a new Corpse is instantiated;
* a new Line is instantiated; 
* a new Style is instantiated (if the User creates a new one); and
* a new User is randomly selected to write the second Line.

### ** Part 3: Adding a Line to a Corpse**

When subsequent Users write Lines for the Corpse, They can only view the Corpse's `:title` and Style and can only see the last five words of the previous author's Line. 

![](http://i.imgur.com/bUpXXxql.png)

### ** Part 4: Viewing a Corpse **

Each line is rendered in a different color, but the authorship remains anonymous. 

![](http://i.imgur.com/yuMklDZl.png)

### ** Part 5: Decomposing a Corpse ** 

There is a "Decompose" button on the page that anyone can press. It randomly permanently removes one word from each line of the Corpse. The button can be pressed until there is only one word left in each line of the Corpse. I wanted everything to be temporary and unstable. I think the surrealists would have appreciated it.

In this example, the button has been pressed 7 times.

![](http://i.imgur.com/5KV5m7gl.png)





