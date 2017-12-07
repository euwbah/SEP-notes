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

JSP:

```jsp
<form action="somePage.jsp" type="GET|POST">
    A: <input type="text" name="text1" />
    B: <input type="password" name="password1" />
    C: <textarea name="textarea1" />

    D:
    <input type="radio" name="radio1" value="A" />
    <input type="radio" name="radio1" value="B" />
    <input type="radio" name="radio1" value="C" />

    E:
    <select name="list1">
        <option value="1" selected>one</option>
        <option value="2" selected>two</option>
        <option value="3" selected>three</option>
    </select>

    submitOption:
    <button type="submit" name="submit" value="option1" />
    <button type="submit" name="submit" value="option2" />
    <input type="reset" value="clear" />
</form>
```

Retrieving data from request:

```java
    String A = request.getParameter("text1");
    String B = request.getParameter("password1");
    String C = request.getParameter("textarea1");
    String D = request.getParameter("radio1"); // A | B | C
    String E = request.getParameter("list1"); // one | two | three
    String submitOption = request.getParameter("submit"); // option1 | option2
```


## WebService REST

### GET

```java
@Path("getUser")
@GET
@Produces("application/json")
public Response getUser(@QueryParam("userid") String userid) {
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
}
```

#### Summary

- `@Path`
- `@GET`
- `@Produces`
- `public Response foobar(@QueryParam("urlKey") Type varname)`
- `try {`
- **Connection**: DriverManager.getConnection
- **PreparedStatement**: Connection.prepareStatement
- `PreparedStatement.setInt/Long/String/Whatever(1basedindex, value)`
- **ResultSet**: PreparedStatement.executeQuery()
- `ResultSet.next() for multiple results`
- `new GenericEntity<T>(POJO)` **OR** primitive type / string
- `Response` -> `.status(respCode)` -> `entity(GenericEntity)` -> `build()`
- `catch (any exception) {`
- `Response` -> `.status(404 or some error)` -> build()

### Post

Instead of `@GET` use the `@POST` annotation

SQL Statement: `UPDATE table SET key1=?, key2=? WHERE id=?`

instead of `PreparedStatement.executeQuery()` it becomes `PreparedStatement.executeUpdate()`

If `.executeUpdate` returns > 0, the operation is successful

### Delete

Use `@DELETE` annotation

SQL Statement: `DELETE FROM table WHERE id=?`

Also use `PreparedStatement.executeUpdate()`

If `.executeUpdate` returns > 0, the operation is successful

### PUT

Use `@PUT` annotation

SQL Statement: `INSERT INTO table (col1, col2, col3) VALUES (v1, v2, v3), (v11, v22, v33)`

Also use `PreparedStatement.executeUpdate()`

If `.executeUpdate` returns > 0, the operation is successful


## Servlet

### GET

```java
HttpSession session = request.getSession();
String userid = request.getParameter("userid");

Client client = ClientBuilder.newClient();
WebTarget target = client
  .target("http://localhost/webservice")
  .path("searchUser")
  .queryParam("userid", userid);

Invocation.Builder invocationBuilder = target.request(MediaType.APPLICATION_JSON);
Response res = invocationBuilder.get();
UserDetails ud = (UserDetails) res.readEntity(UserDetails.class);
session.setAttribute("user", ud);
response.sendRedirect("http://localhost/webservice/searchUser");
```

#### Summary

- **Client**: `ClientBuilder.newClient()`
- **WebTarget**: 
    + `Client`
    + `.target(endpointRoot)`
    + `.path(endpointPath)`
    + `.queryParam("key", value)`
    + `.queryParam("key2", value2)`
    + etc...
- **Invocation.Builder**: `target.request(MediaType.APPLICATION_JSON)`
- **Response**: `Invocation.Builder.get()`
- **POJO**: `Response.readEntity(POJO.class)`
- **HttpSession**: `request.getSession()`
- `HttpSession.setAttribute("key", POJO)`
- `response.sendRedirect(path)`

### POST/PUT/DELETE

instead of `Invocation.Builder.get()`, use:

    Invocation.Builder.put(Entity.entity("", "application/json"));
    Invocation.Build.post(null);
    Invocation.Builder.delete();

Use `Response.getStatus() : int` to get the response status