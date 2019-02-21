# How to convert HTML to PDF in Go

## Documentation

See the full documentation on [PDFShift's documentation](https://docs.pdfshift.io/).

## Installation

Have the latest version of Go installed
### Requirements

* Go

## Usage

Your HTTP requests needs to be configured with your `api_key` received when creating an account.
Setting it is easy as:

```go
package main

import (
	"net/http"
	"log"
	"encoding/json"
	"bytes"
	"io/ioutil"
)

func main() {
	API_KEY := "put your api key here"

	message := map[string]interface{}{
		"source": "https://example.com",
		"sandbox": true,
	}

	bytesRepresentation, err := json.Marshal(message)
	if err != nil {
		log.Fatalln(err)
	}

	client := http.Client{}
	request, err := http.NewRequest("POST", "https://api.pdfshift.io/v2/convert", bytes.NewBuffer(bytesRepresentation))
	if err != nil {
		log.Fatalln(err)
	}
	request.Header.Set("Content-Type", "application/json")
	request.Header.Set("Authorization", "Basic " + API_KEY)

	resp, err := client.Do(request)
	if err != nil {
		log.Fatalln(err)
	}

	if resp.StatusCode >= 200 && resp.StatusCode < 300 {
		body, err := ioutil.ReadAll(resp.Body)
		if err != nil {
			log.Fatalln(err)
		}
		// write the response to file
		ioutil.WriteFile("example.pdf", body, 0644)
	} else {
        // An error occurred
		var result map[string]interface{}

		json.NewDecoder(resp.Body).Decode(&result)

		log.Println(result)
		log.Println(result["data"])
	}
}
```

We also highly recommend checking for errors after the conversion is made, before processing the document, in order to avoid issues later on.

```go
resp, err := client.Do(request)
if err != nil {
    log.Fatalln(err)
}

if resp.StatusCode >= 200 && resp.StatusCode < 300 {
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        log.Fatalln(err)
    }
    // write the response to file
    ioutil.WriteFile("example.pdf", body, 0644)
} else {
    // An error occurred
    var result map[string]interface{}

    json.NewDecoder(resp.Body).Decode(&result)

    log.Println(result)
    log.Println(result["data"])
}
```

Please note that `err` would be reported only if there was an issue connecting to the server and getting a response back. However, it would not be concerned about what http status code the server sent. For example, if the server sends a http 500 (which is internal server error), you will get that status code and error message on the resp object, not on err

From here on, the whole code will not be shared, just new additions and points of interest. But to run the code you will need to write a complete program as above with the added changes.

The `sandbox` parameter allows you to do unlimited conversion, but will add a watermark on top of the generated document.
No credits are deduced from your account when the sandbox mode is on.

### With a URL

Converting an URL with PDFShift is really easy. All you have to do is send a POST request with the `source` parameter set to the URL, like the following:

```go
message := map[string]interface{}{
    "source": "https://example.com"
}
```

### With inline HTML data:

To convert a raw HTML data with PDFShift, simply send the raw string in the `source` parameter:


```go
message := map[string]interface{}{
    "source": "<html><head><title>Hello world</title><body><h1>Hello World</h1></body></head></html>"
}
```

### Save the file to Amazon S3 and get an URL instead

By passing the `filename` parameter to your request, you will receive a JSON response instead of the binary PDF, with a `url` key that contains the path to the file stored on Amazon S3.
All files stored on Amazon S3 are kept for two days, then automatically deleted.

```go
package main

import (
	"net/http"
	"log"
	"encoding/json"
	"bytes"
	
)

func main() {
	message := map[string]interface{}{
		"source": "<html><head><title>Hello world</title><body><h1>Hello World</h1></body></head></html>",
		"filename": "anotherExample.pdf",
	}

	bytesRepresentation, err := json.Marshal(message)
	if err != nil {
		log.Fatalln(err)
	}

	client := http.Client{}
	request, err := http.NewRequest("POST", "https://api.pdfshift.io/v2/convert", bytes.NewBuffer(bytesRepresentation))
	if err != nil {
		log.Fatalln(err)
	}
	request.Header.Set("Content-Type", "application/json")

	resp, err := client.Do(request)
	if err != nil {
		log.Fatalln(err)
	}

	if resp.StatusCode >= 200 && resp.StatusCode < 300 {
		var result map[string]interface{}

		json.NewDecoder(resp.Body).Decode(&result)

		log.Println(result["url"])
	} else {
        // An error occurred
		var result map[string]interface{}

		json.NewDecoder(resp.Body).Decode(&result)

		log.Println(result)
	}
}
```

### Accessing secured pages

If your `source` requires a BASIC AUTH mechanism, you can either use the custom headers part or use the `auth` parameter from the API that behaves the same.

```go
package main

import (
	"bytes"
	"encoding/json"
	"io/ioutil"
	"log"
	"net/http"
)

func main() {
	message := map[string]interface{}{
		"source": "https://httpbin.org/basic-auth/user/passwd",
		"auth": map[string]string{
			"username": "user",
			"password": "passwd",
		},
	}

	bytesRepresentation, err := json.Marshal(message)
	if err != nil {
		log.Fatalln(err)
	}

	client := http.Client{}
	request, err := http.NewRequest("POST", "https://api.pdfshift.io/v2/convert", bytes.NewBuffer(bytesRepresentation))
	if err != nil {
		log.Fatalln(err)
	}
	request.Header.Set("Content-Type", "application/json")

	resp, err := client.Do(request)
	if err != nil {
		log.Fatalln(err)
	}

	if resp.StatusCode >= 200 && resp.StatusCode < 300 {
		body, err := ioutil.ReadAll(resp.Body)
		if err != nil {
			log.Fatalln(err)
		}
		// write the response to file
		ioutil.WriteFile("example.pdf", body, 0644)
	} else {
		var result map[string]interface{}

		json.NewDecoder(resp.Body).Decode(&result)

		log.Println(result)
	}
}
```

### Using cookies

Cookies might help you access unauthorized areas that aren't restricted by a simple Basic Auth mechanism. You can define as many cookies as you want.


```go
package main

import (
	"net/http"
	"log"
	"encoding/json"
	"bytes"
	"io/ioutil"
	
)

func main() {
	cookies := make([]map[string]string, 1)

	cookies[0] = make(map[string]string)
	cookies[0]["name"] = "session"
	cookies[0]["value"] = "4cb496a8-a3eb-4a7e-a704-f993cb6a4dac"

	message := map[string]interface{}{
		"source": "<html><head><title>Hello world</title><body><h1>Hello World</h1></body></head></html>",
		"cookies": cookies,
	}

	bytesRepresentation, err := json.Marshal(message)
	if err != nil {
		log.Fatalln(err)
	}

	client := http.Client{}
	request, err := http.NewRequest("POST", "https://api.pdfshift.io/v2/convert", bytes.NewBuffer(bytesRepresentation))
	if err != nil {
		log.Fatalln(err)
	}
	request.Header.Set("Content-Type", "application/json")

	resp, err := client.Do(request)
	if err != nil {
		log.Fatalln(err)
	}

	if resp.StatusCode >= 200 && resp.StatusCode < 300 {
		body, err := ioutil.ReadAll(resp.Body)
		if err != nil {
			log.Fatalln(err)
		}
		// write the response to file
		ioutil.WriteFile("example.pdf", body, 0644)
	} else {
		var result map[string]interface{}

		json.NewDecoder(resp.Body).Decode(&result)

		log.Println(result)
	}
}
```


### Loading CSS from an URL:

By passing a `css` parameter, you will be able to modify the page with your CSS.
This allows you to customize the rendering of the page.

You can also call multiple CSS by calling a root CSS (like "print.css" in that case) that will call @import in it for each CSS files.

```go
message := map[string]interface{}{
    "source": "https://www.example.com",
    "css": "a {text-decoration: underline; color: blue}",
}
```


### Loading CSS from a string:

Like for the `source` parameter, you can pass a raw set of CSS rules to the `css` parameter and they will be injected to the loaded document.

```go
message := map[string]interface{}{
    "source": "https://www.example.com",
    "css": "a {text-decoration: underline; color: blue}",
}
```

### Adding Watermark

Some documents that you share need a watermark to clearly identify your brand. That's easy with PDFShift:

```go
message := map[string]interface{}{
    "source": "https://www.example.com",
    "watermark": map[string]string{
        "image": "https://pdfshift.io/static/img/logo.png",
        "offset_x": 50,
        "offset_y": "100px",
        "rotate": 45,
    },
}
```

### Custom Header (or Footer)

You can add some custom header or footer to your generated document. These are often used to indicate the current page, or show the logo of your company on every page.

Note that the header and footer **are not related to the body**. For this reason, the CSS in your body doesn't apply to your header/footer.
By default, the font-size will be really small. You will have to set it manually, like in the following example:

```go
message := map[string]interface{}{
    "source": "https://www.example.com",
    "footer": map[string]string{
        "source": "<div>My custom footer</div>",
        "spacing": "50px",
    },
}
```

### Protecting the generated PDF

Protecting your document is easy with PDFShift. You can specify a password for the user and for the owner.
(The owner will have full rights access while the user will have limited access based on your choice).

Please keep in mind that some PDF reader doesn't respect the rights as long as the user is authenticated.
This means that if you set an empty password for the user, with no rights to print or copy, some PDF reader will ignore this and still allow printing and copying.

This is outside of our capabilities here at PDFShift as we can't enforce a reader to respect PDF's standard.

```go
message := map[string]interface{}{
    "source": "https://www.example.com",
    "protection": map[string]interface{}{
        "user_password": "user",
        "owner_password": "owner",
        "no_print": true,
    },
}
```


### Real life example - Sending an invoice by email

One frequent use of PDFShift is to generate an invoice when receiving a payment, and sending that invoice - in PDF format - by email, to the customer.





