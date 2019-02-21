# How to convert HTML to PDF in Java

## Documentation

See the full documentation on [PDFShift's documentation](https://docs.pdfshift.io/).

## Installation

For this we'll use Java 11's new HttpClient. But you can use any version of Java and any HTTP library of your choice. 
We use Java 11 here to avoid adding extra level of dependency.

### Requirements

* Java 11

## Usage

Your HTTP requests needs to be configured with your `api_key` received when creating an account.
Setting it is easy as:

```java
public void convert() throws Exception {
    var httpRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://api.pdfshift.io/v2/convert"))
            .timeout(Duration.ofSeconds(20))
            .header("Content-Type", "application/json")
            .header("Authentication", API_KEY)
            .POST(HttpRequest.BodyPublishers.ofFile(Paths.get("src/main/resources/body.json")))
            .build();

    var httpClient = HttpClient.newBuilder()
            .version(HttpClient.Version.HTTP_1_1)
            .build();

    var response = httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofInputStream());

    // Save the file locally
    var targetFile = new File("src/main/resources/targetFile.pdf");
    Files.copy(response.body(), targetFile.toPath(), StandardCopyOption.REPLACE_EXISTING);
}
```

Here we stored our json body inside a file called body.json in the projects resources folder. The contents of that file is

```json
{
    "source": "https://example.com",
    "sandbox": true
}
```

We also highly recommend checking for errors after the conversion is made, before processing the document, in order to avoid issues later on.
This can be easily handled with `requests` by doing the following:

```java
var statusCode = response.statusCode();
if (statusCode == 200 || statusCode == 201) {
    // save file locally
    var targetFile = new File("src/main/resources/targetFile.pdf");
    Files.copy(response.body(), targetFile.toPath(), StandardCopyOption.REPLACE_EXISTING);
} else {
   // error occurred
}
```

The `sandbox` parameter allows you to do unlimited conversion, but will add a watermark on top of the generated document.
No credits are deduced from your account when the sandbox mode is on.

### With a URL

Converting an URL with PDFShift is really easy. All you have to do is send a POST request with the `source` parameter set to the URL, like the following:

```java
public void convert() throws Exception {
    var httpRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://api.pdfshift.io/v2/convert"))
            .timeout(Duration.ofSeconds(20))
            .header("Content-Type", "application/json")
            .header("Authentication", API_KEY)
            .POST(HttpRequest.BodyPublishers.ofFile(Paths.get("src/main/resources/body.json")))
            .build();

    var httpClient = HttpClient.newBuilder()
            .version(HttpClient.Version.HTTP_1_1)
            .build();

    var response = httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofInputStream());

    // Save the file locally
    var statusCode = response.statusCode();
    if (statusCode == 200 || statusCode == 201) {
        var targetFile = new File("src/main/resources/targetFile.pdf");
        Files.copy(response.body(), targetFile.toPath(), StandardCopyOption.REPLACE_EXISTING);
    } else {
        // error occurred
    }
}
```

body.json
```json
{
    "source": "https://example.com",
    "sandbox": true
}
```

### With inline HTML data:

To convert a raw HTML data with PDFShift, simply send the raw string in the `source` parameter:

body.json
```json
{
  "source": "<html><head><title>Hello world</title><body><h1>Hello World</h1></body></head></html>",
  "sandbox": true
}
```

```java
public static void main(String... args) throws Exception {
    var httpRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://api.pdfshift.io/v2/convert"))
            .timeout(Duration.ofSeconds(10))
            .header("Content-Type", "application/json")
            .header("Authentication", API_KEY)
            .POST(HttpRequest.BodyPublishers.ofFile(Paths.get("src/main/resources/body.json")))
            .build();

    var httpClient = HttpClient.newBuilder()
            .version(HttpClient.Version.HTTP_1_1)
            .build();
    var response = httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofInputStream());

    var statusCode = response.statusCode();
    if (statusCode == 200 || statusCode == 201) {
        // success
    } else {
        // error occurred
    }
}
```


### Save the file to Amazon S3 and get an URL instead

By passing the `filename` parameter to your request, you will receive a JSON response instead of the binary PDF, with a `url` key that contains the path to the file stored on Amazon S3.
All files stored on Amazon S3 are kept for two days, then automatically deleted.

body.json
```json
{
  "source": "<html><head><title>Hello world</title><body><h1>Hello World</h1></body></head></html>",
  "filename": "result.pdf",
  "sandbox": true
}
```

```java
public static void main(String... args) throws Exception {
    var httpRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://api.pdfshift.io/v2/convert"))
            .timeout(Duration.ofSeconds(10))
            .header("Content-Type", "application/json")
            .header("Authentication", API_KEY)
            .POST(HttpRequest.BodyPublishers.ofFile(Paths.get("src/main/resources/body.json")))
            .build();

    var httpClient = HttpClient.newBuilder()
            .version(HttpClient.Version.HTTP_1_1)
            .build();
    var response = httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofString());

    var statusCode = response.statusCode();
    if (statusCode == 200 || statusCode == 201) {
        // Response body is a json string. You can use an external library to convert this to an object
    } else {
        // an error occurred
    }
}
```

### Accessing secured pages

If your `source` requires a BASIC AUTH mechanism, you can either use the custom headers part or use the `auth` parameter from the API that behaves the same.

body.json
```json
{
  "source": "https://httpbin.org/basic-auth/user/passwd",
  "auth": {
      "username": "user",
      "password": "passwd"
  },
  "sandbox": true
}
```
```java
public static void main(String... args) throws Exception {
    var httpRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://api.pdfshift.io/v2/convert"))
            .timeout(Duration.ofSeconds(10))
            .header("Content-Type", "application/json")
            .header("Authentication", API_KEY)
            .POST(HttpRequest.BodyPublishers.ofFile(Paths.get("src/main/resources/body.json")))
            .build();

    var httpClient = HttpClient.newBuilder()
            .version(HttpClient.Version.HTTP_1_1)
            .build();
    var response = httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofInputStream());

    var statusCode = response.statusCode();
    if (statusCode == 200 || statusCode == 201) {
        var targetFile = new File("src/main/resources/targetFile.pdf");
        Files.copy(response.body(), targetFile.toPath(), StandardCopyOption.REPLACE_EXISTING);
    } else {
        // an error occurred
    }
}
```

### Using cookies

Cookies might help you access unauthorized areas that aren't restricted by a simple Basic Auth mechanism. You can define as many cookies as you want.

body.json
```json
{
  "source": "https://httpbin.org/cookies",
  "cookies": [{"name": "session", "value": "4cb496a8-a3eb-4a7e-a704-f993cb6a4dac"}],
  "sandbox": true
}
```

```java
public static void main(String... args) throws Exception {
    var httpRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://api.pdfshift.io/v2/convert"))
            .timeout(Duration.ofSeconds(10))
            .header("Content-Type", "application/json")
            .header("Authentication", API_KEY)
            .POST(HttpRequest.BodyPublishers.ofFile(Paths.get("src/main/resources/body.json")))
            .build();

    var httpClient = HttpClient.newBuilder()
            .version(HttpClient.Version.HTTP_1_1)
            .build();
    var response = httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofString());

    var statusCode = response.statusCode();
    if (statusCode == 200 || statusCode == 201) {
        
    } else {
        // an error occurred
    }
}
```


### Loading CSS from an URL:

By passing a `css` parameter, you will be able to modify the page with your CSS.
This allows you to customize the rendering of the page.

You can also call multiple CSS by calling a root CSS (like "print.css" in that case) that will call @import in it for each CSS files.

```json
{
  "source": "https://www.example.com",
  "css": "https://www.example.com/public/css/print.css",
  "sandbox": true
}
```


### Loading CSS from a string:

Like for the `source` parameter, you can pass a raw set of CSS rules to the `css` parameter and they will be injected to the loaded document.

```json
{
  "source": "https://www.example.com",
  "css": "a {text-decoration: underline; color: blue}",
  "sandbox": true
}
```

### Adding Watermark

Some documents that you share need a watermark to clearly identify your brand. That's easy with PDFShift:

```json
{
  "source": "https://www.example.com",
  "watermark": {
    "image": "https://pdfshift.io/static/img/logo.png",
    "offset_x": 50,
    "offset_y": "100px",
    "rotate": 45
  },
  "sandbox": true
}
```

### Custom Header (or Footer)

You can add some custom header or footer to your generated document. These are often used to indicate the current page, or show the logo of your company on every page.

Note that the header and footer **are not related to the body**. For this reason, the CSS in your body doesn't apply to your header/footer.
By default, the font-size will be really small. You will have to set it manually, like in the following example:

```json
{
  "source": "https://www.example.com",
  "footer": {
    "source": "<div>My custom footer",
    "spacing": "50px"
  },
  "sandbox": true
}
```

### Protecting the generated PDF

Protecting your document is easy with PDFShift. You can specify a password for the user and for the owner.
(The owner will have full rights access while the user will have limited access based on your choice).

Please keep in mind that some PDF reader doesn't respect the rights as long as the user is authenticated.
This means that if you set an empty password for the user, with no rights to print or copy, some PDF reader will ignore this and still allow printing and copying.

This is outside of our capabilities here at PDFShift as we can't enforce a reader to respect PDF's standard.

```json
{
  "source": "https://www.example.com",
  "protection": {
    "user_password": "user",
    "owner_password": "owner",
    "no_print": true
  }
}
```


### Real life example - Sending an invoice by email

One frequent use of PDFShift is to generate an invoice when receiving a payment, and sending that invoice - in PDF format - by email, to the customer.

For this, we’re going to look at sending an email with attachments – using the core Java mail library. You will need the Java mail library in your path. If your project is a maven project, add the following lines in your pom.xml

```xml
<dependency>
    <groupId>javax.mail</groupId>
    <artifactId>mail</artifactId>
    <version>1.5.0-b01</version>
</dependency>
```

If it's a gradle project then add this to your dependencies

```groovy
    compile 'javax.mail:mail:1.5.0-b01'
```

For this tutorial I'll use this email service https://mailtrap.io. It's a fake email service provider used for development purposes. Create a free account on https://mailtrap.io and you'll get a username and password to use here. You can convert and send email using the code below

```java
private static void sendEmail() throws Exception {
    var httpRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://api.pdfshift.io/v2/convert"))
            .timeout(Duration.ofSeconds(10))
            .header("Content-Type", "application/json")
            .header("Authentication", API_KEY)
            .POST(HttpRequest.BodyPublishers.ofFile(Paths.get("src/main/resources/body.json")))
            .build();

    var httpClient = HttpClient.newBuilder()
            .version(HttpClient.Version.HTTP_1_1)
            .build();
    var response = httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofInputStream());

    var statusCode = response.statusCode();
    if (statusCode == 200 || statusCode == 201) {
        // save pdf to file targetFile.pdf
        var targetFile = new File("src/main/resources/targetFile.pdf");
        Files.copy(response.body(), targetFile.toPath(), StandardCopyOption.REPLACE_EXISTING);

        // Send pdf as email attachment
        var prop = new Properties();
        prop.put("mail.smtp.auth", true);
        prop.put("mail.smtp.starttls.enable", "true");
        prop.put("mail.smtp.host", "smtp.mailtrap.io");
        prop.put("mail.smtp.port", "25");
        prop.put("mail.smtp.ssl.trus", "smtp.mailtrap.io");

        var username = "get username from mailtrap.io";
        var password = "get password from mailtrap.io";

        var session = Session.getInstance(prop, new Authenticator() {
            @Override
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication(username, password);
            }
        });

        var message = new MimeMessage(session);
        message.setFrom(new InternetAddress("from@gmail.com"));
        message.setRecipients(
                Message.RecipientType.TO, InternetAddress.parse("to@gmail.com")
        );
        message.setSubject("Mail Subject");

        var attachment = new File("src/main/resources/targetFile.pdf");

        var mimeBodyPart = new MimeBodyPart();
        mimeBodyPart.setContent("Just ignore this message", "text/plain");
        mimeBodyPart.attachFile(attachment);

        var multipart = new MimeMultipart();
        multipart.addBodyPart(mimeBodyPart);

        message.setContent(multipart);

        Transport.send(message);
    } else {
        System.out.println("Error occured");
    }
}
```



