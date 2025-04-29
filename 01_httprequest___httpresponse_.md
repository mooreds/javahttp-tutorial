# Chapter 1: HTTPRequest & HTTPResponse

Welcome to the `src` HTTP server tutorial! We're starting with the absolute basics: how a client (like your web browser) and our server talk to each other.

Imagine ordering food at a restaurant:

1.  You give the waiter an **order ticket** with what you want (e.g., "Burger, medium-rare, no onions").
2.  The kitchen prepares your food.
3.  The waiter brings your **prepared meal** back to you on a tray.

In the world of web servers, this communication happens using two key concepts: `HTTPRequest` and `HTTPResponse`.

*   `HTTPRequest`: This is like the **order ticket**. It contains all the details of what the client (e.g., your browser) is asking the server for.
*   `HTTPResponse`: This is like the **prepared meal** on the tray. It's what the server sends back to the client.

This chapter will explain what information goes into these "tickets" and "trays". Understanding them is the first step to building web applications!

## The Order Ticket: `HTTPRequest`

When your browser wants to load a webpage, send form data, or get some data from a server, it sends an `HTTPRequest`. Our server receives this request and represents it using the `HTTPRequest` object.

Think of it as a structured note containing everything the server needs to know:

1.  **Path (URL):** What specific resource is the client asking for?
    *   Examples: `/index.html`, `/users/profile`, `/search?query=dogs`
    *   This is like specifying "the menu" or "table number 5's order".
2.  **Method:** What action does the client want to perform with that resource?
    *   Common methods:
        *   `GET`: "Please **give me** the resource at this path." (Used for fetching pages, data)
        *   `POST`: "Please **accept this data** I'm sending you for this path." (Used for submitting forms, creating new data)
    *   Think of `GET` as asking for the menu, and `POST` as giving your completed order form to the waiter.
3.  **Headers:** Extra information or instructions for the server.
    *   Examples:
        *   `Accept: text/html`: "I prefer the response in HTML format."
        *   `User-Agent: Chrome/90...`: "I'm using the Chrome browser."
        *   `Authorization: Bearer ...`: "Here are my credentials to prove who I am."
    *   These are like adding "no onions" or "make it spicy" to your food order.
4.  **Cookies:** Small pieces of data the server previously gave to the client to remember them.
    *   Example: `session-id=abc123xyz`
    *   Like a loyalty card the restaurant gave you last time, which you now show to get a discount. The server uses cookies to remember who you are between requests (e.g., keeping you logged in).
5.  **Body:** (Optional) The actual data the client is sending, usually with `POST` requests.
    *   Examples: Data from a login form (`username=alice&password=secret`), uploaded files, JSON data.
    *   This is the content of the order form you filled out. `GET` requests usually don't have a body.

Let's look at some key parts of the `HTTPRequest` object in our code (don't worry about the details yet, just see the names):

```java
// Simplified from main/java/io/fusionauth/http/server/HTTPRequest.java

public class HTTPRequest {
    // What action? (GET, POST, etc.)
    private HTTPMethod method;

    // What resource? (/index.html, /users)
    private String path;

    // Extra instructions (Accept, User-Agent, etc.)
    private Map<String, List<String>> headers;

    // Data remembered from previous visits
    private Map<String, Cookie> cookies;

    // The actual data sent (for POST, etc.) - accessed via an InputStream
    private InputStream inputStream;

    // Parsed query parameters from the path (e.g., ?query=dogs)
    private Map<String, List<String>> urlParameters;

    // Parsed form data from the body (if Content-Type is form data)
    private Map<String, List<String>> formData;

    // Information about uploaded files (if it's a multipart request)
    private List<FileInfo> files;

    // Other info derived from the request
    private String scheme; // http or https
    private String host;   // example.com
    private int port;      // 80, 443, etc.
    private String ipAddress; // Client's IP address

    // ... other fields and methods ...
}
```

This `HTTPRequest` object neatly packages everything the client sent. Our server code will read information from this object to figure out what to do next.

## The Prepared Meal: `HTTPResponse`

Once the server understands the `HTTPRequest` and has done its work (like fetching data from a database or preparing an HTML page), it needs to send something back. This is the `HTTPResponse`, represented by the `HTTPResponse` object.

Think of it as the tray carrying the meal back to the customer:

1.  **Status Code & Message:** Was the request successful?
    *   Examples:
        *   `200 OK`: "Everything went perfectly, here's your resource!"
        *   `404 Not Found`: "Sorry, I couldn't find the resource you asked for."
        *   `500 Internal Server Error`: "Oops, something went wrong on my end."
        *   `302 Found`: "The resource you asked for is actually somewhere else, go here instead." (Redirect)
    *   This is like the waiter saying "Here's your burger!" or "Sorry, we're out of fish today."
2.  **Headers:** Extra information about the response being sent back.
    *   Examples:
        *   `Content-Type: text/html`: "The body of this response is HTML."
        *   `Content-Length: 1234`: "The body is 1234 bytes long."
        *   `Set-Cookie: session-id=xyz456; HttpOnly`: "Please remember this new cookie for next time."
    *   Like the label on the food container saying "Burger, Medium-Rare".
3.  **Cookies:** New or updated cookies for the client to store.
    *   Often used after login to give the client a session cookie.
4.  **Body:** The actual content being sent back to the client.
    *   Examples: HTML for a webpage, JSON data for an API, image data, CSS styles.
    *   This is the actual food on the tray!

Here's a simplified look at the `HTTPResponse` object:

