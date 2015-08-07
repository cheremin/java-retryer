Small library for retrying tasks which can potentially fail by unknown reasons. Includes kinds of back off strategies

Example (quering URL with retry):
```
public static String queryRetryable( final String urlQuery ) throws Exception {
    return new Retryer().doRetryable(
            new RetryableTaskHelper<String, Exception>() {
                @Override
                public String execute( final int tryNo ) throws Exception {
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

More examples in [Tutorial](Tutorial.md)