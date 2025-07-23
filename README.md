
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
- The Relation between DreamMaker and Javascript
- Data Referencing
- Browse & Output
- Server to Client
- Runtime Concerns
- Debugging & Tools

## The Relation between DreamMaker and Javascript
> [!IMPORTANT]
> .dmf info:
> `window1="is-visible=0,is-default=0"`, 
> `window1.browser1="is-visible=0, is-default=0"`
> 
In large part, browsing HTML documents to clients is essentially "Take this text" and "send to 'this' client". The client recieves the payload, along with a task description.
It looks like `(client) << browse(data, "window.browser", params)`. `browse()` tells `client` what to do with `data`, and acts as the task descriptor.
_though we will discuss `browse()` more in detail later_

Essentially, DreamMaker isn't concerned with data's content, it only cares for where to put it. The `"window.browser"` element on the other hand do care about _data_. It takes whatever is inside _data_ and renders it as any web browser does. It accepts HTML, CSS and JS by default. Our job is to format _data_ to our needs. 
**Lets create an example:**
```c
client
	verb
		test_browser()
			set name = "Browse"
			
			// Document data displayed by browser
			// This will be our HTML builder datapoint.
			var/doc_data = ""
			
			// We can add html formatted strings to our HTML build data.
			doc_data += {"
				<!DOCTYPE html>
			"}
```
> [!NOTE]
> Make note of how we stay in `client` type, instead of `mob`. More on this later...


**How does it work**
