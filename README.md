
# Tutorial: A Bridge Between DM and JS
```
    ____  __  ___   ___            _______
   / __ \/  |/  /  ( _ )          / / ___/
  / / / / /|_/ /  / __ \/|   __  / /\__ \ 
 / /_/ / /  / /  / /_/  <   / /_/ /___/ / 
/_____/_/  /_/   \____/\/   \____//____/
```                                      

## INTRODUCTION
> [!WARNING] Create an introduction

## CONTENTS
1. The Relation between DreamMaker and Javascript
2. Data Referencing
3. Browse & Output
4. Server to Client
5. Runtime Concerns
6. Debugging & Tools
7. Building an inventory

   
## 1: The Relation between DreamMaker and Javascript
	
> [!IMPORTANT]
> .dmf info:<br>
> `window1="is-visible=0,is-default=0"`<br>
> `window1.browser1="is-visible=1, is-default=0"`

In large part, browsing HTML documents to clients is essentially "Take this text" and "send to 'this' client". The client recieves the payload, along with a task description.
It looks like `(client) << browse(data, "window.browser", params=null)`. 
 `browse()` `data` in `"window1.browser1` to `client`.
_though we will discuss `browse()` more in detail later._

With exception of building the HTML file, DreamMaker isn't concerned about were to put content, it only cares for which window and browser to use. The browser on the other hand do care about the data-content. It takes whatever is inside _data_ and renders it as any web browser does. It accepts HTML, CSS and JS by default. **Our** job is to combine DM and JS to format _data_ and create a valid html document, tailored to our needs
> [!CAUTION]
> Rewrite me



**Lets create an example:**
```c
mob
	Login()
		..()
		name = "Average Joe"

client
	verb
		test_browser()
			set name = "Browse"

			var/body = {"
				<!DOCTYPE html>
				<html>
					<head>
					</head>
					<body>
						<div>Hello [usr.name]</div>
					</body>
				</html>
			"}
			src << browse(body, "window=window1.browser1")
			winshow(src, "window1", 1)
```
> [!NOTE] 
> - We are making browser interraction `client`-specific. _More on this later..._
> - `<!DOCTYPE html>` is a **requirement** to all HTML documents.

Great! Now we need some **Javascript!**
Let's create a function were we _change_ the value of `usr.name`. <br>
To accomplish this, we need two things: 
- A html hook for Javascript to recognize.
- Get ourselves a proper html element we can apply changes to.
- A function which applies desired changes to our html element.

With Javascript we can hook to an element by identifying tag(`<></>`) ids. `(tag).id` is a unique identifier we use to recognize specific elements. <br>
`getElementById()` looks for a tag within the document that has the specified id, and returns it as an element.

Let us construct an example which changes inner contents of a tag, then responds back to the client with a interface "command".

```html
<!DOCTYPE html>
<html>
	<head>
	</head>
	<body>
		<div>Hello <span id="user-name">[usr.name]</span></div>
		<script>
			<!-- Recieves data from DM -->
			function changeInnerHTMLById(id, content) {
				const elem = document.getElementById(id);
				if(elem) { elem.innerHTML = id.value; }
				responseToDM("id '" + id + "' has been changed to '" + content + "'.");
			}
			
			function responseToDM(text){
				// response = client/verb/response(), text = text;
				// BYOND.command("verb a r g s")
				BYOND.command("response " + text);
			}
		</script>						
	</body>
</html>
```
> [!IMPORTANT]
> **Encode/Decode data between DreamMaker to Browser!** <br>
> Encoding between DreamSeeker & Webview2(browser) is essential to ensure special characters are escaped.
>> BYONDs web interface encodes by default.


Let us take a look at DreamMakers side of things.<br>
Introducing `output()` to the equation, we can specifically call functions within the browsed document. 
```c
// JS(...) helps us pass multiple arguments as a single string.
#define JSArgs(T...) list2params(list(T))
client
	verb
		change_name(new_name as text)
			// Calls changeInnerHTMLById("user-name", new_name)
			usr << output(JSArgs("user-name", new_name), "window1.browser1:changeInnerHTMLById")
		response(text as text)
			src << "browser callback: [url_decode(text)]"
```
This should output `browser callback: id 'user-name' has been changed to '[new_name]'`.