```java
// Simplified from main/java/io/fusionauth/http/server/HTTPResponse.java

public class HTTPResponse {
    // Success? Error? (200, 404, 500, etc.)
    private int status = 200; // Default to OK
    private String statusMessage; // e.g., "OK", "Not Found"

    // Extra info about the response (Content-Type, etc.)
    private Map<String, List<String>> headers;

    // New or updated cookies for the client
    private Map<String, Map<String, Cookie>> cookies;

    // The actual content (HTML, JSON, etc.) - written via an OutputStream
    private HTTPOutputStream outputStream;

    // Optional exception if something went wrong
    private Throwable exception;

    // ... other fields and methods ...
}
```

Our server code will create this `HTTPResponse` object, set the status, add necessary headers (like `Content-Type`), maybe add some cookies, and then write the actual response content (like HTML) into its `outputStream`. The server framework then takes this object and sends it back to the client.

## Helper Objects: Cookies and FileInfo

You might have noticed `Cookie` and `FileInfo` mentioned. These are smaller objects used within the `HTTPRequest` and `HTTPResponse`:

*   **`Cookie`**: Represents a single HTTP cookie, holding its name, value, domain, path, expiration, and security flags. It's used both to *read* cookies from the `HTTPRequest` and to *set* cookies in the `HTTPResponse`.

    ```java
    // Simplified from main/java/io/fusionauth/http/Cookie.java
    public class Cookie {
        public String name;
        public String value;
        public String domain; // e.g., ".example.com"
        public String path;   // e.g., "/" or "/app"
        public ZonedDateTime expires; // When it expires
        public boolean httpOnly; // Cannot be accessed by JavaScript?
        public boolean secure;   // Only send over HTTPS?
        // ... other fields like MaxAge, SameSite ...
    }
    ```

*   **`FileInfo`**: Used specifically within `HTTPRequest` when the client uploads files (usually via an HTML form with `enctype="multipart/form-data"`). It holds information about each uploaded file.

    ```java
    // Simplified from main/java/io/fusionauth/http/FileInfo.java
    public class FileInfo {
        public Path file; // Temporary location where the server saved the file
        public String fileName; // Original name of the file from the client
        public String name; // Name of the form field used for the upload
        public String contentType; // Type of file (e.g., "image/jpeg")
        // ... encoding info ...
    }
    ```

## How They Interact (A Sneak Peek)

We won't dive deep into the code that *uses* these objects yet (that's for later chapters!), but let's visualize the flow:

1.  **Client Sends Request:** Your browser sends raw text data (like `GET /index.html HTTP/1.1\nHost: example.com\n...`) over the network.
2.  **Server Receives & Parses:** Our [HTTPServer](04_httpserver_.md) listens for this data. Internal components ([HTTP Worker & Server Thread](05_http_worker___server_thread_.md) and [HTTP I/O Streams (Input & Output)](06_http_i_o_streams__input___output__.md)) read the raw data and *parse* it.
3.  **`HTTPRequest` Created:** The parser creates an `HTTPRequest` object and fills it with the information extracted from the raw request (method, path, headers, etc.).
4.  **Handler Processes:** The server passes this `HTTPRequest` object to your application code (an [HTTPHandler](02_httphandler_.md), which we'll cover next).
5.  **Handler Creates `HTTPResponse`:** Your handler code examines the `HTTPRequest`. It decides what to do and creates an `HTTPResponse` object.
6.  **Handler Populates `HTTPResponse`:** Your handler sets the status code (e.g., 200), headers (e.g., `Content-Type: text/html`), and writes the response body (e.g., HTML content) to the `HTTPResponse`'s output stream.
7.  **Server Sends Response:** The server takes the completed `HTTPResponse` object, formats it back into raw HTTP text (like `HTTP/1.1 200 OK\nContent-Type: text/html\n...`), and sends it back to the client over the network using [HTTP I/O Streams (Input & Output)](06_http_i_o_streams__input___output__.md).
8.  **Client Receives:** The browser receives the raw response data, parses it, and displays the webpage or uses the data.

Here's a simplified diagram of the server's role:

```mermaid
sequenceDiagram
    participant Client
    participant ServerNetwork as Server Networking Layer
    participant Parser as Request Parser
    participant Handler as Your Application Code (HTTPHandler)
    participant Formatter as Response Formatter
    participant ServerResponse as Server Networking Layer

    Client->>+ServerNetwork: Sends Raw HTTP Request Text
    ServerNetwork->>+Parser: Passes Raw Text
    Parser->>-ServerNetwork: Done Reading
    Parser->>Handler: Creates and Passes HTTPRequest object
    Handler->>Handler: Examines HTTPRequest, Decides Response
    Handler->>Formatter: Creates and Populates HTTPResponse object
    Formatter->>Formatter: Formats HTTPResponse into Raw Text
    Formatter->>+ServerResponse: Passes Raw HTTP Response Text
    ServerResponse->>-Formatter: Done
    ServerResponse->>-Client: Sends Raw HTTP Response Text
```

## Conclusion

You've learned about the two fundamental objects for web communication in our server:

*   `HTTPRequest`: Represents the client's request (the "order ticket").
*   `HTTPResponse`: Represents the server's response (the "prepared meal").

These objects encapsulate all the details needed for the client and server to understand each other. They hold information like the requested path, HTTP method, headers, cookies, status codes, and the actual data being exchanged (the body).

Now that you know *what* information is passed back and forth, you're ready to see *how* our server code actually handles these requests and generates responses.

Let's move on to the next chapter: [HTTPHandler](02_httphandler_.md), where you'll learn how to write the code that processes an `HTTPRequest` and builds an `HTTPResponse`.

---

Generated by [AI Codebase Knowledge Builder](https://github.com/The-Pocket/Tutorial-Codebase-Knowledge)