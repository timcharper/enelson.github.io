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
    @POST
    public void asyncGetDataAgain(MyRequest request, @Suspended AsyncResponse asyncResponse) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                String result = anotherVeryExpensiveOperation(request);
                asyncResponse.resume(result);
            }
         }).start();
    }
}
```

This was a prime candidate for parameterization of behavior. I wanted to get rid of the repetative Thread/Runnable code, and allow the user to pass in their controller behavior to a common method. So I created an abstract base class that all controllers could extend from, and a shared method that could wrap code into an asycn block:

```java
public abstract class BaseController {
    protected void async(AsyncResponse asyncResponse, Consumer<AsyncResponse> f) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                f.accept(asyncResponse);
            }
        }).start();
    }

    protected <T> void async(AsyncResponse asyncResponse, T data, BiConsumer<AsyncResponse, T> f) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                f.accept(asyncResponse, data);
            }
         }).start();
    }
}
```

This worked out great. Now I could simplify my controller code to something like this:

```java
@Path("/")
public class MyController {
    @Path("/one")
    @GET
    public void asyncGetData(@Suspended AsyncResponse asyncResponse) {
        async(asyncResponse, (asyncResp) -> {
            String result = veryExpensiveOperation();
            asyncResp.resume(result);
        });
    }

    @Path("/two")
    @POST
    public void asyncGetDataAgain(MyRequest request, @Suspended AsyncResponse asyncResponse) {
        async(asyncResponse, request, (asyncResp, request) -> {
            String result = anotherVeryExpensiveOperation(request);
            asyncResponse.resume(result);
        });
    }
}
```

No more boilerplate and ugly repetative threading code. That code was abstracted away into our `BaseController`, and we could simply wrap our controller login in a nice `async()` block. We made the `async()` method generic by using behavior parameterization using the functional interfaces of `Consumer<T>` and `BiConsumer<T,U>`. 

One more efficiency adjustment was made in the `BaseController` by replacing the raw Thread/Runnable code with an ExecutorService. Executors are much more efficient and managing and scheduling threads, and they help remove the complexity of doing this manually, which manual management is not a good idea in production systems. The change made our `BaseController` even cleaner to read:

```java
public abstract class BaseController {

    private ExecutorService executor = Executors.newFixedThreadPool(10);

    protected void async(AsyncResponse asyncResponse, Consumer<AsyncResponse> f) {
        executor.submit(() -> f.accept(asyncResponse));
    }

    protected <T> void async(AsyncResponse asyncResponse, T data, BiConsumer<AsyncResponse, T> f) {
        executor.submit(() -> f.accept(asyncResponse, data));
    }

}
```
