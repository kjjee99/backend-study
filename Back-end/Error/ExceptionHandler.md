
# 오류처리



### 목차
- [Spring 오류처리 방식](#Spring_오류처리_방식)
- [예외 처리 방법](#예외_처리_방법)

---

## Spring 오류처리 방식
> Spring은 에러 처리를 위한 `BasicErrorController`가 있음<br>
> Spring Boot는 예외가 발생하면 기본적으로 `/error`로 에러 요청을 다시 전달하도록 WAS 설정<br>
> `WebMvcAutoConfiguration`을 통해 자동 설정이 되는 WAS 설정

- 일반적인 요청 흐름
```plain
WAS(Tomcat) -> Filter -> Servlet(Dispatcher Servlet) -> Interceptor -> Controller
```
- 컨트롤러 하위에서 예외가 발생했을 때, 별도의 예외 처리를 하지 않으면 WAS까지 에러 전달
```plain
Controller(Error) -> Interceptor -> Servlet(Dispatcher Servlet) -> Filter -> WAS(Tomcat) 
```
- 흐름 정리
```plain
WAS(Tomcat) -> Filter -> Servlet(Dispatcher Servlet) -> Interceptor -> Controller
-> Controller(Error) -> Interceptor -> Servlet(Dispatcher Servlet) -> Filter -> WAS(Tomcat) 
-> WAS(Tomcat) -> Filter -> Servlet(Dispatcher Servlet) -> Interceptor -> Controller(BasicErrorController)
```

=> Error Controller를 한 번 더 호출하는 로직

Servlet은 `dispatcherType`으로 요청의 종류 구분
- REQUEST : 요청
- ERROR : 에러
- 필터 등록 시(FilterRegistrationBean)에 호출될 dispatcherType 타입을 설정할 수 있고, 별도의 설정이 없다면 *REQUEST*일 경우에만 필터 호출

### BasicErrorController
> accept Header에 따라 에러 페이지를 반환하거나 에러 메세지를 반환<br>
> 기본적으로 `/error`에 정의<br>
> properties에서 `server.error.path`로 변경가능

```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {

    private final ErrorProperties errorProperties;
    ...

    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
        ...
    }

    @RequestMapping
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        ...
        return new ResponseEntity<>(body, status);
    }
    
    ...
}
```

#### 반환할 에러 속성
> errorHtml() & error()는 모두 *getErrorAttributeOptions*를 호출해 반환할 에러 속성을 얻음<br>
> DefaultErrorAttributes로부터 반환할 정보를 가져옴 

- timestamp: 에러가 발생한 시간
- status: 에러의 Http 상태
- error: 에러 코드
- path: 에러가 발생한 uri
- exception: 최상위 예외 클래스의 이름(설정 필요)
- message: 에러에 대한 내용(설정 필요)
- errors: BindingExecption에 의해 생긴 에러 목록(설정 필요)
- trace: 에러 스택 트레이스(설정 필요)

```plain
{
    "timestamp": "2021-12-31T03:35:44.675+00:00",
    "status": 500,
    "error": "Internal Server Error",
    "path": "/product/5000"
}
```
- 에러 응답 조정
```yaml
# 에러 메시지 전송 여부
server.error.include-message: always
server.error.include-binding-errors: always
server.error.include-stacktrace: always
server.error.include-exception: false
```
```plain
### 결과 ###
{
    "timestamp": "2021-12-31T03:35:44.675+00:00",
    "status": 500,
    "error": "Internal Server Error",
    "trace": "java.util.NoSuchElementException: No value present ...",
    "message": "No value present",
    "path": "/product/5000"
}
```

## 예외 처리 방법

`cross-cutting concerns(공통 관심사)`를 메인 로직으로부터 분리하는 다양한 예외 처리 방식 고안<br>
-> 예외 처리 전략을 추상화한 HandlerExceptionResolver 인터페이스 생성(전략 패턴)

- 대부분의 HandlerExceptionResolver는 발생한 Exception을 catch하고 HTTP 상태나 응답 메세지 등을 설정
- WAS 입장에서는 해당 요청이 정상적인 응답인 것으로 인식
```java
public interface HandlerExceptionResolver {
    ModelAndView resolveException(HttpServletRequest request, 
            HttpServletResponse response, Object handler, Exception ex);
}
```

#### HandlerExceptionResolver 구현체
- `DefaultErrorAttributes` : 에러 속성을 저장하며 직접 예외를 처리하지 않음
  - DefaultErrorAttributes를 제외하고 직접 예외를 처리하는 3가지 ExceptionResovler들을 HandlerExceptionResolverComposite로 모아서 관리
  - Composite 패턴을 적용해 실제 예외 처리기들을 따로 관리
- `ExceptionHandlerExceptionResolver` : 에러 응답을 위한 Controller나 ControllerAdvice에 있는 ExceptionHandler를 처리
- `ResponseStatusExceptionResolver` : HTTP 상태 코드를 지정하는 @ResponseStatus 또는 ResponseStatusException를 처리
- `DefaultHandlerExceptionResolver` : 스프링 내부의 기본 예외 처리

### 에러 처리 동작 방식

#### 1. @ResponseStatus
> Error HTTP 상태를 변경하도록 도와주는 Annotation

- 적용할 수 있는 경우
  - Exception 클래스 자체
  - 메소드에 @ExceptionHandler와 함께
  - 클래스에 @RestcontrollerAdvice와 함께

```java
@ResponseStatus(value = HttpStatus.NOT_FOUND)
public class NoSuchElementFoundException extends RuntimeException {
  ...
}
```
```plain
### 결과 ###
{
    "timestamp": "2021-12-31T03:35:44.675+00:00",
    "status": 404,
    "error": "Not Found",
    "path": "/product/5000"
}
```

- ResponseStatusExceptionResolver는 @ResponseStatus를 처리하며, WAS까지 예외를 전달시키면서 복잡한 WAS의 에러 요청 전달 진행
    - 한계점
    - 에러 응답의 내용(Payload)를 수정할 수 없음(DefaultErrorAttributes를 수정하면 가능하긴 함)
    - 예와 클래스와 강하게 결합되어 같은 예외는 같은 상태와 에러 메세지를 반환함
    - 별도의 응답 상태가 필요하다면 예외 클래스를 추가해야 됨
    - WAS까지 예외가 전달되고, WAS의 에러 요청 전달이 진행됨
    - 외부에서 정의한 Exception 클래스에는 @ResponseStatus를 붙여줄 수 없음

#### 2. ResponseStatusException
> HttpStatus와 함께 선택적으로 reason과 cause를 추가할 수 있음<br>
> Uncheck 예외를 상속받고 있어 명시적으로 에러 처리하지 않아도 됨

```java
@GetMapping("/product/{id}")
public ResponseEntity<Product> getProduct(@PathVariable String id) {
    try {
        return ResponseEntity.ok(productService.getProduct(id));
    } catch (NoSuchElementFoundException e) {
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Item Not Found");
    }
}
```

- 장점
  - 기본적인 예외 처리를 빠르게 적용할 수 있으므로 손쉽게 프로토타이핑할 수 있음
  - HttpStatus를 직접 설정하여 예외 클래스와의 결합도를 낮출 수 있음
  - 불필요하게 많은 별도의 예외 클래스를 만들지 않아도 됨
  - 프로그래밍 방식으로 예외를 직접 생성하므로 예외를 더욱 잘 제어할 수 있음
- 단점
  - 직접 예외 처리를 프로그래밍하므로 일관된 예외 처리가 어려움
  - 예외 처리 코드가 중복될 수 있음
  - Spring 내부의 예외를 처리하는 것이 어려움
  - 예외가 WAS까지 전달되고, WAS의 에러 요청 전달이 진행됨

#### 3. @ExceptionHandler
> Controller의 메소드 & @ControllerAdvice나 @RestControllerAdvice가 있는 클래스의 메소드 를 손쉽게 처리할 수 있음

- ExceptionHandlerExceptionResolver에 의해 처리 가능
```java
@RestController
@RequiredArgsConstructor
public class ProductController {

  private final ProductService productService;
  
  @GetMapping("/product/{id}")
  public Response getProduct(@PathVariable String id){
    return productService.getProduct(id);
  }

  @ExceptionHandler(NoSuchElementFoundException.class)
  public ResponseEntity<String> handleNoSuchElementFoundException(NoSuchElementFoundException exception) {
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(exception.getMessage());
  }
}
```

- Exception 클래스들을 속성으로 받아 처리할 예외를 지정할 수 있음
- 에러 응답(payload)을 자유롭게 다룰 수 있음

```java
@RestController
@RequiredArgsConstructor
public class ProductController {

    ...

    @ExceptionHandler(NoSuchElementFoundException.class)
    public ResponseEntity<ErrorResponse> handleItemNotFoundException(NoSuchElementFoundException exception) {
        ...
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleMethodArgumentNotValid(MethodArgumentNotValidException ex) {
        ...
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleAllUncaughtException(Exception exception) {
        ...
    }
}
```

- **@ExceptionHandler에 등록된 예외 클래스**와 **파라미터로 받는 예외 클래스**가 동일해야 함
    - 값이 다를 때, `Runtime` 시점에 에러 발생

#### @ControllerAdvice & @RestControllerAdvice
> 전역적으로 @ExceptionHandler를 적용할 수 있음<br>

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@ControllerAdvice
@ResponseBody
public @interface RestControllerAdvice {
    ...
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface ControllerAdvice {
    ...
}
```

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NoSuchElementFoundException.class)
    protected ResponseEntity<?> handleNoSuchElementFoundException(NoSuchElementFoundException e) {
        final ErrorResponse errorResponse = ErrorResponse.builder()
                .code("Item Not Found")
                .message(e.getMessage()).build();

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorResponse);
    }
}
```
- 특정 클래스에만 제한적으로 적용하고 싶을 때는 @RestControllerAdvice의 basePackages 등을 설정해 제한할 수 있음
- 스프링 예외를 미리 처리해둔 ResponseEntityExceptionHandler를 추상 클래스로 제공
- ControllerAdvice 클래스가 ResponseEntityExceptionHandler를 상속받게 됨

- 장점
  - 하나의 클래스로 모든 컨트롤러에 대해 전역적으로 예외 처리가 가능함
  - 직접 정의한 에러 응답을 일관성있게 클라이언트에게 내려줄 수 있음
  - 별도의 try-catch문이 없어 코드의 가독성이 높아짐
- 주의점
  - 한 프로젝트당 하나의 ControllerAdvice만 관리하는 것이 좋다.
  - 만약 여러 ControllerAdvice가 필요하다면 basePackages나 annotations 등을 지정해야 한다.
  - 직접 구현한 Exception 클래스들은 한 공간에서 관리한다.


---

cf:: https://mangkyu.tistory.com/204
