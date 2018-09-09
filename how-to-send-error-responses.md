# Best Practice to send Error Response to an API Request

## Response Structure

```json
{
    "error": {
        "code": <int>,
        "type": <string>,
        "userMessage": <string>,
        "message": <string>,
        "description": <string>
    }
}
```

### Description

1. __Code:__ Integer _[Mandatory]_
    + This code should be treated as a contract of the API i.e. don't do breaking changes
    + Error codes which are specific to _our_ documentation. These may or may not be same as HTTP Status Codes.
    + Our documentation should be directly telling the reason of such error codes
    + Can start with just 400 for all codes
    + New codes should be fixed
    + Keeping it int type as easier to do validation and react
    + 1 code should cover 1 exact issue
2. __Type:__ String _[Mandatory]_
    + This code should be treated as a contract of the API i.e. don't do breaking changes
    + Like code but string/enum
    + Can write category of the error i.e. AuthError, ClientError, ValidationError
    + Can also be specific like PANELIST_NOT_FOUND or USER_NOT_FOUND or NOT_FOUND
    + Different types of error can be under a single type
    + Have already been using these
3. __UserMessage:__ String _[Optional]_
    + Not part of contract and can change without change of API version
    + In most cases can be empty or null but, if server wants to display that error directly to the front-end without requiring any change on front-end, this field should be used
    + Example: "We have got the required feedback. Please come later for another test."
4. __Message:__ String _[Optional]_
    + Not part of contract
    + Keeping it for backward compatibility. Should be removed if no one is using it.
    + This can be used as a message to developer and further information can be put in the description
5. __Description:__ String _[Optional]_
    + Put more details about the error here
    + I know mostly, we won't be putting anything here :P
    + Unless to debug, etc. which is though not right way as we may pass sensitive data


## Notes:
1. Use only few HTTP Status codes. Take a look at Twitter error codes
2. Ideally document in Swagger that what all error codes can be returned by this API
3. Can bring HTTP status code as part of the response but, don't see any need as of now.
4. To make sure documentation and code remains consistent, there is docAPI or something which generates Spring API documentation from the tests
5. What you should understand is REST and HTTP are different design/protocol
6. For multiple validation errors, there should be a child element of "error" like "fieldErrors" which is a list of validation errors.


## Implementation
1. Code readability i.e. when throwing exception, if just putting an integer, it will be confusing and have to jump to see what exactly are we doing
2. Same uber error structure. There can be other specific types which have few fields fixed
3. Same things goes for exception
4. Avoid dynamic code as readability and debugging the error code seen in productions logs become difficult
5. Each feature-error-codes get a block of unique codes
6. Unit tests to ensure that no two errors have the same code
7. For documentation, put the range of error codes assigned to feature in javadoc

```java
public enum FeatureErrorCodes extends ErrorCode {
    ...,
    FEATURE_ERROR_NAME(1003, "User facing error message"),
    ...
}
```

```java
public abstract class BaseException extends RunTimeException {
    private ErrorCode code;
    private String description;

    public BaseException(ErrorCode code) {
        super(code.getMessage());
        this.code = code;
        this.description = "";
    }

    public BaseException(ErrorCode code, String format, Object... args) {
        super(code.getMessage());
        this.code = code;
        this.description = String.format(format, args);
    }

    ...
}
```

## Exception
1. Base: PulseLabsException which is abstract to give a structure i.e. required info
2. Make concrete classes of above exception which have specific use cases
3. Don't use inbuild exceptions like IllegalArgument, State, etc. instead create a new class by extending these exceptions and use those. This will provide you flexibility as you can use Error Code, etc. directly. This will also help in tracking the _explicit_ exceptions that we throw and handle. Though, you still have to handle the standard java exeptions.
4. Custom Exceptions should not be defined for each feature. Using it for better code readability instead of just a single custom exception.
5. Use enums for ErrorCodes so, that through data-type, it can be enforces instead of using integer/string which are generic
6. Using Aspect Oriented Programming technique i.e. ExceptionTranslator to translate thrown exception to ErrorCode and Response structure.
7. One of the constructor of exception should be like `String.format`


## References
1. https://nordicapis.com/best-practices-api-error-handling/
2. https://alidg.me/blog/2016/9/24/rest-api-error-handling
3. Take a look at good engineering companies API documentation
4. Other links I forgot to note down

__TODO:__ Think about web-socket error contract.