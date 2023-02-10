---
author: udinic
comments: true
date: 2012-07-30 08:13:19+00:00
layout: post
slug: a-festivus-for-the-rest-of-us
title: A Festivus for the REST of us!
wordpress_id: 291
categories:
- Android
- Java
- Libraries
tags:
- Android
- Guice
- Proxy
- REST
- REST API
- Retrofit
- Roboguice
- square
---

Quite recently, we added a sync feature to Any.DO. The users can sync all their tasks to Any.DO’s server, allowing them to access their tasks from our Android, iOS and Chrome clients.

Our data structure is a classic fit to [REST API](http://stackoverflow.com/questions/671118/what-exactly-is-restful-programming) architecture. After getting our server ready, it was now the time to teach the Android client how to “talk the talk”. I didn’t want to use simple HTTP requests, constructed to use our REST API. I wanted to use some library to make my life easy, and my code readable. I also didn’t want to reinvent the wheel, so I did my research. I found some good and simple libraries for Android, as [Resty](http://beders.github.com/Resty/Resty/Overview.html), but then I found a more elegant solution for that – RestAdapter in [Retrofit](https://github.com/square/retrofit) by Square.

<!--break-->

Using RestAdapter, I can easily create a Java API to represent my REST API.  The HTTP request is done in a background thread, and the result is returned as a callback. No need to open my own thread for the network operation, everything is done for me! Here’s an example for a simple service:


{% highlight java %}
public interface TaskService {
	@GET("tasks/{taskId}/friends")
	void getFriends(@Named("taskId") String taskId, Callback callback);
}
{% endhighlight %}




This is an example for getting all the friends in a shared task. I declare an interface for my API with the name of the service I’m providing, and for each API call I declare a method. For each method I set the HTTP request type (GET/PUT/POST/DELETE) and the REST API URL. The URL in this case is **tasks//friends**. The taskId is the first parameter in the method, and matching it with the parameter **{taskId}** in the URL is done using the “@Named” annotation. The Callback is called when the API call has returned, with the object I’m requesting. The object is constructed from a JSON string (other type handlers, as XMLs, can be added), and the parsing is done by the library all by itself, the developer doesn’t have to deal with JSON parsing at all, he sends an object, and gets an object!

This library was built for general Java use, and it’s using Google’s [Guice](http://code.google.com/p/google-guice/) to set objects in the code. In Android we can use [RoboGuice](http://code.google.com/p/roboguice/wiki/SimpleExample) to get this done. Now, to use our interface, I need to declare it as one of my data members on my class, and use the @Inject annotation, get it all “Guiced” up with some magic implementation (we’ll get to that later).


{% highlight java %}
@Inject
TaskService mTaskService;
{% endhighlight %}




Later on, I just call the method I want for any specific use


{% highlight java %}
mTaskService.getFriends("543", callback);
{% endhighlight %}




The callback is an interface I need to implement, telling what I need to do in case of a server error (HTTP response > 500) or client error ( > 400) or in case of a success, where I get the object I requested. A typical call will be:


{% highlight java %}
        mTaskService.getFriends("543", new Callback() {
            @Override
            public void call(Friends myFriends, int statusCode) {
				// YEY! Success! No I can do stuff with the list of my task friends
            }

            @Override
            public void sessionExpired() {
            }

            @Override
            public void networkError() {
            }

            @Override
            public void clientError(Friends myFriends, int statusCode) {
            }

            @Override
            public void serverError(String message, int statusCode) {
            }

            @Override
            public void unexpectedError(Throwable t) {
            }
        });
{% endhighlight %}




This is cool, since I don’t even have to compare HTTP status codes, I get a clean callback for each case.

HOWEVER,

If I want to call several API calls, one after another, from a background thread, it’ll **get messy**. For example, the tasks sync algorithm: getting all the categories, and all the tasks, merge with local changes and then update the server. We have at least 4 network calls. Since this entire routine runs in a background thread, there’s no use for the callback system, I don’t care if each call will block the thread, It’s not blocking the user’s UI. I also have to wait for each command to finish, before executing the next one. Let’s see how it looks like:


{% highlight java %}
        mTaskService.getCategories(..., new Callback() {
            @Override
            public void call(Categories, int statusCode) {

                 mTaskService.getTasks(..., new Callback() {
                       	@Override
                       	public void call(Categories, int statusCode) {

								mergeStuff();
								mTaskService.updateCategoiries(Categories, ....) {

								....

								}
               	}
            }
		...
        });
{% endhighlight %}




Pretty cumbersome, right?

That’s why I decided to solve this problem by allowing to run it, including the network call, in the current thread. It’s up to the developer to decide whether he wants his API call to run asynchronously, or to be blocking. His intention will be declared in his method declaration. As an example, let’s take the “_getFriends()_” method, and create a blocking version of it:


{% highlight java %}
public interface TasksService {
	@GET("tasks/{taskId}/friends")
	void getFriends(@Named("taskId") String taskId, Callback callback);

	@GET("tasks/{taskId}/friends")
	Friends getFriendsBlocking(@Named("taskId") String taskId);
}
{% endhighlight %}




The synchronous method returns the object we seek for, the Friends object, and does not receive any Callback. All that looks great, but how to make it work in the existing library?

![We need to go deeper](http://t.qkme.me/356wcq.jpg)

Before I could explain the modifications I did to the library, I need to explain how it works. Since Square did a pretty good job making the code well generic, I’ll simplify the code a little to make it easier to explain.

Remember the “magic implementation” for the API’s Java interface we created? Since we’re declaring an interface with no implementing class – how does it know what to do? For that, we’ll get a close encounter with one of Java’s magical creatures – **[Proxy](http://docs.oracle.com/javase/1.4.2/docs/api/java/lang/reflect/Proxy.html)**. The Proxy class is basically a runtime implementation for interfaces. When we declare a Proxy, we set the interface(s) we want to implement and an InvocationHandler class, to bring this interface into life. Let's see how it looks:


{% highlight java %}
TaskService  taskService = Proxy.newProxyInstance(TaskService.class.getClassLoader(),
								new Class[]{TaskService.class},
								handler);
{% endhighlight %}




We took our own TaskService interface from before, and created a Proxy for it using the _newProxyInstance_ static method. We pass the ClassLoader and the Interface class itself. The handler is the last and most important argument here. Let’s take a look at the handler’s implementation:


{% highlight java %}
  private class RestHandler implements InvocationHandler {
    @Override public Object invoke(Object proxy, final Method method, final Object[] args) {

      String url =

      try {
        // Build the request and headers.
        final HttpUriRequest request =

        // Get the last arg, will determine if we have a callback or not
        final Object lastArg = (args == null || args.length == 0)? null : args[args.length - 1];

        // Going the Async way
        if (lastArg instanceof Callback<?>) {
                backgroundInvoke(request, method, finalUrl, start, (Callback<?>)lastArg);
        // The last arg is not a Callback - we do the request on the current thread
        } else {
            return foregroundInvoke(request, method, finalUrl, finalStartTime);
        }
      } catch (Throwable t) {
        < Trigger some alarm or something >
      }

      // Methods should return void.
      return null;
    }
{% endhighlight %}




The InvocationHandler is implementing a single method – _invoke_(). It provides us the method we called on the interface (e.g. “getFriends”) and the arguments we passed to that method (e.g. “taskId” and the Callback object). We build the Http URI request using that data:



	
  * method – We access its annotations to get the HTTP method (GET/POST…) and the REST URI (e.g. “tasks/{taskId}/friends”)

	
  * args – Containing the parameters/objects to send with the request, as taskId itself.


The HTTP URI Request builder’s job is making this:


{% highlight java %}
mTaskService.getFriends("4354", callback);
{% endhighlight %}




into an appropriate HTTP request as “**http:///tasks/4354/friends**”.

Hooray!

{% youtube fYi-99klXAs %}

(That’s what [Festivus](http://en.wikipedia.org/wiki/Festivus) is all about…)

The next few lines is where I added my code, to support running the request on the foreground. I check if the last argument is a Callback object. if not – I run the request on the foreground. Simple as that. Notice that I return the value I get from _foregroundInvoke()_, because that’ll be the object I’ll return for calling this method, in this case – a “Friends” object. Other methods are using callbacks, that’s why they return nothing.

All of this looks so nice, but what about error handling?

We want to believe the world is safe – peace in the middle east and HTTP status code in the 200 area. Unfortunately, things are more complex. Errors will always be around and we need to take care of them. What should we do in case the taskId wasn’t found? The server will return 404, but since we have no Callback object, we need to give some feedback to the program using this adapter. In this case – we’ll throw an Exception.

I built a ResponseNotOkException, to be thrown when we get a status code not in the 200-299 zone. Let’s review the foregroundInvoke():


{% highlight java %}
    private Object foregroundInvoke(HttpUriRequest request, Method method, String url, String startTime) throws Throwable {
        HttpResponse response = httpClientProvider.get().execute(request);

        int statusCode = response.getStatusLine().getStatusCode();
        HttpEntity entity = response.getEntity();

        // If we got an OK response and non-empty entity - let parse it!
        if (statusCode >= 200 && statusCode < 300) {
            if (entity != null) {
                String bodyString = EntityUtils.toString(entity, "UTF-8");

                // If the return type is not void,
                // parse the body with the Gson parser, The return type of the method is the object here
                if (!method.getReturnType().equals(void.class)) {
                     return gson.fromJson(bodyString, method.getReturnType());
                }
            }
        } else {
            ResponseNotOKException exception = new ResponseNotOKException(statusCode);

            if (entity != null) {
                 String dataString = EntityUtils.toString(entity);
                 exception = new ResponseNotOKException(statusCode, dataString);
            }

            throw exception;
        }

        return null;
    }
{% endhighlight %}




In this method, we parse the response from the server, which is in a JSON format, and create an object from its JSON representation. Notice the object type is taken from the Method’s return type attribute, that’s another use in the Method object we got in the **invoke**() method presented earlier. In case the status code is not in the “OK range”, we create a ResponseNotOkException, and optionally fill it with data from the server (could help the client get more info on the problem). To get our code compatible with Exceptions being thrown at him, we need to declare it in our interface’s method. The Final interface will look like:


{% highlight java %}
public interface TasksService {
	@GET("tasks/{taskId}/friends")
	void getFriends(@Named("taskId") String taskId, Callback callback);

	@GET("tasks/{taskId}/friends")
	Friends getFriendsBlocking(@Named("taskId") String taskId) throws ResponseNotOkException;
}
{% endhighlight %}




A usage example will be


{% highlight java %}
Friends myTaskFriends = mTaskService.getFriendsBlocking("23232");
for (Friend currFriend : myTaskFriends) {
	currFriend.email("When will you finish this task?");
}
{% endhighlight %}




Phew.. That was quite a journey.
Now we know more about how to use Proxies and manipulate them to solve our problems. The code I presented here was reduced and simplified. To see the full source code, you can access it here:

[https://github.com/Udinic/retrofit](https://github.com/Udinic/retrofit)

Hopefully it’ll get approved by square, and added to the main repository.

If you want to learn more about the Retrofit library from Square, you can check out this talk by 2 of their top Java/Android developers [here](http://www.infoq.com/presentations/Android-Squared).
