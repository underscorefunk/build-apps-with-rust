# Form Actions with `<ActionForm>`

## Submitting requests to a server (via url)

The internet works on a basis of request/response. We ask for something, and a server responds (or doesn't, which is still a response of sorts). When we enter a URL in our browsers, a request for that resource is sent to the server handling requests for the domain. This is an actual computer or network endpoint somewhere on the web. The DNS system looks up the address of that computer and sends the request on down to that actual address. If all goes well, the server will respond with a new page, static resource, etc—what you asked for. 

We can configure those requests by adding query string variables after the url, separated by a question mark (`?`). Each parameter and argument pair—often called a key/value pair—in the query string is separated by an ampersand (`&`). Query strings can not have spaces or certain special characters. You can imagine how problematic it would be if the value of one of the query string parameters had an ampersand in it. It's customary to encode those special characters for use as query string values/arguments. You'll have seen this all over the web. If you see %20, that is the encoded value of a space. Interestingly, they're called query strings because it is adding specificity to our resource (response) query (what we're asking for).

For example:
`https://some-non-existant-store.com/catalog?page=1&per_page=12&title=Cool%20Products`

This is the most common way that we make requests online. Adding a url in an image source, entering a url into your web browser, linking a css file, they all use this same approach. Take the request and respond with a resource.


## Submitting requests to a server (via forms)

### Get method

It is possible to submit requests to servers while allowing user input. Forms are the foundational web tool for doing this. We do this by authoring a form with `<form> ... </form>` tags, and setting the action property on it to the url that will process the request. It's where the form will be sent. One of the methods forms can use is called `get` which will embed the form's fields to the action as a query string. You can think of the form field ids as the parameters, with their values as the arguments. 

For example:
```html
<form 
	  action="https://some-non-existant-store.com/catalog" 
	  method="get"
>
	<input type="number" id="page" value="1" />
	<input type="number" id="per_page" value="12" />
	<input type="text" id="title" value="Cool Products" />
	
	<inpt type="submit" value="Submit request">
</form>
```
> Clicking the submit button would send the input field values a query string appended to the action url.

The benefits of using `get`:
- URLs are visible
- URLs are stored in your history with the query string allowing for navigation to and from the submitted form's response

Drawbacks of using `get`:
- URLs are visible and make data that is submitted public
- Data is stored in the history and can be introspected
- URLs have a maximum length, limiting the amount of data that can be sent

### Post method

Changing the method of our form to `post` will take us to the same url as written in the action property's value. However, the post method will collect the data and submit it as a payload _with_ the url to the server. `Post` does not append the form's values to the request as a query string.

Benefits of using `post`:
- Data is hidden from the history
- Data doesn't pollute the url
- Complex and multipart data can be sent

Drawbacks of using `post`:
- When pressing reload, it's possible to resubmit the form

## Why forms are an important part of the web platform

Direct resource requests and form submissions are the two main ways that we can use the web platform to submit requests and retrieve new data from servers. An important thing to note is that forms work out of the box. We do not need to iterate over fields, extract values, and then submit some javascript event. They just work, even with Javascript disabled! I encourage you to get comfortable with forms and what we're about to learn here. Forms, though old technology, will make your applications more accessible, robust, and depending on how you load your application, faster to load the initial useful render.

## The lifecycle of a form

