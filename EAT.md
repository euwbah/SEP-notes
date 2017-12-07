# Enterprise application technology

## JSP

JSP files are Java Servlet preprocessors

### Scriptlet `<%`

Scriptlets are placed inline in the `_jspService()` method

```jsp
<%
    int x = Class.doStandardJavaCodeHere();
%>
```

### Declaration `<%!`

Declarations are placed at the class-body level in the Servlet class

**THEY DO NOT HAVE ACCESS TO IMPLICIT OBJECTS** as implicit objects are only available in the `_jspService()` scope.

```jsp
<%! 
    public int myProperty = 3;

    int cubeMyPropertyThenTimes(int n) {
        return (int) Math.pow(myProperty, 3) * n;
    }
%>
```

### Expressions `<%=`

Expressions represent a value per-se

```jsp
this is cubeMyPropertyThenTimes(5) <%= this.cubeMyPropertyThenTimes(5) %>
```

### `<%@page` directive

Note: multiple page directive tags can be used, repeat keys in the KVPs will be overriden

```jsp
<%@page import="java.util.*" contentType="text/html"%>
```

### `<%@include` directive

Injects a JSP file inline at this location

```jsp
<%@include file="navbar.jsp" %>
```

### JSP implicit objects

These variables can only be accessed from within the `_jspService()` scope, *i.e. either in a scriplet or expression*. Declarations are not placed in the `_jspService()` scope, but in the class scope instead.

#### **`out : PrintWriter`**

Use this to append HTML to the browser view

    <% out.println("<a href='welcome.jsp'>click me</a>"); %>

To get this object in the servlet:

    HttpServletResponse.getWriter()

#### **`request : HTTPServletRequest`**

Use this to get HTTP parameters from body of any REST call

##### Example 1: REST request parameters

in `caller.jsp`:

```jsp
<form action="submitUsername.jsp" method="POST">
    <p>
        username: <input name="username" type="text" />
    </p>
</form>
```

in `submitUsername.jsp`:

    username was <%= request.getParameter("username") %>

##### Example 2: Send request attributes from servlet

in `servlet.java`:

```java
    protected void doGet(HttpServletRequest request, HttpServletRepsonse response) {
        Unfried something = (Unfried) request.getParameter("unfried");
        request.setAttribute("deep fried", something.deepFry());
    }
```

#### **`response : HttpServletResponse`**

usually for `sendRedirect("somepage.jsp")`

    <%
        session.setAttribute("error", "du gott das zucc")
        response.sendRedirect("index.jsp") 
    %>

The `HttpServletResponse` is used in the Servlet to get access to the PrintWriter object like so: `HttpServletResponse.getWriter()`

#### **`session : HttpSession`**

    <% session.setAttribute("some key", someObject) %>

To get the `HttpSession` in the Servlet instead:

```java
    // where:
    HttpServletRequest request = from_method_arguments;

    request.getSession(true); // if need to create a session if it doesn't already exist

    // or simply,

    request.getSession(); // if the session is already expected to exist
```

### JSP Forms

#### Form syntax:

```jsp
<form action="somePage.jsp" type="GET|POST">
    <input type="text" name="asdf" />
    <button type="submit" name="submit" value="button1" />
</form>
```




## WebService REST

```java
try {
  Connection c = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/userDB?user=root&password=123456");

  PreparedStatement ps = c.prepareStatement(
    """
    select   ud.userid,
      ud.age,
      ud.gender
    from
      user_details ud
    where
      ud.userid=?
    """);

  ps.setInt(1, Int.valueOf(userid));

  ResultSet rs = ps.executeQuery();

  rs.next();

  UserDetails res = new UserDetails(
    rs.getInt("userid"), 
    rs.getInt("age"), 
    rs.getString("gender"));

  GenericEntity<UserDetails> ge = new GenericEntity<UserDetails>(res);

  return Response
    .status(200)
    .entity(ge)
    .build();

} catch (Exception e) {
  return Response.status(404).build();
}
```

## Servlet

```java
HttpSession session = request.getSession();
String userid = request.getParameter("userid");

Client c = ClientBuilder.newClient();
WebTarget target = client
  .target("http://localhost/webservice")
  .path("searchUser")
  .queryParam("userid", userid);

Invocation.Builder invocationBuilder = target.request(MediaType.APPLICATION_JSON);
Response response = invocationBuilder.get();
UserDetails ud = (UserDetails) response.readEntity(UserDetails.class);
session.setAttribute("user", ud);
response.sendRedirect("http://localhost/webservice/searchUser");
```