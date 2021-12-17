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



# 