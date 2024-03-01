# How to Scrape Amazon for Data and How to Solve Amazon Captcha?


In this article, we will guide you through the process of setting up a simple Amazon scraper to extract data from the website. Additionally, we will address the challenge of solving Amazon CAPTCHAs, which are designed to prevent automated scraping.

To start, we will utilize Python for the purpose of this tutorial, but it's important to note that the concepts discussed can be applied to other programming languages as well.

## Installing the necessary libraries:
To begin, ensure that you have Python installed on your system. Next, install the following libraries using pip:
- Requests: Used to send HTTP requests to the Amazon website.
- BeautifulSoup: A library for parsing HTML and extracting data.
### Making requests to Amazon:
In order to scrape data from Amazon, we need to send HTTP requests to the website and retrieve the HTML content of the pages. We can use the Requests library to achieve this. Here's an example of making a request to retrieve the HTML of an Amazon product page: reviewing the data.

```python
import requests

url = "https://www.amazon.com/product-page-url"
response = requests.get(url)
html_content = response.text

# Now we have the HTML content of the page and can proceed with parsing and extracting data.
```
## Parsing the HTML with BeautifulSoup:
Once we have obtained the HTML content of a page, we can use BeautifulSoup to parse the HTML and extract the desired data. This could include product information, reviews, prices, and more. Here's an example of using BeautifulSoup to extract the title of a product from an Amazon page:

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(html_content, "html.parser")
title = soup.find("span", id="productTitle").text.strip()

# Now we have extracted the product title and can continue with further data extraction.
```
## Dealing with Amazon CAPTCHAs:
During scraping, it's common to encounter CAPTCHAs on Amazon, as the website employs them to prevent automated scraping. Amazon currently has 2 main scenarios for captcha, one is FunCaptcha and one is Imagetotext  
  

**Funcaptcha** is a type of CAPTCHA technology that was developed by a company called Arkose Labs. Unlike traditional CAPTCHAs, Funcaptcha uses interactive puzzles and games to differentiate between humans and bots. These puzzles are designed to be engaging and fun for humans, but difficult for bots to solve.

  
**Image-to-text**, also known as optical character recognition (OCR), is a technology that converts printed or handwritten text within an image into machine-readable text. It involves using algorithms and computer vision techniques to analyze the visual patterns and structures of characters in an image and translate them into editable and searchable text.



### Solving Amazon Funcaptcha

#### Create Task

Create a task with the [createTask](../api-createtask.md) to create a task.

#### Task Object Structure

| Properties               | Type   | Required | Description                                                                                                                                                                                                                                                                                                                |
|--------------------------|--------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| type                     | String | Required | `FunCaptchaTaskProxyLess`                                                                                                                                                                                                                                                                                                  |
| websiteURL               | String | Required | Web address of the website using funcaptcha, generally it's fixed value. (Ex: https://google.com)                                                                                                                                                                                                                          |
| websitePublicKey         | String | Required | The domain public key, rarely updated. (Ex: E8A75615-1CBA-5DFF-8031-D16BCF234E10)                                                                                                                                                                                                                                          |
| funcaptchaApiJSSubdomain | String | Optional | A special subdomain of [funcaptcha.com](http://funcaptcha.com/), from which the JS captcha widget should be loaded. Most FunCaptcha installations work from shared domains.                                                                                                                                                |
| data                     | String | Optional | Additional parameter that may be required by FunCaptcha implementation. Use this property to send "blob" value as a stringified array. See example how it may look like. {"\blob\":\"HERE_COMES_THE_blob_VALUE\"}  Learn [how to get FunCaptcha blob data](https://www.capsolver.com/blog/FunCaptcha/funcaptcha-data-blob) |
| proxy                    | String | Optional | Learn [Using proxies](../api-how-to-use-proxy)                                                                                                                                                                                                                                                                             |

#### Example Request

``` json
POST https://api.capsolver.com/createTask
Host: api.capsolver.com
Content-Type: application/json

{
    "clientKey": "YOUR_API_KEY_HERE",
    "task": {
        "type":"FunCaptchaTaskProxyLess", //Required
        "websiteURL":"", //Required
        "websitePublicKey":"", //Required
        "data": "{\"blob\": \"flaR60YY3tnRXv6w.l32U2KgdgEUCbyoSPI4jOxU...\"}" // Optional
    }
}
```


After you submit the task to us, you should receive in the response a 'Task id' if it's successfull. Please
read [errorCode: full list of errors](https://captchaai.atlassian.net/wiki/spaces/CAPTCHAAI/pages/394062/FuncaptchaTask+solving+FunCaptcha#)
if you didn't receive the task id.

#### Example Response

``` json
{
    "errorId": 0,
    "status": "idle",
    "taskId": "61138bb6-19fb-11ec-a9c8-0242ac110006"
}

```

#### **Getting Result**

Use the [getTaskResult](../api-gettaskresult.md) method to get the recognition results

Depending on the system load, you will get the results within the interval of `1s` to `20s`

#### Example Request

``` json
POST https://api.capsolver.com/getTaskResult
Host: api.capsolver.com
Content-Type: application/json

{
    "clientKey": "YOUR_API_KEY",
    "taskId": "61138bb6-19fb-11ec-a9c8-0242ac110006"
}
```

#### Example Response

``` json
{
    "errorId": 0,
    "solution": {
        "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
        "token": "3AHJ_q25SxXT-pmSeBXjzScW-EiocHwwpwqtk1QXlJnGnU......"
    },
    "status": "ready"
}
```


### Solving Amazon Imagetotext



:::

#### Create Task

Create the task with the [createTask](../api-createtask.md).

**Task Object Structure**

Note that this type of task returns the task execution result directly after createTask, rather than getting it
asynchronously through getTaskResult.

| Properties | Type    | Required | Description                                                                                            |
|------------|---------|----------|--------------------------------------------------------------------------------------------------------|
| type       | String  | Required | ImageToTextTask                                                                                        |
| websiteURL | String  | Optional | Page source url to improve accuracy                                                                    |
| body       | String  | Required | base64 encoded content of the image (no newlines) (no data:image/****\*****; base64, content           |
| module     | String  | Optional | Specifies the module. Currently, the supported modules are common and queueit                          |
| score      | Float   | Optional | `0.8 ~ 1`, Identify the matching degree. If the recognition rate is not within the range, no deduction |
| case       | Boolean | Optional | Case sensitive or not                                                                                  |

#### Example Request

```text
POST https://api.capsolver.com/createTask
Host: api.capsolver.com
Content-Type: application/json
```

```json lines
{
  "clientKey": "YOUR_API_KEY",
  "task": {
    "type": "ImageToTextTask",
    "websiteURL": "https://xxxx.com",
    // You can choose the module you need to use
    // ocr single image model, default common
    "module": "queueit",
    // base64 encoded image
    "body": "/9j/4AAQSkZJRgABA......"
  }
}
```

#### Example Response

```json lines
{
  "errorId": 0,
  "errorCode": "",
  "errorDescription": "",
  "status": "ready",
  "solution": {
    "text": "44795sds"
  },
  "taskId": "2376919c-1863-11ec-a012-94e6f7355a0b"
}
```
