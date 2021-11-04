# Part 2: REST API Implementation

Now that we've seen how to work with databases for long-term storage of information within our system, the next step is figuring out how to get information into and out of the application we're writing.  While we've not yet gotten to the step of writing the dynamic parts of our frontend, when we get there, we'll see it's impossible to have Javascript code directly call the Java code we've written.  The JVM runs a tight ship when it comes to security and who is allowed to do what, and Javascript from a front end will be promptly rejected.

So, how does the frontend communicate with the backend?  The answer is over a REST API, which lets us write Java code that can be interreacted with via HTTP requests to the web application server that runs the backend code.  Here, we'll see how that works and how to write your own REST APIs.


## Background

We covered some background on what APIs are during the Testing Workshop and the Backend Design Workshop, but we'll repeat that content here in case you've forgotten exactly how they work and what types of requests you can make.

An API (or Application Programming Interface) defines a specification for how to communicate with a software system. An API lays out what functionality is offered, what parameters must be provided when making calls, and provides some guarantees about the return type given the preconditions of the call are satisfied. Fundamentally, a complete and well-documented API makes it easier to use a piece of code as a “building block” for building a larger system. 

A REST API is a subtype of API that is geared towards Internet-facing systems. The goal of REST APIs is to provide painless communication and interoperability between these Internet-facing applications and systems. REST APIs typically allow (often with some sort of authentication) a manipulation of the various types of entities (recipes, inventory, etc.) that a web application revolves around.

The key concept behind REST APIs is that they are inherently stateless, that is, everything that a server needs to know to complete a request or a client needs to know about the results is contained within the request itself. This is contrary to the notion of a server maintaining a session where it stores information about a series of transactions with a particular client, or cookies where a client keeps similar information stored on its end.    

Typically, REST API operations are carried out over HTTP; consequently most REST API actions are based around HTTP verbs (POST, PUT, DELETE, and so on). The actions typically available are:

| HTTP Verb	| Action             |	Example	       |Result  | 
| --------- | ------------------ | --------------- |------- |
| GET	    | Retrieve record(s) | GET /users/	   | Retrieves all users|
| DELETE	| Delete record(s)	 | DELETE /users/1 | Deletes user with id 1|
| POST	    | Create record	     | POST /users/	   | Creates a new user|
| PUT	    | Update record	     | PUT /users/6	   | Updates user 6|


There are [other](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) HTTP verbs available, but the four above will let us accomplish everything that we need without making things too complicated.

Our prior discussion of REST APIs was limited to treating them as a black box -- all that we cared was when you send data to an API endpoint, the API will go do something and then respond accordingly.  Today we're going to figure out how that something is done by seeing how to implement a few different API endpoints.


## REST APIs in Spring

