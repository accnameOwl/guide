
# Tutorial: A Bridge Between DM and JS
```
    ____  __  ___   ___            _______
   / __ \/  |/  /  ( _ )          / / ___/
  / / / / /|_/ /  / __ \/|   __  / /\__ \ 
 / /_/ / /  / /  / /_/  <   / /_/ /___/ / 
/_____/_/  /_/   \____/\/   \____//____/
```                                      

## INTRODUCTION
Welcome!

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
> .dmf info:
> `window1="is-visible=0,is-default=0"`, 
> `window1.browser="is-visible=1, is-default=0"`

In large part, browsing HTML documents to clients is essentially "Take this text" and "send to 'this' client". The client recieves the payload, along with a task description.
It looks like `(client) << browse(data, "window.browser", params=null)`. 
> `browse()` `data` in `"window1.browser1` to `client`.
_though we will discuss `browse()` more in detail later._

With exception of building the HTML file, DreamMaker isn't concerned about were to put content, it only cares for which window and browser to use. The browser on the other hand do care about the data-content. It takes whatever is inside _data_ and renders it as any web browser does. It accepts HTML, CSS and JS by default. **Our** job is to combine DM and JS to format _data_ and create a valid html document, tailored to our needs.

**Lets create an example:**
```c
mob
	Login()
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
> - `<!DOCTYPE html>` is a **requirement** at the top of our documents.

Great! Now we need some **Javascript!**
Let's create a function were we _change_ the value of `usr.name`. <br>
To accomplish this, we need two things: 
- A html hook for Javascript to recognize and change
- A function wich edits within the hook.

We can hook an element by using id. `element.id` is an unique identifier we use to recognize specific elements. 
`getElementById()` returns our 'hooked' element.

Let us construct an example, which changes content of element and responds back with a "command".

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
				callback("id '" + id + "' has been changed to '" + content + "'.");
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
> [!TIP]
> Encode/Decode from DreamMaker to Browser!
> encoding between DreamSeeker & Webview2(browser) is essential to ensure special characters are escaped.
> **Example:** no encoding:`space=%20`, with encoding: `space=" "`.
> (!) BYOND web interface already does this for you.

Let us take a look at DreamMaker's side of this. This is were `output()` comes in...
```c
// JS(...) helps us pass multiple arguments as a single string.
#define JSArgs(T...) list2params(list(T))
client
	verb
		change_name(new_name as text)
			// Calls changeInnerHTMLById("user-name", new_name)
			usr << output(JSArgs("user-name", new_name), "window1.browser1:changeInnerHTMLById")
		// In essence outputs 'any' as text. What is important is decoding.
		response(text as text)
			src << "browser callback: [url_decode(text)]"
```

## 2. Data Referencing
Let us talk about referencing data to our browser.<br>
Referenced data is converted to string at runtime, before being browsed. The standard string formatting in DM is: `"[object.data]"`.<br>
In some cases you want to reference appearances or images. You need a memory reference in that case: `<img src="\ref[object.appearance]"`. <br>
This applies to `/image`, `/icon` or image files(.png, .jpg, .svg, etc...). `\ref[]` references a memory location in the RSC.<br>
The browser fetches the image directly from RSC.

**The flow of data**
1. DreamMaker arranges data and formats it.
2. DreamMaker passes formated data to a JS function within a desired window browser.
3. The browser's javascript interracts with the document, deciding what to do with data and where to put it.

**Let us take a look at an example**
We want to display a small, simple character screen. It contains player's name and appearance.<br>
Lets build on our previous example, by adding an image and a function that changes it's appearance.

We need a more generalized javascript function which can handle multiple arguments by parsing params in key=value pairs.<br>
> [!TIP]
> DM allows the use of special characters in stringblocks as such: `@{""}`
 **JS**
```js
var/data = @{"
	...
	<div><img id='user-appearance'></img>Name: <span id='user-name'>[usr.name]</span></div>
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
> [!NOTE]
> The HTML document inherits from appearance's size. If an icon is 32x32, image is scaled 32x32px;
**DM**
```c
#define JS(T...) list2params(list(T))
client
	verb
		change_name_and_appearance(new_name as text)
			// typehint to client.mob
			var/mob/m = src.mob
			src << output(JS("user-name"=new_name, "user-appearance"="\ref[m.appearance]"), "window1.browser1:changeValueById")
```

## Tips & Hints
<details>
  <summary>BYOND Web Interface</summary>
 
 	BYOND.command("verb arg1 arg2 ...");
	BYOND.winset("id=control_id&property=value&...");
	BYOND.winget("callback=js_cb&id=control_id&property=is-checked,size,background-color");

</details>
