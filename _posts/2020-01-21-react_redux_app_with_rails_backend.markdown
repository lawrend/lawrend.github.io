---
layout: post
title:      "React/Redux App with Rails Backend"
date:       2020-01-21 07:59:05 +0000
permalink:  react_redux_app_with_rails_backend
---

* Using APIs from Enigma (a Government records resource), Google Maps, and Wikipedia, and deploying the App to Heroku

* Rails will handle all API calls to Google, Enigma, and Wikipedia. But React will call the Rails API for its data needs.
* *[this project on github](https://github.com/lawrend/fluffy-memory)*


![](https://i.imgur.com/TBcpoRUl.png)
## This App - What is It?
##### Endangered - An Interactive Map of Endangered Species in the U.S.
![](https://i.imgur.com/Md3wm5Hl.jpg)


I pull a list of endangered and threatened species in the United States from [Enigma] which is not only a list of all endangered and threatened species, but also the name of every wildlife sanctuary and the state in which it's located.  

Next, I use Google's [Maps API] to do a few things:
* Add latitude and longitude coordinates to the Enigma records
* Generate customized maps that display markers marking the locations of the protected area
* Markers, when clicked, will display all of the species threatened and/or endangered at that location

Then I use [Wikipedia's API] to pull an image and a description of the species to display. 

*Using both Enigma and Google Maps requires signing up for an account and using an assigned key. It's all free, but it adds friction to the process. I created a .env file and stored the keys in there. If you do the same, **be sure to add .env to your .gitignore file** so it won't end up publically available on github.*

Before you jump in make sure you have the latest Ruby, Rails, and Bundler installed. Bundler just updated to [Bundler 2](https://bundler.io/blog/2019/01/03/announcing-bundler-2.html). Make sure you are current with a quick:

```
$ gem update --system
$ gem install bundler
```

All Set? Okay. Here's what we're going to do...

## <a href="#rails-new">I. Setup - New Rails Project</a>
## <a href="#react-new">II. Setup - New React Project</a>
## <a href="#rails-header">III. Rails Back-End</a>
#### <a href="#models">A. Models</a>
#### <a href="#controllers">B. Controllers</a>
#### <a href="#faraday">C. Using Faraday for API Calls</a>
#### <a href="#data-manipulation">D. Manipulating Returned Data</a>
#### <a href="#rails-routes">E. Routes</a>
## <a href="#react-header">IV. React Front-End</a>
#### <a href="#react-dependencies">A. Add Dependencies: Semantic, Axios, google-maps-api</a>
#### <a href="#containers-components">B. Containers and Components</a>
#### <a href="#redux">C. Adding Redux - Application State</a>
#### <a href="#react-router">D. React Router</a>
## <a href="#heroku">V. Deploying to Heroku</a>

## <a id="rails-new">I. Setup - New Rails Project</a>
 
Rails 5 lets us set up a new Rails project for use just as an api. We are going to tweak things a little bit more by using postgresql as our database as well as making some room for use of rspec instead of the Rails default TestUnit. 

So, in our terminal we type:

```
rails new my-project --api --database=postgresql -T --no-rdoc --no-ri
```

 Now switch into the project directory with `cd my-project`. Get [postgresql](https://www.postgresql.org/download/) up and running on your computer and make sure the Rails server has no issues. 

We can open it up on port 3001 with:

 ```
 rails s -p 3001
 ```
 
*If you get the `NoDatabaseError` it just means rails hasn't yet built the database. Try running `rails db:create` and then try running the server again.*

## <a id="react-new">II. Setup - New React Project With create-react-app</a>
Now we are going to make a new React project **inside the Rails project**!!!

![](https://i.imgur.com/2S8CksK.png)
 
We will call our React app 'client' and it will exist as a subdirectory of the Rails project. So, make sure you are in the parent directory of the Rails project when you proceed. 

It's pretty easy to use `create-react-app`. **A global install of create-react-app is now *not* the way to go.** You can find a more detailed how-to [here](https://create-react-app.dev/docs/getting-started/), but if you haven't previously installed create-react-app globally then the simple instructions are to run the following in your terminal:

 ```
 npx create-react-app client
 cd client
 yarn start
 ```
 
 Now if you open [https://localhost:3000](https://localhost:3000) you will see
 the React logo and you'll be looking at a working React app!
 
 Also, right now both servers are running--Rails on port 3001 and React on port 3000

##### Let React Talk to Rails
We have to set up the React project to reach out to the rails backend when it makes api calls. 

Open up the `package.json` file located in the `/client` folder and add:

```
"proxy": "http://localhost:3001",
```

*If you want to see what your `package.json` folder will look like with this and all other dependencies, see <a href="#package-json-file">this</a>

##### One Command to Start Them All - Procfile & Heroku
Now the React app will use port 3001 for all api calls, but getting everything up and running is going to be a pain if we have to start everything seperately. We can take care of this! Here is how you fire everything up using one command:
###### Procfile
First, `touch Procfile.dev` in the root folder of the Rails project. Then, inside of it add this:

```
web: PORT=3000 yarn --cwd client start
api:  PORT=3001 bundle exec rails s
```

###### Heroku 
We are going to use the `heroku` CLI to manage the procfile. If you use `homebrew` then that's easiest. `brew install heroku` and you are set. 
Once you have the `heroku` CLI installed, test it out by running:

```
heroku local -f Procfile.dev
```

That gets it up and running with one command, but that command is a drag to type everytime.  Instead, `touch ./lib/tasks/start.rake` and open it up. Inside you will need to put the following: 

```
namespace :start do
  task :development do
	   exec 'heroku local -f Procfile.dev'
  end
end

desc 'Start development server'
task :start => 'start:development'
```

Now, when you are in the root directory of the Rails project you can simply run `rake start` and your app will be up and running. 

You can take it from here and build your own app! However, I'm going to walk through what I did when I used the APIs from Google Maps, Wikipedia, and Enigma along with how I folded in Redux, `semantic-ui-react`, `google-maps-react` and so much more...

## <a id="rails-header"></a>III. Rails Back-End
#### <a id="models"></a>A. Models

The relational data structure is pretty straight forward:

* `Species` has a `has_many` relationship with `Location`, and vice versa 
* `SpeciesLocation` has a `belongs_to` relationship with a `Species` and a `Location` 
* Each `Location` has a `belongs_to` relationship with a `State`
* Each `State` has a `has_many` relationship with `Location` 
* `State` also has a `has_many` relationship with `SpeciesLocation` and a `has_many` relationship with `Species`

So, I will have models for:

* `Species`
* `Location`
* `SpeciesLocation`
* `State`

I can generate these using scaffold in Rails.

* `rails g scaffold Species name:string location:string status:string desc:string imgsrc:string`
* `rails g scaffold Location (name etc...)`
* `rails g scaffold State (name etc...).`

SpeciesLocation is a special case since it's just for using with our `has_many :through` relationship. We can strip it down even further and use:

* `rails g model SpeciesLocation species:references location:references`

#### <a id="controllers"></a>B. Controllers
Add some these files to `app/controllers/concerns` - `enigma.rb` and `wikipedia.rb`.
* Controllers
 * Locations Controller
 * Species Controller
 * States Controller
* Helpers/Concerns
* Class Methods

#### <a id="faraday"></a>C. Faraday
Faraday makes GET and POST calls to an api a lot easier to handle than without. But first add this to the Gemfile:

```
gem 'faraday'
gem 'faraday_middleware'
```

#### <a id="data-manipulation"></a>D. Manipulating Returned Data
After using Faraday to get the data from Enigma/Google Maps/Wikipedia, I manipulate the data before sending it back to the Front-End. This includes:

* parsing names of places/species
* sorting data by value
* adding 2-letter state abbreviations
* Parsing names in particular ways for use with Google Maps

#### <a id="rails-routes"></a>E. Routes
Every time I use `axios` in React to make an api call I specify the url. This url matches a declared route in my Rails app which then sends it to the appropriate controller and function within that controller. Here's my `config/routes.rb` file:

![](https://i.imgur.com/QQlpSd7l.png)
## <a id="react-header"></a>IV. React Front-End
[React](https://reactjs.org) is a JavaScript front-end library that lets you easily make single-page applications with an eye towards modularity and reuse of code.

#### <a id="react-dependencies"></a>A. Add Dependencies: Semantic UI, Axios, google-maps-api, logger, thunk, redux-devtools-extension

I'm adding `react` and it's many many dependencies using `yarn`. 

[Redux](https://redux.js.org) pairs with `react` to manage state in a centralized location.  I'm adding `redux` using `yarn` as well. I'm also adding `react-redux` which let's the two play nicely together.

I'm assuming you have some familiarity with the concepts and have a basic understanding of `react` and `redux`. If not, check out the getting started guides for each.

All of these can be added from the root of the Rails project using `yarn --cwd client add name`. Beyond adding `react` and `redux` I'm adding: 

* `axios` for api calls
* `semantic-ui-react` and `semantic-ui-css` for looks
* `google-maps-react` for help dealing with the Google Maps API
* `logger` for a verbose console, letting me know what's being called and served
* `thunk` to let us use middleware
* `redux-devtools-extension` to let us use redux devtools
* `react-router-dom` for handling url routes and keeping everything on a single page

After all of that my `package.json` file looks like this: 

<a id="package-json-file"></a>
![](https://i.imgur.com/wzVF5lSl.png)

#### <a id="containers-components"></a>Containers and Components
In my project I will want a folder structure that helps me keep track of all the kinds of components I'll be dealing with. I'm using a definition of "if it has a state, it's a container". So, I made these folders to keep track of containers vs components, the store (a redux-related folder), css, and resources (.jpegs and the like):

```
src/components/
src/containers/
src/css/
src/store/
src/resources/
```

Also, I will use a few Google fonts so add a link to that library. Open `client/public/index.html` and add `<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">` between the `<head></head>` tags.

##### React Components
When a user first hits the homepage, I want to query the Enigma database and build it. So, I want the user to have something to look at but also keep the user from using anything that's not ready yet.
So, I use `componentDidMount` to dispatch a call to the store to get all the data from Enigma, parse it, create records in the database, and serve up some of the data back to the app, which it will use to set the application state. 

#### <a id="redux"></a> C. Adding Redux - Application State
*It is entirely possible for me to wire this app up without Redux, but there is a not negligible reason for using it and it's good to get the practice. Yee.*

[Redux] lets multiple components access the same state. It simplifies things tremendously if you have a large application with many, many components that need to display and/or modify state. With Redux, state isn't kept within the component so it's not actually modified by the component (i.e. we rarely if ever call `setState` within a component). 

But if we don't call `setState`, how do we set the state?!?!

##### The Store

Redux maintains a "Store" which contains the application-level state. We `dispatch` `actions` which calls a `reducer` function which returns a fresh copy of the state with any changes the action caused. If we want to display the state within our component, we subscribe to the store's state. This keeps our component current, as any update to that state will update the component. 

Sound a little complicated? It is at first. But once it clicks it's pretty clear. Here's how it's done:

1. State - `mapStateToProps`: defining a `const` called `mapStateToProps` then using `connect` to connect it to the store lets us pick and choose whichever pieces of the state we need and have them available as named `props`. 
2. Actions - `mapDispatchToProps`: defining a `const` called `mapDisptatchToProps` then using `connect` to connect it to the store lets us pick and choose the actions we need to call within that component. The actions are available as `props`, just as if they'd been passed down from a parent component. 

#### <a id="react-router"></a>D. React Router

By using `react-router` and `react-router-dom`, we can easily define multiple url routes that render different components rather than render different pages. We drop a `<Routes />` component within the `<App />` component, so that navigation to the url will instead render the components defined in my `client/src/routes.js` file Here is my routes file:
![](https://i.imgur.com/tNxNMry.png)

## <a id="heroku"></a>V. Deploying to Heroku - uh, not yet
I will come back and update this with more info once i get the bugs worked out - it seems my multiple API calls and API keys are more than i bargained for...

1. add package.json to root with `yarn init`
2. add `Procfile` to root

Add to package.json:

```
{
"name": "endangered",
"license": "MIT",
  "engines": {
    "node": "13.6.0",
    "yarn": "1.21.1"
  },
  "scripts": {
    "build": "yarn --cwd client install && yarn --cwd client build",
    "deploy": "cp -a client/build/. public/",
    "heroku-postbuild": "yarn build && yarn deploy"
  }
}
```

Add to `Procfile`:

```
web: bundle exec rails s
release: bin/rake db:migrate
```


##### *Resources Referenced*
* React: https://reactjs.org
[React]: https://reactjs.org
* Redux: https://redux.js.org
[Redux]: https://redux.js.org
* Enigma: https://public.enigma.com
[Enigma]: https://public.enigma.com
* Google Maps API:  https://developers.google.com/maps/documentation/javascript/tutorial
 [Maps API]: https://developers.google.com/maps/documentation/javascript/tutorial
 * Wikipedia API: https://www.mediawiki.org/wiki/API:Main_page
 [Wikipedia's API]: https://www.mediawiki.org/wiki/API:Main_page
 * Bundler 2: https://bundler.io/blog/2019/01/03/announcing-bundler-2.html
 [Bundler 2]: https://bundler.io/blog/2019/01/03/announcing-bundler-2.html
 * Postgresql: https://www.postgresql.org/download/
 [postgresql]: https://www.postgresql.org/download/
 * google-maps-react: https://github.com/fullstackreact/google-maps-react


##### *Resources Used*
* A great [post by Bruno Broem](https://medium.com/@bruno_boehm/reactjs-ruby-on-rails-api-heroku-app-2645c93f0814) which is an update on another great post.
* [This one](https://medium.com/superhighfives/a-top-shelf-web-stack-rails-5-api-activeadmin-create-react-app-de5481b7ec0b) from Charlie Gleason; it's not only helpful, it's entertaining.
* And [this](https://www.fullstackreact.com/articles/how-to-get-create-react-app-to-work-with-your-rails-api/) from the fullstackreact folks. It's from 2016 but just about still works as described. 
