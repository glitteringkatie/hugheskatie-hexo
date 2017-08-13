---
title: listsaredumb pt 1
date: 2017-08-12 23:56:54
tags:
- reason
- useless projects
cover_index: https://reasonml.github.io/reason-react/images/logoMini.png
---

Recently I’ve been increasingly interested as making really dumb and useless projects, maybe things that might get people annoyed at their existence. This is mostly because I always feel pressure when making a project that it should provide some value and then I get hung up on that and rarely make things because I think they should all be Good™. Recently a list of JavaScript Influencers came out which resulted in some conversation and ultimately this tweet that inspired me for some reason:
{% twitter https://twitter.com/sarah_edo/status/896025355764457472 %}


In my quest to embrace Bad™ and get to learning/practicing a lot faster, I came up with an idea for what listsaredumb.com could be. I’m planning to take the standard TODO app and make it utterly useless. The features that nobody asked for will probably be the following:
* Randomly insert in todo items using mashed words associated with the intended todo item
* Randomly delete or complete items
* Show items in a non-linear format (scattered)

Since the point in my pointless app is to get to learning something faster, I chose to use ReasonReact for this project. I’ve been wanting to play with it since React Conf but just haven’t gotten around to it. The nice thing is that the [ReasonReact example repo](https://github.com/chenglou/reason-react-example) had a todo app already in that. My first step was to get over the fact that I always think I need to start from point 0 and create everything myself. The second step, ironically, was making my todo list so I could calm down (context: this was when literal Nazis were terrorizing Virginia). Third step was a face mask.

Once I actually got into the code, I figured that the function responsible for saving each of these todos must have the word “save” around it somewhere. I searched for just that and used my best detective skills to figure out I probably want to be focusing on handleNewTodoKeyDownEvent. Great. There was some weird syntax around state that I guessed was setting it?? I have yet to look up the at symbol in Reason tbh.

```
let todos =
    state.todos @ [
        {id: string_of_float (Js.Date.now ()), title: nonEmptyValue, completed: false}
    ];
```

So initially I just copied the object that looked like it was being pushed (added? inserted? idk) onto the todos array in state and made another one where title was just “cats” because that’s my go to sample data.

```
let todos =
    state.todos @ [
        {id: string_of_float (Js.Date.now ()), title: nonEmptyValue, completed: false},
        {id: string_of_float (Js.Date.now ()), title: “cats", completed: false}
    ];
```

I booted up the app and added a todo item expecting the item to be added as well as a task just saying “cats”. This did not happen. “cats” was no where to be seen. I did notice that in the corner the item count read as 2 but I was only seeing 1 item. I added a few more items and noticed that I only saw half of what was listed according to the count. My hypothesis was that something must be working with the IDs and since they were both being made within the same tick, `Js.Date.now ()` was probably not a unique-enough ID anymore. Hardcoded one ID to “3” because ¯\_(ツ)_/¯ and was able to see the “cats” pile up.

Enter: my first rabbit hole. I initially tried just adding 1 to the date. That didn’t work. The compiler complained about the types being added. For some reason instead of just looking up float addition (`+.`, by the way) I decided to write up this whole recursive thing that checked for key existence and either used the current date or tried it all again but with date + 0.1. Again: hit the type issue so I finally looked up float addition and fixed that. Still no luck. After too long I realized because the recursive function was looking at the _saved_ todos, the pending todo ID I just created and wanted to also be checked was not being included. My trial and error process continued, which included trying to write a recursive function with optional parameters without realizing what I was doing, finding [BadErrors](https://github.com/reasonml/BetterErrors) since the output/error is pretty unhelpful (this made it marginally less shitty), installing [the appropriate plugin for vscode](https://github.com/reasonml-editor/vscode-reasonml), and stream-of-conciousness texting my friend this whole process.

I stepped back at this point, said yolo, and then just concatenated “2” onto the end of the string ID. Recompiled, everything went well, pulled up the app, added a todo item, and there it was: “cats” after every item I added. Hallelujah.

My next task was to add “cats” at a random chance. At first I figured I could set the array to a variable above and then conditionally add the second item in. This didn’t work for some reason? I still need to look up that @ sign. There was some weird syntax highlighting happening and bad errors so I put the array back how it was and then just used `Random.int` with a cap of 3 and a ternary to change how state would be altered.


```
let todos = Random.int 3 != 0
    ? state.todos @ [
        {id: string_of_float (Js.Date.now ()), title: nonEmptyValue, completed: false},
        {id: string_of_float (Js.Date.now ()) ^ "2", title: "cats", completed: false}
    ]
    : state.todos @ [
        {id: string_of_float (Js.Date.now ()), title: nonEmptyValue, completed: false}
    ];
```  

This... actually worked. Now I had cats appearing 66% of the time. I like those odds (also I miss my cat—his grandparents are currently catsitting but I see him in 5 days). The only task I had left for the night was figuring out what API I wanted to use. I searched a little bit and found [Twinword Word Associations](https://www.twinword.com/api/word-associations.php) which is a freemium model and essentially looked like what I wanted. I felt like that was a good place to stop for the night and get to reading Watchmen, and by “reading Watchmen” I mean accidentally writing this post instead.

So what do I expect to happen in the next episodes of this project? Pay for api, talk to api, randomly delete todos, randomly complete todos, redesign, Mom!? argument, police, fleeing the scene, hiding in a dumpster, crashing on your couch for a week because

![image](https://uproxx.files.wordpress.com/2015/06/j-ralph-homeless.gif?w=650)

Disclaimer: This may or may not have all been done with Parks and Rec playing in the background.