Fundamentally, all REST APIs work by sending and receiving [JSON](https://www.json.org/json-en.html).  We saw how JSON works before, but at its core, it's a way of "flattening" objects down into key-value pairs so that you can easily transmit them across a network.  From our point of view, this means that on the frontend you build up a JSON structure based on what the API expects, send it by making an API call, and then will get back JSON according to the API's specification.

From the backend, however, we don't work directly with JSON.  Instead, we work with normal Java objects, and Spring will handle converting these to and from JSON.  This is a process known as serialization (going from Java to JSON) or deserialization (going from JSON to Java).  The advantage of this is that we get to enjoy all of the benefits of static typing when writing the REST APIs, and the pain points of "but wait, this data doesn't look like what I expect" is limited to just two points: 1) when the data gets deserialized to Java objects, and 2) when you're working with the serialized JSON on the frontend.

We'll see how this works in a few minutes when we look at method parameters and return values from the API endpoint methods that we write.

## Implementation

We'll be working on some of the API endpoints for your `Ingredient` object.  To get started, we'll need to create a new Java class to hold these methods.  The way that the CoffeeMaker (and later, iTrust2) systems are architected is that for each class that gets persisted to the database (that is, the ones under `models.persistent`) there's a matching class that provides the API endpoints for doing things with that object (under `controllers`).

Create a new class, `APIIngredientController` in the `CoffeeMaker.controllers` package .  `APIIngredientController` must extend the base `APIController` class.  Once you've done that, add the `@RestController` annotation to the class.  This is used to indicate to Spring, "this is a class where I want to keep REST API endpoints" and that helps it know that parameters to/from these methods should be automatically serialized/deserialized.  Once you've done that, you should have a class that looks like the following:

```
package edu.ncsu.csc.CoffeeMaker.controllers;


import org.springframework.web.bind.annotation.RestController;

@RestController
public class APIIngredientController extends APIController {

}

```

Let's look at a general template for endpoint methods.  The code below is a method with an optional list of parameters that will return a `ResponseEntity`.  The method will contain code appropriate for the action and after the action is complete, we'll create a `ResponsEntity` that will wrap up the data to return with an `HttpStatus` that indicates if the result was successful or not. 

The method has a leading annotation that indicates the type of HTTP request we're making, the URL that a caller would use for this request, and any additional information needed for the request.


```
@MappingType ( URL this API endpoint is available at )
public ResponseEntity methodName ( PARAMETERS ) {

    /* Go do so something */

    return new ResponseEntity( Data to return, HttpStatus indicating what happened );
}

```

Suppose we're creating an API endpoint to retrieve a single Ingredient. The method might look like:


```
@GetMapping ( BASE_PATH + "ingredients/{name}" )
public ResponseEntity getIngredient ( @PathVariable final String name ) {

    final Ingredient ingr = ingredientService.findByName( name );

    if ( null == ingr ) {
        return new ResponseEntity( HttpStatus.NOT_FOUND );
    }

    return new ResponseEntity( ingr, HttpStatus.OK );
}

```


Let's discuss each part of the method:
  * `@GetMapping` indicates that this API endpoint will respond to GET requests.  There's also `@PostMapping` and so on for responding to the other types mentioned above.
  * `BASE_PATH` is defined in `APIController` and is used to create a common URL structure for all API endpoints.
  * `ingredients` indicates the collection of information that we're working with and is typically a similar name to the persistent classes.  
  * `{name}` creates a _wildcard_ where the user can pass in the name of the specific ingredient they want.  Spring will handle this automatically, provided that....
  * You also have something like `@PathVariable final String name` defined.  This says "I expect there to be a variable passed in by the user, and it's going to be part of the path (URL)".  Matching is done on name, which means you can pass multiple parameters (we'll see a slightly tortured example of this in a moment).
  * The body of the method calls methods in the persistent class to get the `Ingredient`.  This is the "go do something" portion of the method.
  * `return new ResponseEntity( ingr, HttpStatus.OK );` says we'll return a ResponseEntity, which is one of the special types that Spring will automatically turn into JSON data for us.  `HttpStatus.OK` is used to indicate that everything worked.  More response codes you can use are available from [http.cat](https://http.cat/).  Generally you'll use 2xx codes to indicate everything worked fine, 4xx to indicate the user did something wrong, and 5xx to indicate the server did something wrong.

You can then call this API endpoint with a call similar to `GET /api/v1/ingredients/vanilla` and would get either the details of the ingredient if it exists, or a 404 NOT FOUND if it doesn't.

As mentioned above, you can pass multiple parameters to the API endpoint.  This means if we wanted to create some sort of "universal" API endpoint for retrieving any object, we could do something like:

```
@GetMapping ( BASE_PATH + "{object}/{field}/{value}" )
public ResponseEntity getObject ( @PathVariable final String object, @PathVariable final String field,
        @PathVariable final String value ) {
    try {
        switch(object){
		    case "Ingredient": // go do something
			break;
			
			case "Recipe": // go do something else
			break;
		}

        if ( null == obj ) {
            return new ResponseEntity( HttpStatus.NOT_FOUND );
        }

        return new ResponseEntity( obj, HttpStatus.OK );

    }
    catch ( final Exception e ) {
        return new ResponseEntity( "Something went wrong with your request :: " + e.getCause(),
                HttpStatus.BAD_REQUEST );
    }

}

```

If I call it with values that exist in my database, for instance `GET api/v1/Recipe/name/SweetLoot` I get a valid response back: `200: {"id":1,"name":"SweetLoot","price":50,"coffee":1,"milk":2,"sugar":3,"chocolate":4}`.  If instead I call it with nonsense (`GET api/v1/nope/nothere/orhere`), I likewise get back nonsense: `400: Something went wrong with your request :: null`.


This is _not_ strictly a good idea, but it shows how we can have multiple parameters as part of the URL, and Spring will automagically map from each one in the URL to one within the Java method parameter list.



Thus, we've so far seen how we can define GET API endpoint that take in one or more URL parameters, do something, and finally return a response back to the user.  This works fine for a GET request if we have one or a few parameters that we want to send to constrain what happens.  But, how do we handle sending other requests, such as creating or updating an object?  

Let's look at an example from `APIRecipeController`:


```
@PostMapping ( BASE_PATH + "/recipes" )
public ResponseEntity createRecipe ( @RequestBody final Recipe recipe ) {

    final Recipe db = recipeService.findByName( recipe.getName() );

    if ( null != db ) {
        return new ResponseEntity( HttpStatus.CONFLICT );
    }

    try {
        recipeService.save( recipe );
        return new ResponseEntity( HttpStatus.CREATED );
    }
    catch ( final Exception e ) {
        return new ResponseEntity( HttpStatus.BAD_REQUEST ); // HttpStatus.FORBIDDEN would be OK too.
    }

}
```

Let's unpack what we're doing here, a piece at a time:
  * `@PostMapping`: This says we're going to respond to a POST request at the given URL.
  * `@RequestBody final Recipe recipe`: This says that we expect to get a Recipe that is sent as JSON data alongside the request.    For now, we won't focus on how the frontend does this, but the important part is that you can send one "blob" of JSON alongside the request.
  * `return new ResponseEntity( HttpStatus.BAD_REQUEST );`: It's good practice to surround anything that can fail inside of a try/catch block, so that if the thing fails, we gracefully return an error to the user rather than letting the Exception propagate upwards to the user in the form of a 500 error.


## Special Cases, Caveats and Pitfalls

This really is basically all there is to writing (simple) REST APIs in Spring.  However, there are a few more things to be aware of before you start writing many of your own:

  * When want to retrieve a bunch of objects, you don't return a `ResponseEntity`, but instead use a `List`.  A list with all matching elements we found indicates we got something, and a list with no objects means there was nothing.   Spring will automagically serialize this to JSON too:

```
@GetMapping ( BASE_PATH + "/recipes" )
public List<Recipe> getRecipes () {
    return recipeService.findAll();
}
```
  * GET (and DELETE) routes don't support extra JSON data sent alongside the request (the extra data is what would be turned into a `RequestBody` as in the POST example).  Only POST and PUT let you do this.  Thus, if you want to make a complicated retrieval request (for instance, "Give me all users whose name starts with `Ja` born between 1950 and 1965") it would create a really nasty GET request because of all of the parameters that would go in the URL.  You thus would be better off with a POST request, where the details of what you want become a blob of JSON sent with the request.  This is reasonably common for performing requests where the server will need to do more extensive lookup.
  * Spring makes a best-effort approach to convert from your JSON data into Java objects of the appropriate type.  However, if you do something wrong, the conversion will fail and your request won't go through.  For instance, if we have an API endpoint such as:

```
@GetMapping ( BASE_PATH + "/recipes/id/{id}" )
public ResponseEntity getRecipeById ( @PathVariable final Integer id ) {
    return new ResponseEntity( recipeService.findById( id ), HttpStatus.OK );
}
```

and then call it as:

`GET /api/v1/recipes/id/notreallyanumber` Spring will give us an error back:

`> Failed to convert value of type 'java.lang.String' to required type 'java.lang.Integer'; nested exception is java.lang.NumberFormatException: For input string: "notreallyanumber"`

This is pretty easy to quickly solve (or better yet, never cause in the first place) when making simple GET requests such as this.  However, if you're making a POST or PUT request with a more complicated object, and it can't be properly deserialized, it can be a bit more difficult to track down what's gone wrong.  In this case, you'll likely want to use the Debugger in your browser, and inspect the request that was sent to the server to see exactly the structure of the data sent.  In Firefox, you an access this by clicking F12, then Network, then clicking on the request that failed (it'll probably be marked 4xx, and in purple, to indicate things didn't work) and then the Request tab to see the data sent.  Other browsers have similar functionality you can use.
  * If you forget the appropriate annotations on your method parameters (`@RequestBody`/`@PathVariable`) then Spring won't know to match data received from the API endpoint to your method parameters, and you'll get `null`.  If you're not getting anything and think you should be, take a look at it.

# Testing

## Testing
You'll need to update all existing tests so they compile and pass! 

## Testing REST APIs

You should always test your REST APIs!  If you need a reminder about how test REST APIs, take a look at Testing Workshop ! 

Return to [the main page](README.md) for the tasks to carry out on your own and as a team.
