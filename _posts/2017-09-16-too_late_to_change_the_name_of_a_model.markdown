---
layout: post
title:  "Too Late to Change the Name of a Model?"
date:   2017-09-16 20:36:03 +0000
---


Sometimes you give a Model a name and realize much too late that it is NOT the name you want a user to see.

Hypothetically, let's imagine you made an app where several users were going to collectively create something like a multi-line piece of poetry, and let's say that because it was based on a Surrealist game called *Exquisite Corpse* and that the body of text is a *Corpus* that you should give the Model the name of ***Corpse***.

Hypothetically. Because you would never do that. 

But of course I would. And after writing a decent amount of code and generating controllers and nailing down associations and building some views I realized that users may not really feel all that great about clicking on the "Make a new Corpse!" link. 

Luckily, with Rails things are pretty easy. 
```
Rails.application.routes.draw do
  resources :styles do
	  resources :corpses, :path => "payams"
  end
  resources :lines, :except => [:update, :edit]
  resources :corpses, :path => "payams" do
    member do
      post 'decompose'
    end
  end
  devise_for :users, controllers: { omniauth_callbacks: 'users/omniauth_callbacks' }
  resources :players do
    resources :corpses, :path => "payams"
    resources :lines
  end
end
	```
	
Everywhere I had used `:corpses` I added a `:path => "payams"` and the urls were SFW again.

