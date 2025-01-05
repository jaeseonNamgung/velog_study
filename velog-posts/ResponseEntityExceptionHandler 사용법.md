<h1 id="responseentityexceptionhandler-란">ResponseEntityExceptionHandler 란</h1>
<p>ResponseEntityExceptionHandler는 Spring MVC에서 제공하는 예외처리 클래스로, Spring MVC 애플리케이션에서 발생하는 표준적인 예외들을 효율적으로 처리할 수 있도록 도와준다.</p>
<p>ResponseEntityExceptionHandler는 아래 사진과 같이 Spring MVC에서 발생하는 예외를 미리 @ExceptionHandler로 정의해 놓았다. ResponseEntityExceptionHandler로 상속만 받으면 이러한 예외를 직접 구현할 필요가 없어진다.</p>
<pre><code class="language-java"> @ExceptionHandler({
HttpRequestMethodNotSupportedException.class, 
HttpMediaTypeNotSupportedException.class, 
HttpMediaTypeNotAcceptableException.class, 
MissingPathVariableException.class, 
MissingServletRequestParameterException.class, 
MissingServletRequestPartException.class, 
ServletRequestBindingException.class, 
MethodArgumentNotValidException.class, 
HandlerMethodValidationException.class, 
NoHandlerFoundException.class, 
NoResourceFoundException.class, 
AsyncRequestTimeoutException.class, 
ErrorResponseException.class, 
MaxUploadSizeExceededException.class, 
ConversionNotSupportedException.class, 
TypeMismatchException.class, 
HttpMessageNotReadableException.class, 
HttpMessageNotWritableException.class, 
MethodValidationException.class, BindException.class
})</code></pre>
<p>하지만 ResponseEntityExceptionHandler는 body 값을 null로 처리하는 단점이 있다.</p>
<pre><code class="language-java">@Nullable
    protected ResponseEntity&lt;Object&gt; handleHttpRequestMethodNotSupported(HttpRequestMethodNotSupportedException ex, HttpHeaders headers, HttpStatusCode status, WebRequest request) {
        pageNotFoundLogger.warn(ex.getMessage());
        return this.handleExceptionInternal(ex, (Object)null, headers, status, request);
    }

    @Nullable
    protected ResponseEntity&lt;Object&gt; handleHttpMediaTypeNotSupported(HttpMediaTypeNotSupportedException ex, HttpHeaders headers, HttpStatusCode status, WebRequest request) {
        return this.handleExceptionInternal(ex, (Object)null, headers, status, request);
    }</code></pre>
<p>그렇기 때문에 handleExceptionInternal를 재정의 해서 body에 값을 넣어 주어야 한다.</p>
<pre><code class="language-java">    // Validation 에러 재정의 
    @Override
    protected ResponseEntity&lt;Object&gt; handleMethodArgumentNotValid(MethodArgumentNotValidException ex, HttpHeaders headers, HttpStatusCode status, WebRequest request) {
        return handleExceptionInternal(ex, ValidationResponse.of(status.value(), ex.getBindingResult()),headers,status, request);
    }

    @ExceptionHandler
    public ResponseEntity&lt;Object&gt; customException(CustomException customException, WebRequest webRequest){
        return handleExceptionInternal(customException, customException.getErrorCode(), webRequest);
    }

    @ExceptionHandler
    public ResponseEntity&lt;Object&gt; exception(Exception ex, WebRequest webRequest){
        return handleExceptionInternal(ex, BaseErrorCode.INTERNAL_SERVER_ERROR, webRequest);

    }

    private ResponseEntity&lt;Object&gt; handleExceptionInternal(Exception  ex, ErrorCode errorCode, WebRequest webRequest) {
        return handleExceptionInternal(ex, errorCode, HttpHeaders.EMPTY,errorCode.getHttpStatus(), webRequest);
    }

    // Validation Exception 처리를 위한 handleExceptionInternal 재정의
    protected ResponseEntity&lt;Object&gt; handleExceptionInternal(Exception  ex, ValidationResponse validationResponse,HttpHeaders headers, HttpStatusCode status, WebRequest webRequest) {
        return super.handleExceptionInternal(ex, validationResponse, headers, status, webRequest);
    }

    // handleExceptionInternal 메서드 재정의
    protected ResponseEntity&lt;Object&gt; handleExceptionInternal(Exception ex, ErrorCode errorCode, HttpHeaders headers, HttpStatusCode statusCode, WebRequest request) {
        return super.handleExceptionInternal(
                ex,
                // 커스텀 Response를 만들어 body에 값을 넣는다.
                ApiExceptionResponse.of(errorCode),
                headers,
                statusCode,
                request);
    }
</code></pre>