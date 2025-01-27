# Web Scraping With AIOHTTP in Python

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/) 

This guides explains the basics of using AIOHTTP in Python for web scraping.

- [What Is AIOHTTP?](#what-is-aiohttp)
- [Scraping with AIOHTTP: Step-By-Step Tutorial](#scraping-with-aiohttp-step-by-step-tutorial)
  - [Step #1: Setting Up a Scraping Project](#step-1-setting-up-a-scraping-project)
  - [Step #2: Setting Up the Scraping Libraries](#step-2-setting-up-the-scraping-libraries)
  - [Step #3: Getting the HTML of the Target Page](#step-3-getting-the-html-of-the-target-page)
  - [Step #4: Parsing the HTML](#step-4-parsing-the-html)
  - [Step #5: Writing the Data Extraction Logic](#step-5-writing-the-data-extraction-logic)
  - [Step #6: Exporting the Scraped Data](#step-6-exporting-the-scraped-data)
  - [Step #7: Putting It All Together](#step-7-putting-it-all-together)
- [AIOHTTP for Web Scraping: Advanced Features and Techniques](#aiohttp-for-web-scraping-advanced-features-and-techniques)
  - [Setting Custom Headers](#setting-custom-headers)
  - [Setting a Custom User Agent](#setting-a-custom-user-agent)
  - [Setting Cookies](#setting-cookies)
  - [Proxy Integration](#proxy-integration)
  - [Error Handling](#error-handling)
  - [Retrying Failed Requests](#retrying-failed-requests)
- [AIOHTTP vs Requests for Web Scraping](#aiohttp-vs-requests-for-web-scraping)
- [Conclusion](#conclusion)

## What Is AIOHTTP?

[AIOHTTP](https://docs.aiohttp.org/en/stable/) is an asynchronous client/server HTTP framework built on Python’s [`asyncio`](https://docs.python.org/3/library/asyncio.html) library. Unlike traditional HTTP clients, AIOHTTP uses client sessions to manage connections across multiple requests, making it a highly efficient choice for high-concurrency, session-based tasks.


**⚙️ Features**

- Supports both client and server implementations of the HTTP protocol.  
- Natively supports WebSockets for both client and server.  
- Provides middleware and pluggable routing for building web servers.  
- Efficiently manages streaming of large data.  
- Includes client session persistence, allowing connection reuse and minimizing overhead for multiple requests.  


## Scraping with AIOHTTP: Step-By-Step Tutorial

In the context of web scraping, AIOHTTP is just an HTTP client to fetch the raw HTML content of a page. To parse and extract data from that HTML, you need an HTML parser like [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/).

> **Warning**:\
> Although AIOHTTP is mainly utilized in the initial stages of the process, this guide walks you through the entire scraping workflow. If you're looking for more advanced AIOHTTP web scraping techniques, you can skip ahead to the next chapter after completing Step 3.

### Step #1: Setting Up a Scraping Project

Install Python3+ and create a directory for your AIOHTTP scraping project:

```bash
mkdir aiohttp-scraper
```

Navigate into that directory and set up a [virtual environment](https://docs.python.org/3/library/venv.html):

```bash
cd aiohttp-scraper
python -m venv env
```

Open the project folder in your preferred Python IDE and create a file named `scraper.py` within the project folder.


In your IDE’s terminal, activate the virtual environment. On Linux or macOS, use:

```bash
./env/bin/activate
```

On Windows, run:

```powershell
env/Scripts/activate
```

### Step #2: Setting Up the Scraping Libraries

Install AIOHTTP and BeautifulSoup:

```bash
pip install aiohttp beautifulsoup4
```

Import the installed the [`aiohttp`](https://docs.aiohttp.org/en/stable/) and [`beautifulsoup4`](https://pypi.org/project/beautifulsoup4/) dependencies into your `scraper.py` script:

```python
import asyncio
import aiohttp 
from bs4 import BeautifulSoup
```

> **Note**:\
> `aiohttp` requires the `asyncio` to work.

Now, add the following `async` function workflow to your `scrper.py` file:

```python
async def scrape_quotes():
    # Scraping logic...

# Run the asynchronous function
asyncio.run(scrape_quotes())
```

`scrape_quotes()` defines an asynchronous function where your scraping logic will run concurrently without blocking. Finally, `asyncio.run(scrape_quotes())` starts and runs the asynchronous function.

### Step #3: Getting the HTML of the Target Page

This example explains how to scrape data from the [“Quotes to Scrape”](https://quotes.toscrape.com/) site:

![The target site](https://github.com/luminati-io/aiohttp-web-scraping/blob/main/Images/s_C07E0B72CB9153F9B6E6EF6B76FDCD439C9910ACC1C4E94E70E103EE716CD2E2_1737465124750_image.png)

With libraries like Requests or AIOHTTP, making a GET request directly retrieves the HTML content of the page. However, AIOHTTP operates using a [different request lifecycle](https://docs.aiohttp.org/en/stable/http_request_lifecycle.html).  

The AIOHTTP primary component is the [`ClientSession`](https://docs.aiohttp.org/en/stable/client_reference.html), which manages a pool of connections and supports [`Keep-Alive`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Keep-Alive) by default. Rather than opening a new connection for each request, it reuses existing connections, enhancing performance.

The process of making a request generally involves three key steps:

1. Opening a session through `ClientSession()`.
2. Sending the GET request asynchronously with [`session.get()`](https://docs.aiohttp.org/en/stable/client_reference.html#aiohttp.ClientSession.get).
3. Accessing the response data with methods like `await response.text()`.

This design allows the event loop to use different [`with` contexts](https://docs.python.org/3/reference/datamodel.html#context-managers) between operations without blocking, making it ideal for high-concurrency tasks.

With that in mind, you can use AIOHTTP to fetch the homepage's HTML using the following approach:

```python
async with aiohttp.ClientSession() as session:
    async with session.get("http://quotes.toscrape.com") as response:
        # Access the HTML of the target page
        html = await response.text()
```

Behind the scenes, AIOHTTP handles sending the request to the server and waits for the server's response, which includes the page's HTML content. After receiving the response, the `await response.text()` method retrieves the HTML content as a string.

Print the `html` variable and you will see:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Quotes to Scrape</title>
    <link rel="stylesheet" href="/static/bootstrap.min.css">
    <link rel="stylesheet" href="/static/main.css">
</head>
<body>
    <!-- omitted for brevity... -->
</body>
</html>
```

### Step #4: Parsing the HTML

Parse the HTML content by passing it to the BeautifulSoup constructor:

```python
# Parse the HTML content using BeautifulSoup
soup = BeautifulSoup(html, "html.parser")
```

[`html.parser`](https://docs.python.org/3/library/html.parser.html) is the default Python HTML parser used to process the content.

The `soup` object contains the parsed HTML and offers methods to extract the required data.  

### Step #5: Writing the Data Extraction Logic

The following code can be used to scrape the quotes data from the page:

```python
# Where to store the scraped data
quotes = []

# Extract all quotes from the page
quote_elements = soup.find_all("div", class_="quote")

# Loop through quotes and extract text, author, and tags
for quote_element in quote_elements:
    text = quote_element.find("span", class_="text").get_text().get_text().replace("“", "").replace("”", "")
    author = quote_element.find("small", class_="author")
    tags = [tag.get_text() for tag in quote_element.find_all("a", class_="tag")]

    # Store the scraped data
    quotes.append({
        "text": text,
        "author": author,
        "tags": tags
    })
```

This code snippet initializes a list called `quotes` to store the scraped data. It locates all the quote HTML elements and iterates through them to extract details such as the quote text, author, and tags. Each extracted quote is stored as a dictionary in the `quotes` list, organizing the data for easy access or export.

### Step #6: Exporting the Scraped Data

You can use the following code to export the scraped data to a CSV file:

```python
# Open the file for export
with open("quotes.csv", mode="w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=["text", "author", "tags"])
    
    # Write the header row
    writer.writeheader()
    
    # Write the scraped quotes data
    writer.writerows(quotes)
```

The above snippet opens a file named `quotes.csv` in write mode. Then it, sets up column headers (`text`, `author`, `tags`), writes the headers, and then writes each dictionary from the `quotes` list to the CSV file.

[`csv.DictWriter`](https://docs.python.org/3/library/csv.html#csv.DictWriter) simplifies data formatting, making it easier to store structured data. To make it work, import `csv` from the Python Standard Library:

```python
import csv
```

### Step #7: Putting It All Together

Here’s the complete AIOHTTP web scraping script:

```python
import asyncio
import aiohttp
from bs4 import BeautifulSoup
import csv

# Define an asynchronous function to make the HTTP GET request
async def scrape_quotes():
    async with aiohttp.ClientSession() as session:
        async with session.get("http://quotes.toscrape.com") as response:
            # Access the HTML of the target page
            html = await response.text()

            # Parse the HTML content using BeautifulSoup
            soup = BeautifulSoup(html, "html.parser")

            # List to store the scraped data
            quotes = []

            # Extract all quotes from the page
            quote_elements = soup.find_all("div", class_="quote")

            # Loop through quotes and extract text, author, and tags
            for quote_element in quote_elements:
                text = quote_element.find("span", class_="text").get_text().replace("“", "").replace("”", "")
                author = quote_element.find("small", class_="author").get_text()
                tags = [tag.get_text() for tag in quote_element.find_all("a", class_="tag")]

                # Store the scraped data
                quotes.append({
                    "text": text,
                    "author": author,
                    "tags": tags
                })

            # Open the file name for export
            with open("quotes.csv", mode="w", newline="", encoding="utf-8") as file:
                writer = csv.DictWriter(file, fieldnames=["text", "author", "tags"])

                # Write the header row
                writer.writeheader()

                # Write the scraped quotes data
                writer.writerows(quotes)

# Run the asynchronous function
asyncio.run(scrape_quotes())
```

You can run it with:

```bash
python scraper.py
```

Or, on Linux/macOS:

```bash
python3 scraper.py
```

A `quotes.csv` file will appear in the root folder of your project. Open it and you will see:

![The final quotes file](https://github.com/luminati-io/aiohttp-web-scraping/blob/main/Images/s_C07E0B72CB9153F9B6E6EF6B76FDCD439C9910ACC1C4E94E70E103EE716CD2E2_1737466185816_image.png)

## AIOHTTP for Web Scraping: Advanced Features and Techniques

In the following examples, the target site will be the [HTTPBin.io `/anything` endpoint](https://httpbin.io/anything). This API returns the IP address, headers, and other data sent by the requester.

### Setting Custom Headers

You can [specify custom headers](https://docs.aiohttp.org/en/stable/client_advanced.html#custom-request-headers) in an AIOHTTP request with the `headers` argument:

```python
import aiohttp
import asyncio

async def fetch_with_custom_headers():
    # Custom headers for the request
    headers = {
        "Accept": "application/json",
        "Accept-Language": "en-US,en;q=0.9,fr-FR;q=0.8,fr;q=0.7,es-US;q=0.6,es;q=0.5,it-IT;q=0.4,it;q=0.3"
    }

    async with aiohttp.ClientSession() as session:
        # Make a GET request with custom headers
        async with session.get("https://httpbin.io/anything", headers=headers) as response:
            data = await response.json()
            # Handle the response...
            print(data)

# Run the event loop
asyncio.run(fetch_with_custom_headers())
```

This way, AIOHTTP will make a GET HTTP request with the [`Accept`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) and [`Accept-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language) headers set.

### Setting a Custom User Agent

[`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) is one of the most critical [HTTP headers for web scraping](https://brightdata.com/blog/web-data/http-headers-for-web-scraping). By default, AIOHTTP uses this `User-Agent`:

```
Python/<PYTHON_VERSION> aiohttp/<AIOHTTP_VERSION>
```

The default value mentioned above can make your requests easily identifiable as coming from an automated script, increasing the likelihood of being blocked by the target site.

To reduce the chances of getting detected, you can set a custom real-world `User-Agent` as before:

```python
import aiohttp
import asyncio

async def fetch_with_custom_user_agent():
    # Define a Chrome-like custom User-Agent
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/132.0.0.0 Safari/537.36"
    }

    async with aiohttp.ClientSession(headers=headers) as session:
        # Make a GET request with the custom User-Agent
        async with session.get("https://httpbin.io/anything") as response:
            data = await response.text()
            # Handle the response...
            print(data)

# Run the event loop
asyncio.run(fetch_with_custom_user_agent())
```

### Setting Cookies

Just like HTTP headers, you can [set custom cookies](https://docs.aiohttp.org/en/v3.7.3/client_advanced.html#custom-cookies) using the `cookies` in `ClientSession()`:

```python
import aiohttp
import asyncio

async def fetch_with_custom_cookies():
    # Define cookies as a dictionary
    cookies = {
        "session_id": "9412d7hdsa16hbda4347dagb",
        "user_preferences": "dark_mode=false"
    }

    async with aiohttp.ClientSession(cookies=cookies) as session:
        # Make a GET request with custom cookies
        async with session.get("https://httpbin.io/anything") as response:
            data = await response.text()
            # Handle the response...
            print(data)

# Run the event loop
asyncio.run(fetch_with_custom_cookies())
```

Cookies allow you to include session data essential for your web scraping requests.

> **Note**:\
> Cookies set in `ClientSession` are shared across all requests made with that session. To access session cookies, refer to [`ClientSession.cookie_jar`](https://docs.aiohttp.org/en/v3.7.3/client_reference.html#aiohttp.ClientSession.cookie_jar).

### Proxy Integration

In AIOHTTP, you can route your requests through a proxy server to reduce the risk of IP bans. Do that by using the [`proxy` argument](https://docs.aiohttp.org/en/v3.7.3/client_advanced.html#proxy-support) in the HTTP method function on `session`:

```python
import aiohttp
import asyncio

async def fetch_through_proxy():
    # Replace with the URL of your proxy server
    proxy_url = "<YOUR_PROXY_URL>"

    async with aiohttp.ClientSession() as session:
        # Make a GET request through the proxy server
        async with session.get("https://httpbin.io/anything", proxy=proxy_url) as response:
            data = await response.text()
            # Handle the response...
            print(data)

# Run the event loop
asyncio.run(fetch_through_proxy())
```

### Error Handling

By default, AIOHTTP raises errors only for connection or network issues. To raise exceptions for HTTP responses when receiving [`4xx`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#client_error_responses) and [`5xx`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#server_error_responses) status codes, you can use any of the following approaches:

1. **Set `raise_for_status=True` when creating the `ClientSession`**: Automatically raise exceptions for all requests made through the session if the response status is `4xx` or `5xx`.
2. **Pass `raise_for_status=True` directly to request methods**: Enable error raising for individual request methods (like `session.get()` or `session.post()`) without affecting others.
3. **Call `response.raise_for_status()` manually**: Give full control over when to raise exceptions, allowing you to decide on a per-request basis.

Option #1 example:

```python
import aiohttp
import asyncio

async def fetch_with_session_error_handling():
    async with aiohttp.ClientSession(raise_for_status=True) as session:
        try:
            async with session.get("https://httpbin.io/anything") as response:
                # No need to call response.raise_for_status(), as it is automatic
                data = await response.text()
                print(data)
        except aiohttp.ClientResponseError as e:
            print(f"HTTP error occurred: {e.status} - {e.message}")
        except aiohttp.ClientError as e:
            print(f"Request error occurred: {e}")

# Run the event loop
asyncio.run(fetch_with_session_error_handling())
```

When `raise_for_status=True` is set at the session level, all requests made through that session will raise an `aiohttp.ClientResponseError` for `4xx` or `5xx` responses.

Option #2 example:

```python
import aiohttp
import asyncio

async def fetch_with_raise_for_status():
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get("https://httpbin.io/anything", raise_for_status=True) as response:
                # No need to manually call response.raise_for_status(), it is automatic
                data = await response.text()
                print(data)
        except aiohttp.ClientResponseError as e:
            print(f"HTTP error occurred: {e.status} - {e.message}")
        except aiohttp.ClientError as e:
            print(f"Request error occurred: {e}")

# Run the event loop
asyncio.run(fetch_with_raise_for_status())
```

In this case, the `raise_for_status=True` argument is passed directly to the `session.get()` call. This ensures that an exception is raised automatically for any `4xx` or `5xx` status codes.

Option #3 example:

```python
import aiohttp
import asyncio

async def fetch_with_manual_error_handling():
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get("https://httpbin.io/anything") as response:
                response.raise_for_status()  # Manually raises error for 4xx/5xx
                data = await response.text()
                print(data)
        except aiohttp.ClientResponseError as e:
            print(f"HTTP error occurred: {e.status} - {e.message}")
        except aiohttp.ClientError as e:
            print(f"Request error occurred: {e}")

# Run the event loop
asyncio.run(fetch_with_manual_error_handling())
```

If you prefer greater control over individual requests, you can manually call `response.raise_for_status()` after making a request. This approach lets you determine the precise moment to handle errors.  


### Retrying Failed Requests

AIOHTTP does not provide built-in support for retrying requests automatically. To implement that, you must use custom logic or a third-party library like [`aiohttp-retry`](https://github.com/inyutin/aiohttp_retry). This enables you to configure retry logic for failed requests, helping to handle transient network issues, timeouts, or rate limits.

Install [`aiohttp-retry`](https://pypi.org/project/aiohttp-retry/):

```bash
pip install aiohttp-retry
```

Use it in the code:

```python
import asyncio
from aiohttp_retry import RetryClient, ExponentialRetry

async def main():
    retry_options = ExponentialRetry(attempts=1)
    retry_client = RetryClient(raise_for_status=False, retry_options=retry_options)
    async with retry_client.get("https://httpbin.io/anything") as response:
        print(response.status)
        
    await retry_client.close()
```

This configures retry behavior, with an exponential backoff strategy. Learn more in the [official docs](https://github.com/inyutin/aiohttp_retry?tab=readme-ov-file#documentation).

## AIOHTTP vs Requests for Web Scraping

Below is a summary table to compare AIOHTTP and [Requests for web scraping](https://brightdata.com/blog/web-data/python-requests-guide):

| **Feature** | **AIOHTTP** | **Requests** |
| --- | --- | --- |
| **GitHub stars** | 15.3k | 52.4k |
| **Client support** | ✔️  | ✔️  |
| **Sync support** | ❌   | ✔️  |
| **Async support** | ✔️  | ❌   |
| **Server support** | ✔️  | ❌   |
| **Connection pooling** | ✔️  | ✔️  |
| **HTTP/2 support** | ❌   | ❌   |
| **User-agent customization** | ✔️  | ✔️  |
| **Proxy support** | ✔️  | ✔️  |
| **Cookie handling** | ✔️  | ✔️  |
| **Retry mechanism** | Available only via a third-party library | Available via `HTTPAdapter`s |
| **Performance** | High | Medium |
| **Community support and popularity** | Medium | Large |

For a complete comparison, check out our blog post on [Requests vs HTTPX vs AIOHTTP](https://brightdata.com/blog/web-data/requests-vs-httpx-vs-aiohttp).

## Conclusion

AIOHTTP is a fast and reliable tool for making HTTP requests to gather online data. However, automated HTTP requests can expose your public IP address. To protect your privacy and security, consider using Bright Data's proxy servers to mask your IP address.

- [Datacenter proxies](https://brightdata.com/proxy-types/datacenter-proxies) – Over 770,000 datacenter IPs.
- [Residential proxies](https://brightdata.com/proxy-types/residential-proxies) – Over 72M residential IPs in more than 195 countries.
- [ISP proxies](https://brightdata.com/proxy-types/isp-proxies) – Over 700,000 ISP IPs.
- [Mobile proxies](https://brightdata.com/proxy-types/mobile-proxies) – Over 7M mobile IPs.

Create a free Bright Data account today to test our proxies and scraping solutions!
