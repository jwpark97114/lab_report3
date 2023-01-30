# StringServer and JUnit

**Implementing String Server in Java**

To create StringServer, I used the server structure we had on our second lab files. 
So the codes are as follows

```

import java.io.IOException;
import java.io.OutputStream;
import java.net.InetSocketAddress;
import java.net.URI;
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;

class Handler {

    public static String[] inputList = new String[500];
    int counter = 0;
    String returnS = "";

    public String handleRequest(URI url) {
        if (url.getPath().contains("/add-message")) {
            String[] inputs = url.getQuery().split("=");

            inputList[counter] = inputs[1];
            counter++;
            returnS = "";
            for (int i = 0; i < counter; i++) {
                returnS = returnS + '\n' + inputList[i];
            }

            return String.format(returnS);

        } else {

            return "404 not found";

        }
    }
}

class ServerHttpHandler implements HttpHandler {
    Handler handler;

    ServerHttpHandler(Handler handler) {
        this.handler = handler;
    }

    public void handle(final HttpExchange exchange) throws IOException {
        // form return body after being handled by program
        try {
            String ret = handler.handleRequest(exchange.getRequestURI());
            // form the return string and write it on the browser
            exchange.sendResponseHeaders(200, ret.getBytes().length);
            OutputStream os = exchange.getResponseBody();
            os.write(ret.getBytes());
            os.close();
        } catch (Exception e) {
            String response = e.toString();
            exchange.sendResponseHeaders(500, response.getBytes().length);
            OutputStream os = exchange.getResponseBody();
            os.write(response.getBytes());
            os.close();
        }
    }
}

class Server {

    public static void start(int port, Handler handler) throws IOException {
        HttpServer server = HttpServer.create(new InetSocketAddress(port), 0);

        // create request entrypoint
        server.createContext("/", new ServerHttpHandler(handler));

        // start the server
        server.start();
        System.out.println("Server Started! Visit http://localhost:" + port + " to visit.");
    }
}

class StringServer {
    public static void main(String[] args) throws IOException {
        int port = 5000;
        Server.start(port, new Handler());
    }
}

``` 

After running the code, I tried the following lines.

>http://localhost:5000/add-message?s=abcd

![cap1](https://user-images.githubusercontent.com/66867608/215407184-f98aa6cd-5d86-4086-8129-80b16334e953.png)

After this request

> `handleRequest(URI url)`
> `getQuery().split("=")`
> `String.format(returnS)`

are called, and the parameters are like

> * `url` equals http://localhost:5000/add-message?s=abcd
> * `returnS` equals "abcd"

And then as `String[] inputLust` is now {"abcd", null, null, ... , null}



>http://localhost:5000/add-message?s=icici

![cap2](https://user-images.githubusercontent.com/66867608/215407215-a98211d0-8f5e-4c36-b6f7-ca3adf93c4be.png)

After this request the same

> `handleRequest(URI url)`
> `getQuery().split("=")`
> `String.format(returnS)`

are called, and the parameters are like

> * `url` equals `http://localhost:5000/add-message?s=icici`
> * `returnS` equals `"abcd \n icici"`

And then as `String[] inputLust` is now {"abcd", "icici", null, ... , null}



** JUnit Tests from lab 3 **

While testing the reversed method from ArrayTests, I found the following results

Failure inducing test:
```
@Test
public void testReversed(){
int[] original = {1,2,3,4};
int[] reversed = {4,3,2,1,};
assertArrayEquals(reversed, ArrayExamples.reversed(original));
}
```

<img width="689" alt="스크린샷 2023-01-26 오전 10 34 37" src="https://user-images.githubusercontent.com/66867608/215412383-6bcd89fb-b787-4d7e-8c5a-b3f223599690.png">


Successful test:
```
@Test
public void testReversed(){
int[] original = {0};
int[] reversed = {0};
assertArrayEquals(reversed, ArrayExamples.reversed(original));
}
```
<img width="876" alt="스크린샷 2023-01-29 오후 11 31 11" src="https://user-images.githubusercontent.com/66867608/215414722-172a5f51-9ae3-4c1e-ab3e-86d124963981.png">




Then by changing the code from this one 

```
  static int[] reversed(int[] arr) {
    int[] newArray = new int[arr.length];
    for(int i = 0; i < arr.length; i += 1) {
      arr[i] = newArray[arr.length - i - 1];
    }
    return arr;
  }
```

to this one

```
static int[] reversed(int[] arr) {
    int[] newArray = new int[arr.length];
    for(int i = 0; i < arr.length; i += 1) {
      newArray[i] = arr[arr.length - i - 1];
    }
    return newArray;
  }
```

I was able to get successful results as follows

<img width="876" alt="스크린샷 2023-01-29 오후 11 17 51" src="https://user-images.githubusercontent.com/66867608/215412989-e505cf17-5dfa-4b3a-8b9d-8cf7b84ff784.png">

The reason it was causing error was due to the line `arr[i] = newArray[arr.length - i - 1];`

It was assigning 0 to `arr[i]` the whole time as newArray was a newly created array filled with 0.
Then the line `return arr` returned original arr filled with 0.

So by changing it as `newArray[i] = arr[arr.length - i - 1];` and return newArray, I was able to fix the problem.


**Part 3**

I was able to learn how http requests and responses are made. As I never studied how web pages work, I feel like I'm now starting to understand the basics of internet structure, and I hope to expand my interest into detailed webpages or networks in the near future. 


