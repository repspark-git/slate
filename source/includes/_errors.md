# Errors

RepSpark's API uses the following error codes:

Error Code | RepSpark Error | Reason
---------- | -------------- | ------
400 Bad Request | MissingRequiredHeader | {0} header was not specified.
400 Bad Request | UnsupportedHeader | HTTP header {0} specified in the request is not supported.
400 Bad Request | UnsupportedQueryParameter | Query parameter {0} specified in the request URI is not supported.
400 Bad Request | InvalidQueryParameter | Query parameter {0} specified in the request URI is not valid.
400 Bad Request | InvalidHttpVerb | The HTTP verb specified was not recognized by the server.
400 Bad Request | InvalidInput | Request input {0} is not valid.
400 Bad Request | InvalidHeaderValue | The value provided for HTTP header {0} was not valid. Verify the value including the format.
400 Bad Request | UnsupportedContentType | 	The specified content type is not supported.
401 Forbidden | MissingAuthenticationInfo | The authentication information was not provided. Verify the value of Authorization header.
401 Forbidden | InvalidAuthenticationInfo | The authentication information was not provided in the correct format. Verify the value of Authorization header.
401 Forbidden | AuthenticationFailed | Server failed to authenticate the request. Make sure the value of the Authorization header is formed correctly.
404 Not Found | ResourceNotFound | The specified resource does not exist.
404 Not Found | SchemaNotFound | The specified schema does not exist.
404 Not Found | ProcessorNotFound | The specified processor does not exist.
404 Not Found | TransactionNotFound | The specified transaction does not exist.
405 Method Not Allowed | UnsupportedHttpVerb | The resource doesn't support the specified HTTP verb.
409 Conflict | TransactionAlreadyExists | Transaction with the same token already exists.
500 Internal Server Error | InternalError | The server encountered an internal error. Please retry the request.
500 Internal Server Error | OperationTimedOut | The operation could not be completed within the permitted time.
500 Internal Server Error | UnexpectedErrorOnPreProcessing | Unexpected error occurred on pre-processing.
500 Internal Server Error | UnexpectedErrorOnProcessing | Unexpected error occurred on processing.
500 Internal Server Error | UnexpectedErrorOnPostProcessing | Unexpected error occurred on post-processing.
503 Service Unavailable | ServerBusy | 	The server is currently unable to receive requests. Please retry your request.
