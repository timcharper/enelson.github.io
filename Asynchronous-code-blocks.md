# Async Code Blocks

## Code

```java
@Path("/")
public class MyController {
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
}
```