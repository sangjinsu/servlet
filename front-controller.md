# MVC 프레임워크 만들기

### 프론트 콘트롤러 패턴 특징

- 프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받음
- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출
- 공통 처리 가능
- 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿으로 사용하지 않아도 된다 

### 스프링 웹 MVC 와 프론트 컨트롤러

- 스프링 웹 MVC DispatcherServlet이 FrontController 패턴으로 구현되어 있음

  

### 서블릿 종속성 제거

- 컨트롤러가 Http ServletRequest, HttpServletResponse 가 필요하지 않은 경우가 있다 
- request 객체를 Model로 사용하는 대신에 별도의 Model 객체를 만들어서 반환하면 된다



#### 뷰 이름 중복 제거 

- 컨트롤러에서 지정하는 뷰 이름에 중복이 있음

- 컨트롤러가 뷰의 논리 이름을 반환하고 실제 물리 위치 이름은 프론트 컨트롤러에서 처리하도록 단순화 한다 

   

## 스프링 MVC 구조 이해

### DispatcherServlet 구조 이해하기

스프링 MVC 도 프론트 컨트롤러 패턴으로 구현되어 있다 

스프링 MVC의 프론트 컨트롤러가 바로 디스패처 서블릿이다 



#### DispatcherServlet 등록

- DispatcherServlet 도 부모 클래스에서 HttpServlet을 상속 바다 사용하고 서블릿으로 동작
- DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
- 모든 경로에 대해 매핑하며 더 자세한 경로가 우선순위가 높다 



#### 요청 흐름

- 서블릿 호출되면 HttpSErvlet  service  호출
- FrameworkServlet.service() 를 시작으로 해서 여러 메서드가 호출되면서 DispatcherServlet.doDispatch() 호출 



#### 동작 순서 

1. 핸들러 조회: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러를 조회한다 
2. 핸들러 어댑터 조회: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다 
3. 핸들러 어댑터 실행: 핸들러 어댑터를 실행한다 
4. 핸들러 실행: 핸들러 어댑터가 실제 핸들러를 실행한다 
5. ModelAndView 바환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다 
6. viewResolver 호출: 뷰 리졸버를 찾고 실행한다 
   - JSP의 경우 InternalResourceViewResolver가 자동 등록되고 사용된다 

7. View 반환: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고 렌더링 역할을 담당하는 뷰 객체를 반환한다. 
   - JSP의 경우 InternalResourceView(JstlView)를 반환하는데, 내부에 forward() 로직이 있다 =. 

8. 뷰 렌더링: 뷰를 통해서 뷰를 렌더링한다.



### 핸들러 매핑과 핸들러 어댑터

#### OldController

```java
@Component("/springmvc/old-controller")
public class OldController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("OldController.handleRequest");
        return null;
    }
}
```

1. 핸들러 매핑으로 핸들러 조회
   1. HandlerMapping을 순서대로 실행해서 핸들러를 찾는다
   2. 이 경우 빈 이름으로 핸들러를 찾아야 하기 때문에 이름 그대로 빈 이름으로 핸들러를 찾아주는 BeanNameUrlHandlerMapping가 실행에 성공하고 핸들러인 Oldcontroller를 반환한다

2. 핸들러 어댑터 조회
   1. HandlerAdapter의 supports() 를 순서대로 호출한다
   2. SimpleControllerHandlerAdapter가 Controller 인터페이스를 지원하므로 대상이 된다 

3. 핸들러 어댑터 실행
   1. 디스패처 서블릿이 조회한 SimpleControllerHandlerAdapter를 실행하면서 핸들러 정보도 함께 넘겨준다
   2. SimpleControllerHandlerAdapter는 핸들러인 OldController를 내부에서 실행하고 그 결과를 반환한다.

#### 사용 객체

HandlerMapping = BeanNameUrlHandlerMapping

HandlerAdapter = SimpleControllerHandlerAdapter



#### HttpRequestHandler

```java
@Component("/springmvc/request-handler")
public class MyHttpRequestHandler implements HttpRequestHandler {
    @Override
    public void handleRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("MyHttpRequestHandler.handleRequest");
    }
}
```

1. 핸들러 매핑으로 핸들러 조회
   1. HandlerMapping을 순서대로 실행해서 핸들러를 찾는다
   2. 이 경우 빈 이름으로 핸들러를 찾아야 하기 때문에 이름 그대로 빈 이름으로 핸들러를 찾아주는 BeanNameUrlHandlerMapping가 실행에 성공하고 핸들러인 MyHttpRequestHandler를 반환한다

2. 핸들러 어댑터 조회
   1. HandlerAdapter의 supports() 를 순서대로 호출한다
   2. HttpRequestHandlerAdapter가 HttpRequestHandler 인터페이스를 지원하므로 대상이 된다 

3. 핸들러 어댑터 실행
   1. 디스패처 서블릿이 조회한 HttpRequestHandlerAdapter를 실행하면서 핸들러 정보도 함께 넘겨준다
   2. HttpRequestHandlerAdapter는 핸들러인 MyHttpRequestHandler를 내부에서 실행하고 그 결과를 반환한다.

#### 사용 객체

HandlerMapping = BeanNameUrlHandlerMapping

HandlerAdapter = HttpRequestHandlerAdapter



## 뷰 리졸버

1. 핸들러 어댑터 호출
2. ViewResolver 호출
   - new-form 이라는 뷰 이름으로 viewResolver를 순서대로 호출'
   - BeanNameViewResolver는 new-form 이라는 이름의 스프링 빈으로 등록된 뷰를 찾아야 하는데 없다
   - IntenalResourceViewResolver 가 호출된다 
3. InternalResourceViewResolver
   - 이 뷰 리졸버는 InternalResourceView를 반환한다 

4. 뷰 - InternalResourceView
   - InternalResourceView는 JSP 처럼 포워드 forward() 를 호출해서 처리할 수 있는 경우에 사용한다 

5. view.render()
   - view.render()가 호출되고 InternalResourceView는 forward()를 사용해 JSP 를 실행한다.

JSP를 제외한 나머지 뷰 템플릿들은 forward() 과정 없이 바로 렌더링 된다. 

Thymelearf 뷰 템플릿을 사용하면 ThymeleafViewResolver를 등록해야 한다. 



















