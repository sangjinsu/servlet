# 서블릿, JSP, MVC 패턴

## HttpServletRequest 역할

- 개발자 대신 HTTP 요청 메시지 파싱
- HttpServletRequest 객체에 담아서 제공

 #### HttpServletRequest, HttpServletResponse 를 사용할 때 가장 중요한 점은 이 객체들이 Http 요청 / 응답 메시지를 편리하게 사용하도록 돕는다. HTTP 에 대한 이해가 필요하다 

개발자를 위한  HTTP 지식 강의를 듣자 



### 복수 파라미터에서 단일 파라미터 조회

`?username=jinsu&username=wanhee`

- 중복일 때 `request.getParameterValues()`  사용
- 단일일 때 `request.getParamerter()` 사용 // 증복인 경우 첫번 째 값 반환



### HTTP Body message

- Http 메시지 바디의 데이터를 InputStream을 사용해서 직접 읽는다 

```java
@WebServlet(name = "RequestBodyStringServlet", value = "/RequestBodyStringServlet")
public class RequestBodyStringServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletInputStream inputStream = req.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
        System.out.println("messageBody = " + messageBody);

        resp.getWriter().write("ok");
    }
}
```

- InputStream은 byte 코드를 반환한다. byte 코드를 문자형으로 보내려면 문자표(Charset) 설정이 필요하다 



### JSON

```java
@WebServlet(name = "RequestJsonServlet", value = "/request-body-json")
public class RequestBodyJsonServlet extends HttpServlet {

    private ObjectMapper objectMapper = new ObjectMapper();

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletInputStream inputStream = req.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
        System.out.println("messageBody = " + messageBody);

        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);

        System.out.println("helloData.username = " + helloData.getUsername());
        System.out.println("helloData.age = " + helloData.getAge());

        resp.getWriter().write("ok");
    }
}
```

- JSON 결과를 파싱해서 자바 객체로 변환하기 위해서는 Jackson, Gson 같은 JSON 변환 라이브러리를 추가해서 사용해야 한다. 스프링부트로 Spring MVC 를 선택하면 Jackson 의 ObjectMapper 를 제공한다 

  

## HttpServletResponse

- 응답 메시지 생성

  - 응답코드 지정
  - 헤더 생성
  - 바디 생성
  - 쿠키 생성 등등등

  ### ResponseJsonServlet

  ```java
      @Override
      protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          resp.setHeader("content-type", "application/json");
          resp.setCharacterEncoding("utf-8");
  
          HelloData data = new HelloData();
          data.setUsername("sang");
          data.setAge(27);
  
          String result = objectMapper.writeValueAsString(data);
          resp.getWriter().write(result);
      }
  ```

  - `objectMapper.writeValueAsString()` 을 사용하면 객체를 JSON문자로 변경할 수 있다 

    

# 회원 관리 웹 애플리케이션

### 템플릿 엔진이 나온 이유

- HTML 문서에 동적으로 변경해야 하는 부분만 자바 코드를 넣을 수 있게 한다 

- 필요한 곳만 코드를 적용해 동적으로 변경한다 

  

## JSP

- import

```jsp
<%@ page import="hello.servlet.domain.Member" %>
```

- java 입력

```jsp
<ul>
    <li>id=<%=member.getId()%>
    </li>
    <li>username=<%=member.getUsername()%>
    </li>
    <li>age=<%=member.getAge()%>
    </li>
</ul>
```

- java 출력

```jsp
    <%
        for (Member member : members) {
            out.write("    <tr>");
            out.write("        <td>" + member.getId() + "</td>");
            out.write("        <td>" + member.getUsername() + "</td>");
            out.write("        <td>" + member.getAge() + "</td>");
            out.write("    </tr>");
        }
    %>
```



# 서블릿과 JSP 한계

- 서블릿으로 개발할 때 뷰 화면을 위한 html 만드는 작업이 자바 코드에 섞여서 지저분하고 복잡
- JSP는 뷰를 생성하는 html 작업이 깔끕하게 되고 동적으로 변경이 가능해졌다
- JSP 단점
  - JSP 에 너무 많은 역할이 몰려 자바 코드, 데이터 조회, html 등 다양한 코드가 모여져 있다 

# MVC 패턴

- 비즈니스 로직은 서블릿처럼 다른 곳에서 처리하고 JSP 는 목적에 맞게 HTML 로 화면을 그리는 일에 집중한다 

#### 너무 많은 역할

- 한 서블릿이나 JSP만으로 비즈니스 로직과 뷰 렌더링까지 처리하면 너무 많은 역할로 유지보수 어려움 
- 비즈니스 로직을 호출하는 부분에 변경이 발생해도... UI 를 변경해도 ...... 해당 파일을 수정해야 한다 

#### *<u>**변경의 라이프 사이클이 다르다**</u>* 

- UI 수정과 비즈니스 로직 수정하는 일이 각각 다르게 발생할 가능성이 높다 
- 대부분 영향을 주지 않는다
- 변경에 대한 라이프 사이클이 다른 경우에는 한 코드로 관리하기 힘들다 

#### 기능 특화

- JSP 같은 뷰 템플릿은 화면을 렌더링 하는데 최적화 되어 있어서 이 부분의 업무만 담당하는 것이 가장 효과적이다

## Model View Controller

서블릿이나 JSP 처럼 한 코드로 관리하던 것을 컨트롤러, 뷰라는 영역으로 서로 역할을 나눈 것이다

- 컨트롤러: HTTP 요청을 받아 파라미터를 검증하고 비즈니스 로직 실행, 그리고 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다 
- 모델: 뷰로 출력할 데이터를 담아둔다. 뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되고 화면을 렌더링 하는 일에 집중한다 
- 뷰: 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다 

컨트롤러에 비즈니스 로직을 두면 컨트롤러는 너무 많은 역할을 수행한다. 비즈니스 로직은 서비스라는 계층을 별도로 만들어서 처리한다. 컨트롤러는 비즈니스 로직이 있는 서비스를 호출하는 담당이다. 

### redirect vs forward

리다이렉트는 클라이언트에 응답이 나갔다가 클라이언트가 redirect 경로로 다시 요청한다. 따라서 클라이언트가 인지할 수 있고 URL 이 변경된다.

포워드는 서버 내부에서 일어나는 호출로 클라이언트가 인지하지 못한다



### MVC 패턴 한계

MVC 패턴을 적용한 덕분에 컨트롤러의 역할과 뷰의 렌더링한 역할 구분

##### 단점

1. 포워드 중복

   - View 로 이동하는 코드가 항상 중복 호출되어야 한다 

   ```java
           RequestDispatcher dispatcher = req.getRequestDispatcher(viewPath);
           dispatcher.forward(req, resp);
   ```

2. ViewPath 중복 

   ```java
           String viewPath = "/WEB-INF/views/new-form.jsp";
   ```

3. 사용하지 않는 코드 
   - 특히 response는 현재 코드에서 사용되지 않는다 

4. ***<u>공통 처리가 어렵다</u>***

- 기능이 복잡할 수록 컨트롤러에서 공통으로 처리해야하는 역할이 증가한다 
- 공통 기능을 메서드로 뽑으면 될 것 같지만 메서드를 항상 호출해야 한다.

#### 공통 처리가 어렵다는 문제가 있다 

- 프론트 컨트롤러 패턴을 도입한다 

  