# Tried and true

Tried and True (TNT) is a recipe management app that aims to bring simplicity to you meal planning and recipe management work flow.

After the massive success of Movie Sync's initial launch. We fell in love with that feeling of launching something cool.

"What else do you have in your Figma account that you wanna build?", I asked Tim.

"Well nothing here but I have been thinking for a long time that I wanna make something to take the hassle of thinking about what to eat out of the way you know?"

"Well, ideas are cheap, executions baby!"

Tim and I are both very action driven, so it's only a few hours of talking and planning before Tim started making his magic on the big canvas of Figma.

And just like Movie Sync. I started planning out the infrastructures.

NoSQL really opened up my eyes but I find the lack of strict structure was rather a con than a pro. I much prefer traditional relational databases because it catches errors for me, like my dear friend Typescript:)

It was the same time on reddit I saw this framework called Hasura Graphql Engine. I've dabbled in with graphql but never have I use graphql in a proper app. I do know that writing resolvers on the backend is a total pain in the butt. If Hasura can take care of that for me then lets give it a go!

Whenever I want to learn something new, I first make a TODO list just to get the gist of the technology, and then I jump straight a big project. I truly believe frustration === learning.

As I was making the little TODO list, Tim on the other hand already got the first two tabs' layout done. Sweet, better hurry myself up now before Tim runs out of stuff to do.

I quickly mocked out our design for the database. Of course we need a user table. Then recipe and ingredient table. We'll implement the reset later but this should do for the views in the first tab. After I have the database setup. This is what Hasura spit out for me.

Boom! You have CRUD right there! WOW that is too easy.

With good Agile practices in mind, I started to setup the boiler plate for the front end. I am sticking with Ionic because I really enjoyed the simplicity of it and again I did not anticipate the app to be super resource heavy.

But before I can do anything cool, I need to work out the authentication flow. Hasura provide you with access control but you need to implement a jwt system yourself. No problem, my best friend Firebase there does authentication for free!

There is one road block though. I needed to add additional Hasura specific custom claims to the jwt so Hasura can determine the role of the user. But you can't do it when you first create the user, you need to do it after the first JWT token is returned to you, at that point you are authenticated on the front end but not on the backend and any subsequent Graphql request you make will throw errors.

Again some good old head banging on the desk and Googling. I was inspired with a solution. The client will first sign in, which will trigger an onUserCreate() function in Firebase that sends an request to Hasura with admin privilege to add the user into the "user" table. At the same time I have the client call another cloud function that returns a new token with the custom id. And I force refresh the Token on the front end. Boom! The user is now authenticated both on the frontend and backend.

Now let's build the actual UI for the recipes tab. The main recipes collection view consist of a bunch of cards with infinite scroll and a search bar. With the help of Apollo client, everything was so easy. All I have to do is just mount the Query custom hook on the view and I have all the recipes! I left a wild card where clause in the query so I can change that with my search bar input and it will return me only the recipes with a partial matching name. The infinite scroll was even simpler, all I had to do was call the refetch function returned by the query custom hook and fetch the next batch of 10 recipes!

Now comes to the juicy part of the app, we need to enter ingredients in, but we don't just want the ingredients, we want to actually parse it and transformed into meaningful data we can use later-on on other part of the app.

Never code something unless you know there isn't a open source code base out there first. So I got on to Github and started searching. There was quite a few ones out there that can parse ingredients, but they are all regular expression based which means that they can only parse simple recipes. Then I came across this parser based on some kind of Machine Learning from 130k manually parsed ingredients, this looks much better than the regex based one so I decided to go with this one.

But nothing is this easy ever in software dev. The author build a docker container for it for ease of use but he didn't include the model file, which means I have to train it myself. With no prior experience with ML what so ever. And my lack of experience with docker didn't help at all. So I whipped out docker's doc and started reading like a hungary mice feasting on rice.

It turns out docker wasn't even that hard, it's essentially an automated script to instruct an virtual machine on how to build your app. Once that is figured out, I spun up a 16 core vm that I didn't even need and ran the training script.

It took about an hour and somehow it only utilized 2 core but whatever, I got the model file I needed. I quickly tested out some simple ingredients and it worked like a charm.

Alright lets package this thing up. I first thought about making this container a dependency and have my server call it via child process but that will make it impossible to manage. So instead I created a simple flask server that my main server can send an array of strings and retrieve the parsed ingredient in JSON.

To intergrade this parser, we have to make a separate microservice on top of Hasura. for the sake of simplicity, I used express and have Hasura forward the request to it. The webhook calls the parser right here and I run a raw sql query to insert the returned data manually (which turned out to be a huge mess afterwards)

The parser got the job done but it was only trained on one particular style of ingredient format. As hacky as it sounds, I used another parser that is regex based and simply put it in front of the main parser to format the raw string a little bit.

And with a few hundred commit of front end code. This app is coming alive!