## 2. Data Referencing

Let us talk about referencing data to our browser.<br>
Referenced data is converted to string at runtime, before being passed to browser. The standard string formatting in DM is: `"[object.data]"`.<br>
In some cases you want to reference appearances or images.You can refer to images by typing "\ref" before a datavalue, like so: `"\ref[object.appearance]"`. <br>
This applies to `/image`, `/icon` or image files(.png, .jpg, .svg, etc...). <br>
`\ref[]` references an index in the games object table.<br>
This tells the browser to look within the object tree, fetch the indexed data, which in this case returns `/image`.

**The flow of data**
1. DreamMaker arranges data and formats it as a string. We call the result of that string parameters.
2. DreamMaker passes parameters to a JS function within a desired window browser.
3. The browser's javascript can use the params and interracts with the document, deciding what to do with the data and where to put it.

**Let us take a look at an example**<br>
We want to display a small, simple character screen. It contains the players name and appearance.<br>
Lets build on our previous example by adding an empty image tag and a function that changes it.

Before we make that happen, we need to tailor our document a bit more. We need a more generalized javascript function which can handle multiple arguments by parsing params in `[key,value]` pairs.<br>
> [!TIP]
> DM allows the use of special characters in stringblocks as such: `@{""}`


```html
<!-- We update our tags by adding <img></img>  -->
<div><img id='user-appearance'></img>Name: <span id='user-name'>[usr.name]</span></div>
```

```js
@{"
	...
	<script>
		function setValueById(param) {
			// Create anonymous object, which becomes a body to param
			const obj = {};
			// split "key1=value1&key2=value2", which returns an array.
			// "key1=value1&key2=value2" -> ["key1=value1","key2=value2"]
			const pairs = param.split("&");
			
			for (const pair of pairs) {
				// itterate over array "pairs", and split them to two datapoints(key,value).
				// "key1=value1" -> {key1=value1}
				const [key,value] = pair.split("=");
				
				const elem = document.getElementById(key);
				
				//Create an exception to image elements, as we aren't changing innerHTML.
				if(elem.tagName == "IMG") {
					// set src directly.
					elem.src = value;
				} else {
					elem.innerHTML = value;
				}
			}
		}
	</script>
	...
"}
```
> [!TIP]
> Image size is inherited by default. Icon size of 32x32 results in an image 32x32px.

```c
#define JS(T...) list2params(list(T))
client
	verb
		change_name_and_appearance(new_name as text)
			// typehint to client.mob
			var/mob/m = src.mob
			src << output(JS("user-name"=new_name, "user-appearance"="\ref[m.appearance]"), "window1.browser1:changeValueById")
```



## Browse() & Output()

Let us delve a bit deeper into things and talk about key features and differences between `browse()` and `output()`.

> [!NOTE]
> **Browse(content, control_id, params)** <br>
> The primary feature to browse, surprisingly, has very little to do with browsers. The main use-case is to open/build a new window frame, interact with already existing windows or send cache files. The main reason to why `browse()` is used in our example, and why it is great in our case--it both builds a window and outputs data.

> [!NOTE]
> **output(msg, control)** <br>
> The key feature to output is as self explanatory as the function name itself. It acts as a bridge between the game and the interface. It outputs any message as text to potentially any part of your skin. However, it also supports function calls to browsers, which is a key function we need to bridge the gap between the game and HTML content.

**Are they really functions, though?** Actually, no. They are **instructions**. They explain to the client what do to with given information, thus computing happens client-side, not server-side.

A very important use of `browse()`, which is often overlooked is caching files to clients. 

> [!NOTE]
> **Client** <br>
> In the context of this topic we are referring to the client, not the type `/client`. 


The main difference, and what tells them apart, is `Browse()` is mostly used to "create" or "open" a new windowframe,. `output()` is used to communicate with a browser which is already created.


## Tips & Hints
<details>
  <summary>BYOND Web Interface</summary>
 
 	BYOND.command("verb arg1 arg2 ...");
	BYOND.winset("id=control_id&property=value&...");
	BYOND.winget("callback=js_cb&id=control_id&property=is-checked,size,background-color");

</details>
