
# Tutorial: A Bridge Between DM and JS

```
____ __ __ ___ _ ____  
| _ \| \/ | ( _ ) | / ___|  
| | | | |\/| | / _ \/\ _ | \___ \  
| |_| | | | | | (_> < | |_| |___) |  
|____/|_| |_| \___/\/ \___/|____/
```

## INTRODUCTION
Welcome!
I assume you opened this tutorial post to read some about HTML, Javascript, Browser() and such. **Great!** You have come to the right place. This tutorial will describe to you the bridge between browser content and in-game data

This tutorial will be somewhere within the lower range of intermediate difficulty. It is expected of you to know what datums are, relationships between objects and object data, proc, verbs and 'some' interface knowledge-primarely about browsers. We will not discuss much about the interface within this tutorial. It is recommended you teach yourself HTML and CSS. CSS is the only tool which _really_ changes your HTML document from amateur to professional. __NOTE: Need to add more as the content grows__

##### You will learn about:
- How to properly use `Browse()` and `Output()`
- Building an HTML document with DreamMaker.
- Update HTML element content with in-game data.
- What is the server-client communication of `Browse()` and `Output()`.
- Theory around `Browse_rsc()`.
- Optimizing runtime efficiency by applying critical thinking and `world.tick()` and `buffer`
- Debugging your HTML and JS functions.

## CONTENTS
- The Relation between DreamMaker and Javascript
- Data Referencing
- Browse & Output
- Server to Client latency
- Runtime Concerns
- Debugging & Tools


## The Relation between DreamMaker and Javascript
DreamMaker uses browser, an interface window, as a form of browser-based interaction with players. The browser window accepts HTML code by default, which means you can directly output HTML data to a desired browser window.

Though diving through the reference can seem a bit overwhelming to the average Joe, fret not. Once you understand the principles, the process becomes self-explanatory.

**How does it work**
You start off by visualizing whatever you want to display through the browser, to the player. Then you transcribe your imaginary page to HTML. Today, we will make something simple: "Hello world", followed by the players name, exclamation mark.

```js
mob
	name = "Average Joe"
```

This will be our player for today, and his name is 'Average Joe'.
> [!IMPORTANT]
> By default, `mob.client` is set to `null`. Though, should a client connect to the server and a mob is attached to it, `mob.Login()` is called -> `mob.client` is then set to your client,  instead of `null`.

