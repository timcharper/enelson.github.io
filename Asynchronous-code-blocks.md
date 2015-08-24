# Async Code Blocks

## Starting Point

I needed a way to make my controller endpoints asynchronous, which allows greater throughput of requests to the server. The Jersey way, shown below, requires some boilerplate code to put the async code into a background thread. I started to see two patterns that emerged in writing a few of the async methods:

1. There was a lot of repeated boilerplate code around wrapping your code in background threads
2. This was a perfect example of behavior parameterization, where after the boilerplate threading code, the code that goes in after is user-defined behavior. 

This is what the code was looking like initially:

```java
@Path("/")
public class MyController {
    @Path("/one")
    @GET
    public void asyncGetData(@Suspended AsyncResponse asyncResponse) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                String result = veryExpensiveOperation();
                asyncResponse.resume(result);
            }
         }).start();
    }

    @Path("/two")
    @GET
    public void asyncGetDataAgain(@Suspended AsyncResponse asyncResponse) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                String result = anotherVeryExpensiveOperation();
                asyncResponse.resume(result);
            }
         }).start();
    }
}
```

This was a prime candidate for parameterization of behavior. I wanted to get rid of the repetative Thread/Runnable code, and allow the user to pass in their controller behavior to a common method. So I created an abstract base class that all controllers could extend from, and a shared method that could wrap code into an asycn block:

```java
public abstract class BaseController {

    protected  <T> void async(AsyncResponse asyncResponse, T data, BiConsumer<AsyncResponse, T> f) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                f.accept(asyncResponse, data);
            }
         }).start();
    }

}
```
