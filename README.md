
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

## 1: The Relation between DreamMaker and Javascript
> [!IMPORTANT]
> .dmf info:
> `window1="is-visible=0,is-default=0"`, 
> `window1.browser1="is-visible=1, is-default=0"`
> 
In large part, browsing HTML documents to clients is essentially "Take this text" and "send to 'this' client". The client recieves the payload, along with a task description.
It looks like `(client) << browse(data, "window1.browser1", params=null)`. `browse()` is the task descriptor, which tells `client` to browse `data` with the interface element `window1.browser1`.
_though we will discuss `browse()` more in detail later._

Essentially, DreamMaker isn't concerned with data's content, it only cares for where to put it. The `"window1.browser1"` element on the other hand do care about _data_. It takes whatever is inside _data_ and renders it as any web browser does. It accepts HTML, CSS and JS by default. Our job becomes to format _data_ to create a valid html document, tailored to our needs.

**Lets create an example:**
```c
client
	verb
		test_browser()
			set name = "Browse"
			
			// Lets create our "data" variable, we call it 'body'.
			// This will become our final HTML composition,
			// which we will output to our client.
			var/body = ""
			
			// We can add html formatted strings to our HTML build data.
			body += {"
				<!DOCTYPE html>
			"}
```
> [!NOTE]
> Make note of how we stay in `client` type. More on this later...
> `<!DOCTYPE html>` is a **requirement** at the top of our documents.

**lets create a simple html window** that displays `client`'s `mob.name`.
```c
			body += {"
				<!DOCTYPE html>
				<html>
					<head>
					</head>
					<body>
						<div>Hello [usr.name]</div>
					</body>
				</html>
			"}
```

**How does it work?**
Lets compose the code and run:
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
			winset(src, "window1", "is-visible=true")
			src << browse(body, "window=window1")
```

Great! Now we need some **Javascript!**
Let's create a function were we _change_ the value of usr name. To accomplish this, we need two things: A hook for Javascript to recognize and change, and the Javascript function.
Lets look at the html code.
```html
<--var/body-->
<!DOCTYPE html>
<html>
	<head>
	</head>
	<body>
		<div>Hello <span id="user-name">[usr.name]</span></div>
		<script>
			
			function ChangeName(new_name) {
				getElementById("user-name").innerHTML = new_name;
			}
			
		</script>						
	</body>
</html>
```
Then we need something that calls that JS function from DreamMaker. This is were `output()` comes in...
```c


```
