

# Introduction #

Retryer is simple library for using in case when you need to run some task which can potentially fails by reasons out of your control -- so if it happens you want to make several attempts before surrender.

# Components #

[IRetryableTask](http://code.google.com/p/java-retryer/source/browse/trunk/code/java/src/main/java/org/retryer/IRetryableTask.java) -- it is the interface your (potentially failing) task must implement. It has two methods.
First is `execute(int tryNo)` -- it is the place for your (retryeable) business logic. You can use tryNo (0-based) argument to choose different servers to query from, for example.
Second is `isFatalReason(int tryNo, Throwable reason)` -- this method will be called by library (IRetryer) after each failed run of execute(). It gives you the ability to detect "undoubtly fatal" errors and stop retryes immediately. You should return false from this method if given Throwable reason is not fatal, so continue retry may have sense, or true if continue after such error make no sense at all.

[IRetryStrategy](http://code.google.com/p/java-retryer/source/browse/trunk/code/java/src/main/java/org/retryer/IRetryStrategy.java) -- it is interface for making decision when to retry and how. Most of the time you should use existing implementations from backoffs package

[IRetryer](http://code.google.com/p/java-retryer/source/browse/trunk/code/java/src/main/java/org/retryer/IRetryer.java) (the only implementation currently is [Retryer](http://code.google.com/p/java-retryer/source/browse/trunk/code/java/src/main/java/org/retryer/impl/Retryer.java)) is the main component, which do the job

# Usage #

Lets start from simple code what google for Retryer with standard URL api and `CharStreams` class from google guava:
```
private static final String URL_TO_QUERY = "http://google.com/?q=Retryer";

public static String simpleQuery( final String urlString ) throws Exception {
    final URL url = new URL( urlString );
    final InputStream is = url.openStream();
    try {
        final InputStreamReader r = new InputStreamReader( is, "ISO-8859-1" );
        try {
            return CharStreams.toString( r );
        } finally {
            r.close();
        }
    } finally {
        is.close();
    }
}
...

final String result = simpleQuery(URL_TO_QUERY);
```
([QueryUrlOnce.java](http://code.google.com/p/java-retryer/source/browse/trunk/code/java/src/main/java/org/retryer/tutorial/QueryUrlOnce.java))

## Retry now ##
What is OK, but ~~what if google will be unavailable~~ ok, ok, what if your network will be temporary overloaded, and connection can't be established?
```
public static String queryRetryableNow( final String urlQuery ) throws Exception {
    return new Retryer().doRetryable(
            new IRetryableTask<String, Exception>() {
                @Override
                public String execute( final int tryNo ) throws Exception {
                    return QueryUrlOnce.simpleQuery( urlQuery );
                }

                @Override
                public boolean isFatalReason( final int tryNo, final Throwable reason ) {
                    return false;//no fatal exceptions for now
                }
            },
            RetryNow.INSTANCE
    );
}
```
([QueryUrlRetryable.java](http://code.google.com/p/java-retryer/source/browse/trunk/code/java/src/main/java/org/retryer/tutorial/QueryUrlRetryable.java)) here we will retry immediately after each fail, until the end of the universary.

## Retry with fixed delay ##
Ok, but lets make a pause between retryes -- 1 second:

```
public static String queryRetryableDelayed( final String urlQuery ) throws Exception {
    return new Retryer().doRetryable(
            new IRetryableTask<String, Exception>() {
                @Override
                public Object execute( final int tryNo ) throws Exception {
                    return QueryUrlOnce.simpleQuery( urlQuery );
                }
                @Override
                public boolean isFatalReason( final int tryNo,
                                       final Throwable reason ) {
                    return false;//still no fatal errors
                }
            },
            new RetryDelayed( TimeUnit.SECONDS.toMillis( 1 ) )
    );
}
```
([QueryUrlRetryableDelayed.java](http://code.google.com/p/java-retryer/source/browse/trunk/code/java/src/main/java/org/retryer/tutorial/QueryUrlRetryableDelayed.java))

## Configure more realistic retry strategy ##

OK, lets make complex backoff strategy: "try to run given code, no more then 5 times, with exponentially increasing delay between tryes, starting with 1 second delay, but do not delay more then 10 seconds"

```
public static String queryRetryableComplex( final String urlQuery ) throws Exception {
    return new Retryer().doRetryable(
            new RetryableTaskHelper<String, Exception>() {
                @Override
                public String execute( final int tryNo ) throws Exception {
                    return QueryUrlOnce.simpleQuery( urlQuery );
                }
            },
            Backoff
                        .withExponentialGrowingDelay()
                        .startingWithDelay( 1, TimeUnit.SECONDS )
                        .maxTryes( 5 )
                        .maxDelay( 10, TimeUnit.SECONDS )
                        .build()
    );
}
```
(note usage of `RetryableTaskHelper` -- abstract helper class, which implemented `IRetryableTask` and gives you the default implementation for `isFatalReason(int tryNo, Throwable reason)` method -> return false)
([QueryUrlRetryableComplex.java](http://code.google.com/p/java-retryer/source/browse/trunk/code/java/src/main/java/org/retryer/tutorial/QueryUrlRetryableComplex.java))