# Logging Guidelines, Draft

These Logging Guidelines can be used as a draft for developing your own logging guidelines. They should be seen as a reference and should be adopted to your own project, your own team and your own needs.

The "Log levels" chapter ensures a common understanding of the different log levels. "Guidelines" try to define a set of rules for helping the developer in creating meaningful log messages at the right place and in the right severity. Since these "Guidelines" are not totally strict and complete, there is also a "Best Practices" chapter for reoccurring situations within your specific project.
Especially the last chapter is a constantly updated and always refined chapter during the lifetime of a project.


## Log levels
- ERROR: for something fatal / not recoverable, user will have a bad experience, developer or admin will have to restart / cleanup / ... the system
- WARN: for something unexpected, user will have bad experience but system will recover
- INFO: for system wide updates, logged by the component which is responsible for the information (in order to not log the same information twice)
- DEBUG: information to retrace the steps toward an exception / error (state changes within a component)
- TRACE: any (verbose) developer information

## Guidelines
- Log should always be 1-liners (except in TRACE)
- Developer log messages with unreadable, unformatted output should go into log level TRACE
- Do not print stack traces (except in TRACE)
- Stack traces of (and only of) unhandled / totally unexpected / system crashing / .. exceptions can be logged in DEBUG (but better implement a correct failure recovery!)
- Format for log messages: `Statement: param1=value1, param2=value2`
  - Good:
  ```
  log.info("Processing booking request: request id=42, user id=43")
  ```
  - Bad:
  ```
  log.info("Processing user 43 with request id 42")
  ```
  - Start the `Statement` with a capital letter
  - The `Statement` is a brief, precise and unique description, i.e.
    - it should be easy to `grep` for the line in the codebase and
    - the statement should be, to some degree, understandable for a non-tech-guy
  - `param1` should be the most important parameter

- Log level DEBUG is used in most cases, i.e. DEBUG should be used for internal changes of a component
- Log level INFO is used to describe the overall system behavior, i.e. INFO is used for all messages being received by or sent by a component:
  - all events which are handled by a component should be logged before being processed
  - all events emitted by a component should be logged before being sent

- Log level DEBUG, INFO, WARN and ERROR should contain only relevant information for
  - deriving the current state of the application and
  - deriving the path of execution.
  - No more, and no less.

- Note: Log messages are also a way to describe your code, i.e. methods, method calls (next to using comments)
- Log what will happen next, not what has already happened; for INFO and DEBUG by using the -ing Form
  - Good:
  ```
  log.info("Processing booking request")
  ```
  - Bad:
  ```
  log.info("Booking request received")
  ```

- Do not log success (except in TRACE); in general it is assumed that the request was processed correctly if no WARN or ERROR is logged
  - Ok (note that `log.trace` is used):
  ```
  log.trace("Booking request processed")
  ```
  - Bad:
  ```
  log.debug("booking request processed")
  ```
- if it is possible to log the trace up to an exception only if an `Exception` was thrown, do so! (i.e. you see the DEUBG and INFO messages of an processed request only if there is also an ERROR message in the trace)

- Do not log exception when they are thrown, log them when they are handled
- Provide enough information in the error message of a thrown exception to retrace the error
- Do not log errors by third-party libraries or external services which are handled properly by the system
- Start with "Cannot", "Failed to", .. for WARN and ERROR
  - Good:
  ```
  log.error("Failed to process booking request")
  ```
  - Bad:
  ```
  log.error("Exception: booking request failed")
  ```

- Developer logging for finding a hard error should be logged to DEBUG or TRACE and it should be possible to turn these on or off by configuration

## Best Practices
- Parameters can be logged with String Interpolation (note: only if the String parameter of error(..) is lazily evaluated)
```
log.error(s"Processing booking request: request id=$requestId, user id=$userId")
```
- For exceptions
  - log the exception message with level ERROR and then
  - the exception itself with level TRACE
  - Good:
  ```
  log.error("Failed to process request: message=${exception.getMessage}")
  log.trace("Failed to process request: trace=", exception)
  ```
  - Bad:
  ```
  log.error("Failed to process request: message=${exception.getMessage}", exception)
  ```

- How to do state logging in controllers of server:
  - Controller startup with INFO (i.e. setup was successful)
  ```
  log.info("Starting booking controller")
  ```
  - Incoming REST requests with DEBUG (add parameters after a ":" if available; log message should be after payload was parsed since a parsing error is logged in the error handling mechanism)
  - PUT
  ```
  log.debug("Setting booking end date: id=$bookingId, end date=$timestamp")
  ```
  - POST
  ```
  log.debug("Creating booking")
  ```
  - DELETE
  ```
  log.debug("Deleting booking")
  ```
  - GET (Getting?)
  ```
  log.debug("Getting booking")
  ```

- How to do state logging in event publisher/subscriber (e.g. towards/from Kafka)
  - Publishing an event with DEBUG
  ```
  log.debug("Publishing new booking: booking id=$bookingId, user id=$userId, channel=$channel")
  ```
  - Handling an incoming event with DEBUG
  ```
  log.debug("Processing booking stopped: id=$bookingId, timestamp=$timestamp")
  ```

- Updating a collection, i.e. in repositories (e.g. `BookingRepository`) with DEBUG
  - Inserting a document with DEBUG
  ```
  log.debug("Inserting booking: id=$bookingId, collection=$bookingCollection")
  ```
  - Updating a document with DEBUG
  ```
  log.debug("Updating booking status: status=$status, id=$bookingId, collection=$bookingCollection")
  ```
  - Removing a document with DEBUG
  ```
  log.debug("Removing booking: id=$bookingId, collection=$bookingCollection")
  ```

- Logging stack traces only in debug level with `ExceptionUtils` from logback library; reuse same log message text
```
log.info(s"Could not map google response, retrying: status code=$status, response=$body, message=${e.getMessage}")
if (log.isDebugEnabled)
 log.debug(s"Could not map google response, retrying: stack trace=${ExceptionUtils.getStackTrace(e)}")
```

- [new best practices]


## References
- Android Log Sparingly http://source.android.com/source/code-style.html#log-sparingly
- The Art of Logging http://www.codeproject.com/Articles/42354/The-Art-of-Logging

