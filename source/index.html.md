---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - python

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>


search: true
---

# Introduction

Welcome to the Factmata Moderation API!

The API provides access to Factmata's Moderation - a risk scoring system. The system helps publishers and brands ensure they associate with safe inventory, and UGC platforms and publisher networks to flag more suspicious content for trust and safety teams. It enables avoiding unwanted ad placements, fines, user complaints, reputation damage and more. 

The system provides provides a risk score based on a number of fine-grained models.


## Models
Id | Name | Description |
-- | ---| -----| -
8 |Political Bias | Detects strongly biased political language |
7 |Hate speech | Detects demeaning and abusive language based on people's group identity |
14 |Sexism | Detects demeaning and abusive language based on people's gender identity|
11 |Racism |Detects demeaning and abusive language targeted towards a particular ethnicity| 
20 |Toxicity | Detects language which insults, threatens, attacks any individual or group with the purpose of humiliating, degrading or excluding that person or group |
22 |Obscenity | Detects obscene and profane language | 
17 |Insult | Detects scornful remarks directed towards an individual |
18 |Threat | Detects a wish or intention for pain, injury, or violence against an individual or group | 
10 |Clickbait | Detects headlines which at the expense of being informative, are designed to entice readers into clicking the accompanying link| 


Submitted content is scored by each of the models. The API does not allow to score content on just a selection of the available models.

Model's scores range from 0.00 to 1.00 and represent model's confidence. The higher the score the bigger probability of the content being risky. The scores are not directly comparable between the individual models, i.e. 0.75 for hate speech does't have the same significance as 0.75 for clickbait. 

Besides providing a score per each of the models, the API also returns a combined risk score. The score represents an overall risk level of the scored content and consists of a weighted average of all the individual scores.


## Content type
The system scores any type of textual content on the Internet: articles, blog posts, forum comments, etc. It works for text only and exclusively for English-language content (see the section 'Coming next' for other languages). 

The API enables:
- scoring URLs: provides risk score for an individual URL, e.g. https://www.bbc.co.uk/news/92170379
- scoring domains: provides risk score for an entire domain, e.g. https://www.bbc.co.uk


## Volume and latency
The API can support up to 20,000 URLs a day. 
One URL takes between 5 seconds and 1.5 minute to be scored. The time depends on whether the text scraped from the URL is already in our DB or requires scraping, which take 20-60 seconds. 
For domain scoring, one domain takes on average 20 minutes to be scored. This time varies depending on how many URLs are available under a domain.

# Authorization

Factmata API is authorized using API Key. If you don't have a key please contact info@factmata.com to get one set up.

The API_KEY is accepted as a request header `x-api-key`.

Do not share you API Key in publicly accessible platforms.

# Scoring URLs

Scoring URLs using Factmata API works in two steps:

## Submitting URL for scoring
   
    First, a URL needs to be submitted so the content can be scraped. The URL is scored by all the models in the Moderation pipeline once the content has been scraped.
    

## Fetching the scores

    After the URL has been submitted, the results will be available in some time.

    URL Scoring is completed when the text has been scored by all the models.

    During the time between submission and fetching, URL might already have 
    been scored by some models, while some are still being processed. In such case,
    the user has the option to fetch partial scores.


## Submit an URL for scoring

To score a URL, it first needs to be submitted.

```python
import requests

url = "https://api.factmata.com/api/v0.1/score/url"

data = {
  'url': 'www.example.com/some-page'
}
headers = {
  'x-api-key': API_KEY
}
res = requests.post(url, headers=headers, json=data)
```

```shell
curl 'https://api.factmata.com/api/v0.1/score/url?url=https://example.com/page' \
  -X POST \
  -H "Content-Type: application/json" \
  -H "x-api-key: API_KEY" 
```

> When the URL is successfully submitted for processing, the following response is returned.

```json
STATUS: 202
```
```json
{"answer": "Request Sent Successfully"}
```

> If the URL is already being processed, the following response is returned.

```json
STATUS: 200
```
```
{ "answer": "URL is being processed" }
```

### HTTP Request

`POST https://api.factmata.com/api/v0.1/score/url/?url=www.example.com/some-page`

### Request Payload

Parameter | Description
--------- | -----------
url | A valid URL string. This parameter is `required`. In case the URL is malformed or absent, request is aborted with status `422`. If the URL has already been submitted for scoring, status `200` is returned.

<aside class="notice">
URL once submitted once, does not need to be submitted again.
</aside>

<aside class="notice">
Currently only English web-pages are scored.
</aside>

<aside class="warning">
URL must contain a path along with the domain/subdomain name. 
</aside>

## Fetch the scores of a URL

After a URL has been submitted for scoring, you can fetch the scores
using this API.


```python
import requests

url = "https://api.factmata.com/api/v0.1/score/url/"

data = {
  'url': 'www.example.com/some-page'
}

headers = {
  'x-api-key': API_KEY
}

res = requests.get(url, headers=headers, params=data)
```

```shell
curl 'https://api.factmata.com/api/v0.1/score/url?url=https://example.com/page' \
  -X GET \
  -H "Content-Type: application/json" \
  -H "x-api-key: API_KEY" 
```


> If the URL has finished processing, the JSON structured like this is returned:

```json
{
  "model_names_scores": [
    {
      "model": 7,
      "model_name": "hate",
      "score": 0.314
    },
    {
      "model": 8,
      "model_name": "hype",
      "score": 0.315
    },
    {
      "model": 10,
      "model_name": "bait",
      "score": 0.316
    },
    {
      "model": 11,
      "model_name": "racism",
      "score": 0.317
    },
    {
      "model": 14,
      "model_name": "sexism",
      "score": 0.317
    },
    {
      "model": 17,
      "model_name": "insult",
      "score": 0.318
    },
    {
      "model": 18,
      "model_name": "threat",
      "score": 0.319
    },
    {
      "model": 20,
      "model_name": "toxic",
      "score": 0.320
    },
    {
      "model": 22,
      "model_name": "obscene",
      "score": 0.320
    }
  ],
  "combined_score": 0.320
}
```

> If `partial_results` flag is set and the URL is still processing, the following response is returned

```json
{
  "model_names_scores": [
    {
      "model_name": "hate",
      "score": 0.314
    },
    ## At this stage only model_name 1 has finished processing
  ]
}
```

> If the `partial_results` flag is not set and the URL is still processing, the following response is returned 

```json
STATUS: 202
```
```json
{
  "answer": "Please try again later"
}
```

### HTTP Request

`GET https://api.factmata.com/api/v0.1/score/url/?url=www.example.com/some-page&partial_results=False`

Fetching the scores has the following use cases:

### URL is fully processed 

When the URL is fully processed, all the model_name scores, as well as the `combined score` are returned.

### URL is partially processed, `partial_results` is True

The case when GET request is made, but only some model_names have finished scoring the URL, the user can fetch the scores of the `finished` model_names using the `partial_results` query parameter.
When `partial_results` is set to `True`, scores from model_names that have finished processing are returned.

### URL is partially processed, `partial_results` is False

In this case, no scores are returned, and the API returns status code `202`
### URL Parameters

Parameter | Default | Description
--------- | ------- | -----------
url | None | A valid URL string. This parameter is `required`. In case the URL is malformed or absent, request is aborted with status `422`. If the URL has already been submitted for scoring, status `200` is returned.
partial_results | False | Optional Boolean flag if partial results are wanted

# Scoring domains

Scoring domains using Factmata API works in two steps:

## 1. Submitting domains for scoring
   
    Before domains can be scored, they need to be crawled, and the crawled urls need to be
    scraped, and then the urls are scored by our model_names. Hence, the domains 
    need to be submitted first.

## 2. Fetching the scores

    After the domains have been submitted, the results will be available in some time.

    Domain Scoring is complete when all of the urls have been scored by all model_names.

    During the time between submission and fetching, some domains might already have 
    been scored, while other are still processing. In this case,
    the user has the option to fetch the partial scores.


## Submit domains for scoring

To score domains, they first needs to be submitted.

```python
import requests

url = "https://api.factmata.com/api/v0.1/score/domain"

data = {
  'domain': [
    'www.example.com',
    'example.com',
  ],
  'threshold': 5,
  'crawl_number': 100
}
headers = {
  'x-api-key': API_KEY
}
res = requests.post(url, headers=headers, json=data)
```

```shell
curl 'https://api.factmata.com/api/v0.1/score/domain' \
  --data '{"domain": ["www.example.com", "example.com"], "threshold": 5, "crawl_number": 100}' \
  -X POST \
  -H "Content-Type: application/json" \
  -H "x-api-key: API_KEY" 
```

> When the domains are successfully submitted for processing, the following response is returned

```json
STATUS: 202
```
```json
{"answer": "request sent successfully, added 2 domains: example.com, www.example.com"}

```

> If the domains are already being processed, the following response is returned

```json
STATUS: 406
```
```json
{"answer": "domains are already processing"}
```
> If the domains are already successfully processed, the following response is returned

```json
STATUS: 406
```
```json
{"answer": "These domains are successfully processed, send GET"}
```

### HTTP Request

`POST https://api.factmata.com/api/v0.1/score/domain/

### Request Payload

Parameter | Required | Default | Description
--------- | -------- | ------- | -----------
domain | True | None | A valid domain string. In case any of the domains is malformed, request is aborted with status `422`. If all of the domains are already successfully processed in the last 90 days, status `406` is returned. If all of the domains are already processing, status `406` is returned.
threshold | False | 5 | The minimum number of crawled urls for the domains.
crawl_number | False | 100 | The desired number of crawled urls for the domains.
report | False | False | Flag to treat the endpoint as the GET endpoint (in order to send domains in body and not in a query string).

<aside class="notice">
Only new domains will be submitted from the domain list.
</aside>

<aside class="notice">
Domain is submitted once per 90 days, does not need to be submitted again in this period.
</aside>

<aside class="notice">
Currently only English web-pages are scored.
</aside>

<aside class="notice">
In case of a very big list of domains (100+), 504 status can be returned. 
It does not mean that the request failed.
</aside>

<aside class="warning">
A domain must not contain a protocol (http, https) or a subroute (example.com/page/1). 
</aside>

## Fetch the scores for domains

After domains have been submitted for scoring, you can fetch the scores.
using this API.


```python
import requests

url = "https://api.factmata.com/api/v0.1/score/domain/"

data = {
  'domain': [
    'www.example.com',
    'example.com',
  ],
}

headers = {
  'x-api-key': API_KEY
}

res = requests.get(url, headers=headers, params=data)
```

```shell
curl 'https://api.factmata.com/api/v0.1/score/domain/?domain='www.example.com'&domain='example.com'' \
  -H "x-api-key: API_KEY" 
```

> or you can use post with report flag for large amount of domains (not to hit query length limits)

```python
import requests

url = "https://api.factmata.com/api/v0.1/score/domain/"

data = {
  'domain': [
    'www.example.com',
    'example.com',
  ],
  'report': True
}

headers = {
  'x-api-key': API_KEY
}

res = requests.post(url, headers=headers, params=data)
```

```shell
curl 'https://api.factmata.com/api/v0.1/score/domain/' \
  --data '{"domain": ["www.example.com", "example.com"], "report": true}' \
  -X POST \
  -H "Content-Type: application/json" \
  -H "x-api-key: API_KEY" 
```

> If the domain has finished processing, the JSON structured like this is returned:

```json
{
  "domain": "example.com", 
  "status": "success",
  "score": {
    "model_names_scores": [
      {
        "model": 7,
        "model_name": "hate",
        "score": 0.314
      },
      {
        "model": 8,
        "model_name": "hype",
        "score": 0.315
      },
      {
        "model": 10,
        "model_name": "bait",
        "score": 0.316
      },
      {
        "model": 11,
        "model_name": "racism",
        "score": 0.317
      },
      {
        "model": 14,
        "model_name": "sexism",
        "score": 0.317
      },
      {
        "model": 17,
        "model_name": "insult",
        "score": 0.318
      },
      {
        "model": 18,
        "model_name": "threat",
        "score": 0.319
      },
      {
        "model": 20,
        "model_name": "toxic",
        "score": 0.320
      },
      {
        "model": 22,
        "model_name": "obscene",
        "score": 0.320
      }
    ],
    "URLs": ["https://example.com/feed/", "https://example.com/about/", ...],
    "Example URL": "https://example.com/feed/", 
    "Domain Score": 0.32
  }, 
  "dt_created": "2019-12-11T14:38:35.246635+00:00",
  "dt_updated": "2019-12-14T07:30:08.748324+00:00"
}
```

> If a domain is still processing, the following response is returned

```json
{
   "domain": "example.com", 
   "status": "progress",
   "score": null,
   "dt_created": "2019-12-19T09:42:11.110089+00:00", 
   "dt_updated": "2019-12-19T09:42:11.110094+00:00"
}
```

> If a domain is absent, the following response is returned

```json
{
   "domain": "example.com", 
   "status": "absent"
}
```

> If a domain finished with an error, the following response is returned

```json
{
  "domain": "example.com", 
  "status": "error",
  "score": "Error message", 
  "dt_created": "2019-11-04T13:24:12.213925+00:00", 
  "dt_updated": "2019-11-05T00:45:06.583050+00:00"
}
```


### HTTP Request

`GET https://api.factmata.com/api/v0.1/score/domain/?domain="www.example.com"&domain="example.com"`

Fetching the scores has the following use cases:

### Domain is successfully processed 

Status `success` and `score` with model_name scores are returned.

### Domain is processed with an error

Status `error` and `score` containing the error message are returned.

### Domain is still processing

Status `progress` is returned.

### Domain is absent

Status `absent` is returned.


Parameter | Default | Required | Description
--------- | ------- | -------- | -----------
domain | None | True | A valid domain string. In case ant of the domains are malformed, request is aborted with status `422`. 

# Intelligence

Intelligence API provides users with insights on topics of their choice. Users are provided with opinions, narratives and themes that emerge on a topic and with metrics denoting stance.
The endpoints enable obtaining insights for individual topic, as well comparing a number of topics (no upper limit on the number of topics to be compared).

## Authentication

The  API uses JWT tokens to authenticate requests. Login is performed via AWS Cognito. 
Once the JWT is received via Cognito, it should be passed in every API request via the Authorization 
header using the Bearer schema.

The NodeJS script for issuing JWT tokens from the CLI can be found in [this repository](https://github.com/factmata/cognito-jwt-token).
The JWT_TOKEN env variable can be set using the script from the repo above.

```shell
export JWT_TOKEN=`node issue_token.js`
```



Example of a request after the authentication:

```python
import requests

url = "https://api-gw.staging.factmata.com/api/v1/intelligence/report"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/report' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN" 
```

<aside class="warning">
Do not share your API Key in publicly accessible platforms.
</aside>

<aside class="notice">
Only one report can be handled at a time.
</aside>

## Pagination
All list endpoints give a response in the following format

```json
{
  "count": 5,
  "previous": 1,
  "next": 3,
  "result": [
    {
      "id": 1,
      "title": "Avon Hydra Fusion"
    },
    {
      "id": 2,
      "title": "Avon Hydra"
    }
  ]
}
```

### Response data
Name | Type | Description |
-----| -----| ----------- | -
count | int | Number of all elements
previous | int | Number of the previous pages, empty if there is no previous page
next | int | Number of the next pages, empty if there is no next page
result | list | List of objects limited by `page_size`

## Query params
Name | Type | Default | Description |
-----| -----| ------- | ----------- | -
page | int | 1 | Page to retrieve. Has to be more than 0
page_size | int | 20 | Size of the result. Has to be between 1 and 100

## Common errors
Code | Text | Description |
-----| ---- | ----------- | -
401 | {'message': 'Unauthorized'} | API Gateway response when the custom or Amazon Cognito authorizer failed to authenticate the caller.
403 | {'message': 'Access denied'} | API Gateway response for authorization failure. Access is denied by Amazon Cognito authorizer 
403 | {'message': 'Expired token'} | API Gateway response for an AWS authentication token expired error 
403 | {'message': 'Invalid API key'} | API Gateway response for an invalid API key submitted for a method requiring an API key
403 | {'message': 'Invalid signature'} | API Gateway response for an invalid AWS signature error
403 | {'message': 'Missing authentication token'} | API Gateway response for a missing authentication token error, including the cases when the client attempts to invoke an unsupported API method or resource
403 | {'message': 'WAF filtered'} | API Gateway response when a request is blocked by AWS WAF
404 | {'message': 'Object with id $ID not found'} | The wrong id passed to detail endpoint


## Definitions

Report - a set of insights on a particular topic. There are two types of reports: Insights and Comparison.
- Insights - a set of insights on a single topic (e.g. Insights Report on protein powders).
- Comparison - a comparison of insights across a number of topics (e.g. Comparison Report on Adidas vs Nike).

*Note*: The current API doesn't allow user to create new comparisons. Only existing ones can be fetched from the DB.

Topic - the subject for which Factmata generates insights for a customer, e.g. an industry (e.g. Protein powders), a brand (e.g. Johnson & Johnson), a product (Avon Hydra Fusion) or an event (e.g. Covid-19 outbreak). The topic is defined by the customer. 

Theme - a prominent aspect of a topic based on opinions that are the most interesting or popular (e.g. themes for the protein powders topic would be: price, flavour, ingredients, etc. Themes for Covid-19 outbreak would be: economy, vaccine, NHS, etc.). The themes are automatically extracted from the data.

Narrative - describes a story that emerges in a theme (e.g. narratives under the theme ‘flavour’ could be ‘plain taste’, ‘chalky flavour.’ For the theme 'vaccine', the narratives could be 'inefficient vaccination', 'vaccine availability', etc). Narratives are clusters of similar opinions and are generated automatically from the data.

Opinion - a statement made on a specific topic (e.g. 'The new range of Protein Powder X has a horrible chalky taste’ or 'It will take at least 18 months before Covid-19 vaccine is available for the public'). Opinions are automatically extracted from the data and then grouped into narratives.

Opinion Maker - author of the opinion (e.g. John Smith, the World Health Organisation). Opinion makers are automatically extracted from the data.


## Intelligence Metrics


The following table lists the current metrics and their definitions.

## Metrics definition
Name | Definition  
-----| ---- |-
negative_stance_score | Likelihood of a piece of text being unfavourable towards the topic 
positive_stance_score | Likelihood of a piece of text being favourable towards the topic 
outlierness_score | Interesting content which is less frequently featured on the Internet or seem unusual
popularity_score | Number of shares and likes a piece of text has received 
influencer_score | Number of Twitter followers an opinion maker has 
propaganda_score | Likelihood that a piece of text is politically biased
bot_generated_score | Likelihood that a piece of text have been generated by bots
threat_score |Likelihood of biased views being widely spread. The metric takes into account propaganda score and popularity


The following table lists the current metrics and their type and range.

## Metrics type and range
Name | Type | Range  
-----| ----- | ----- |-
negative_stance_score  | float | 0.00 - 1.00
positive_stance_score | float | 0.00 - 1.00
outlierness_score | float | 0.00 - 1.00
popularity_score |  integer | 0+
influencer_score | integer | 0+
propaganda_score | float | 0.00 - 1.00
bot_generated_score  | float | 0.00 - 1.00
threat_score | float | 0.00 - 1.00


The following table lists the current metrics and their availability per the type of product proposition.

##  Metrics by Factmata’s product proposition
Name | Brands proposition | Fake news proposition
-----| --------------------------- | -------------------------------- | -
negative_stance_score | yes | yes
positive_stance_score | yes | yes
outlierness_score | yes | yes
popularity_score | yes | yes
influencer_score | yes | yes
propaganda_score | no | yes
bot_generated_score | no | yes
threat_score | no | yes


The following table lists the current metrics and their availability per text's domain type.

##  Metrics for Brands proposition by text's domain type
Name | Social media | Any other* 
-----| --------------------------- | -------------------------------- | -
negative_stance_score | yes | yes
positive_stance_score | yes | yes
outlierness_score | yes | no
popularity_score | yes | no
influencer_score | yes | no

*E.g. reviews, news, customer care transcripts.


The following table lists the current metrics and their availability per entity type.

## Metrics by entity type
Name | Theme | Narrative | Opinion | Opinion Maker | WebContent
-----| ----- | --------- | ------- | ------------- | ---------- | -
negative_stance_score | yes | yes | yes | no | no
positive_stance_score | yes | yes | yes | no | no
outlierness_score | yes | yes | no | no | no
popularity_score | yes | yes | yes | no | no
influencer_score | yes | yes | no | yes | no
propaganda_score | yes | yes | no | no | yes
bot_generated_score | yes | yes | yes | no | no
threat_score | yes | yes | no | no | no



## API endpoints

## Reports

Inights - a set of insights on a particular topics (e.g. insights report on protein powders).

Report may be in one of the following statuses: `pending`, `success`, `error`

## List reports
```python
import requests

url = "https://api-gw.staging.factmata.com/api/v1/intelligence/report"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/report' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN"

```

> Response example

```json
[
  {
    "id": 1,
    "type": "insights",
    "name": "Report 1",
    "error": null,
    "created_at": "2019-01-01T00:00:00+00:00",
    "updated_at": "2019-01-01T00:00:00+00:00",
    "status": "pending"
  },
  {
    "id": 2,
    "type": "comparison",
    "name": "Report 2",
    "error": null,
    "created_at": "2019-01-01T00:00:00+00:00",
    "updated_at": "2019-01-01T00:00:00+00:00",
    "status": "success"
  }
]
```

Returns a list customer's reports.

#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/report`

## Detail reports
```python
import requests

url = "https://api-gw.staging.factmata.com/api/v1/intelligence/report/{report_id}"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/report/$REPORT_ID' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN"

```

> Response example

```json
{
  "id": 1,
  "type": "insights",
  "name": "Report 1",
  "error": null,
  "created_at": "2019-01-01T00:00:00+00:00",
  "updated_at": "2019-01-01T00:00:00+00:00",
  "status": "success"
}
```

Returns a report by its id.
In the code, replace 'ID' with the id number of the report you wish to obtain. The id numbers are provided by 'list reports' query.

#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/report/:reportId`

## Retrieve report upload presigned link
```python
import requests

url = "https://api-gw.staging.factmata.com/api/v1/intelligence/report/s3-link"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}

params = {
  'client': f'{CLIENT_NAME}',
  'product': 'report'
}

res = requests.port(url, headers=headers, json=params)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/report/s3-link' \
  --data '{"client": "$CLIENT", "product": "report"}' \
  -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN"

```

> Response example

```json
{
    "url": "https://s3.bucket_url",
    "fields": {
        "acl": "private",
        "key": "object_name",
        "AWSAccessKeyId": "access_key_id",
        "policy": "policy",
        "signature": "signature"
    }
}
```

Returns an S3 link for file upload.

#### HTTP Request

`POST https://api-gw.staging.factmata.com/api/v1/intelligence/report/s3-link`

### Request Payload

Parameter | Required | Default | Description
--------- | -------- | ------- | -----------
client | True | None | Name of the client.
product | True | None | Must be 'report' in this version.


### CSV File format

Column name | Data type | Required | Placeholder value | Details
----------- | --------- | -------- | ----------------- | -------
product_name | string | True | |
product_brandname | string | True | |
title | string | True | |
full_text_review | string | True | |
Body | string | True | |
star_rating | int | False | 0 |
reviewer_name | string | True | | 
date_of_review | string |True | | Date in dd-mmm-yy format (e.g. 01-Mar-19)
review_date_formatted | string | True | | Date in dd-mmm-yy format (e.g. 01-Mar-19)
comments | string | False | "" |
product_url | string | False | "" |
reviewer_url | string | False | "" |
review_url | string | True | | A valid URL.
style | string | False | "" |
helpful | bool | False | True |
verified_purchaser | bool | False | True |
flavor | string | False | "" |
size | string | False | "" |
color | string | False | "" |
Count_of_Word | int | False | 0 |


## Report create
```python
import requests

url = "https://api-gw.staging.factmata.com/api/v1/intelligence/report"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}

params = {
  'client': f'{CLIENT_NAME}',
  'product': 'report',
  'name': f{NAME},
  'topics': [<list of topics>],
  's3_object_name': f'{S3_LINK}'
}

res = requests.port(url, headers=headers, json=params)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/report' \
  --data '{"client": "$CLIENT", "product": "report", "name": "$NAME", "topics": ["$TOPIC1", "$TOPIC2"], "s3_object_name": "$S3_LINK"}' \
  -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN"

```

> Response example

```json
{
   "id":1,
   "type":"insights",
   "name":"Report 1",
   "error": null,
   "created_at": "2019-01-01T00:00:00+00:00",
   "updated_at": "2019-01-01T00:00:00+00:00",
   "status": "pending"
}
```

Creates a Report entity. A S3 link should be generated for the new Report and provided as a parameter.


<aside class="notice">
 Before creating second report you should wait till first is done, otherwise you get HTTP 429 Too Many Requests. 
</aside>


#### HTTP Request

`POST https://api-gw.staging.factmata.com/api/v1/intelligence/report`

### Request Payload

Parameter | Required | Default | Description
--------- | -------- | ------- | -----------
client | True | None | Name of the client.
product | True | None | Must be 'report' in this version of API.
name | True | None | Name of the report
topics | True | None | List of the topics in the uploaded file. List[string]
s3_object_name | True | None | name of the s3 uploaded file (like /input/bcg/protein/123.csv).

## Report version

A report version represents different versions of the same report and the data (topic, narratives) that belong to the specific version.

## List versions
```python
import requests

url = "https://api-gw.staging.factmata.com/api/v1/intelligence/report/{report_id}/version"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/report/$REPORT_ID/version' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN"

```

> Response example

```json
[
   {
      "id":1,
      "name":"Version 1",
      "created_at": "2019-01-01T00:00:00+00:00",
      "updated_at": "2019-01-01T00:00:00+00:00"
   },
   {
      "id":2,
      "name":"Version 2",
      "created_at": "2019-01-01T00:00:00+00:00",
      "updated_at": "2019-01-01T00:00:00+00:00"
   }
]
```

Returns a list of versions available for a report.

#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/report/:reportId/version`

## Copy version
```python
import requests

url = "https://api-gw.staging.factmata.com/api/v1/intelligence/report/{report_id}/version"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}

data = {
  'base_version_id': base_version_id
}
res = requests.post(url, headers=headers, json=data)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/report/$REPORT_ID/version' \
  --data '{"base_version_id": $BASE_VERSION_ID}'\
  -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN"

```

> Response example

```json
{
  "id":2
}
```

Makes a copy of a report by creating a new version with all of the data copied.

#### HTTP Request

`POST https://api-gw.staging.factmata.com/api/v1/intelligence/report/:reportId/version`

### Request Payload

Parameter | Required | Default | Description
--------- | -------- | ------- | -----------
base_version_id | True | None | Id of a report version to copy all data from.

## Update report version
```python
import requests

url = "https://api-gw.staging.factmata.com/api/v1/intelligece/version/{report_id}"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.post(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/version/$REPORT_ID' \
  -X PATCH \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN"

```

> Response example

```json
{
  "id":2,
  "name":"Version 2",
  "created_at": "2019-01-01T00:00:00+00:00",
  "updated_at": "2019-01-01T00:00:00+00:00"
}
```



#### HTTP Request

`PATCH https://api-gw.staging.factmata.com/api/v1/intelligence/version/:reportId`

## Topics

Topic - the subject for which Factmata generates insights for a customer, e.g. an industry (e.g. Protein powders), a brand (e.g. Avon), a product (J&J Baby Shampoo) or an event (e.g. Coronavirus outbreak). The topic is defined by the customer. Customer can obtain insights on individual topic (Insights Report) as well as compare multiple topics (Insights Comparison Report).


## List topics


```python
import requests

url = "https://api-gw.staging.factmata.com/api/v1/intelligence/report/{report_id}/version/{report_version_id}/topic"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/report/$REPORT_ID/version/$REPORT_VERSION_ID/topic' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN" 
```

> Response example

```json
[
  {
    "id": 12,
    "title": "Avon Hydra Fusion",
    "created_at": "2019-01-01T00:00:00+00:00"
  }
]
```

Returns a list of topics analyzed for a customer.

#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/report/:reportId/version/:reportVersionId/topic`

## Detail topics

```python
import requests

url = f"https://api-gw.staging.factmata.com/api/v1/intelligence/topic/{topic_id}"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/topic/$TOPIC_ID' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN" 
```

> Response example

```json
{
  "id": 12,
  "title": "Avon Hydra Fusion",
  "created_at": "2019-01-01T00:00:00+00:00"
}
```

Returns a topic by its id.


#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/topic/:topicId`

## Themes

Theme - a prominent aspect of a topic based on opinions that are the most interesting or popular (e.g. themes for the protein powders topic would be: price, flavour, ingredients, etc. Themes for Covid-19 outbreak would be: economy, vaccine, NHS, etc.). The themes are automatically extracted from the data.


## List topic themes

```python
import requests

url = f"https://api-gw.staging.factmata.com/api/v1/intelligence/topic/{topic_id}/theme"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/topic/$TOPIC_ID/theme' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN" 
```
> Response example

```json
[
  {
    "id": 1,
    "title": "cost",
    "metrics": {
      "negative_stance_score": 0.32324123,
      "positive_stance_score": 0.22623175,
      "outlierness_score": 0.296705679,
      "popularity_score": 0.56848747,
      "propaganda_score": 0.48008755,
      "threat_score": 0.08953477,
      "num_narratives": 65,
      "num_opinions": 120
    },
    "metrics_by_date": [
      {
        "recorded_on": "2018-09-06T00:00:00+00:00",
        "values": {
          "negative_stance_score": 0.41308644,
          "positive_stance_score": 0.48008755,
          "outlierness_score": 0.254146592,
          "popularity_score": 0.276934175,
          "propaganda_score": 0.090985734,
          "threat_score": 0.07531446
        }
      }
    ],
    "created_at": "2019-11-01T00:00:00+00:00",
    "oldest_webcontent_published_at": "2018-01-03T00:00:00+00:00",
    "newest_webcontent_published_at": "2018-04-23T00:00:00+00:00"
  }
]
```

Returns a list of themes in a topic.

#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/topic/:topicId/theme`

#### Query parameters
Name | Type | Description  
-----| ---- | ----------- | -
sort_by | string | Sorting key. The default value is `num_narratives`. Supported values include the ones returned in the metrics array. Items in `metrics_by_date array` are sorted in chronological order based on `recorded_on`.
created_at_lt | ISO 8601 string | Filter by created_at, less than 
created_at_gt | ISO 8601 string | Filter by created_at, greater than  
created_at_lte | ISO 8601 string | Filter by created_at, less than or equal 
created_at_gte | ISO 8601 string | Filter by created_at, greater than or equal 
metrics_created_at_lt | ISO 8601 string | Filter metrics list by created_at,  less than
metrics_created_at_gt | ISO 8601 string | Filter metrics list by created_at,  greater than 
metrics_created_at_lte | ISO 8601 string | Filter metrics list by created_at,  less than or equal
metrics_created_at_gte | ISO 8601 string | Filter metrics list by created_at,  greater than or equal
num_narratives_gte  |  int  |  The default is 3. Filter by num_narratives, greater than or equal

## Detail themes

```python
import requests

url = f"https://api-gw.staging.factmata.com/api/v1/intelligence/theme/{theme_id}"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/theme/$THEME_ID' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN" 
```

> Response example

```json
{
  "id": 1,
  "title": "cost",
  "metrics": {
    "negative_stance_score": 0.32324123,
    "positive_stance_score": 0.22623175,
    "outlierness_score": 0.296705679,
    "popularity_score": 0.56848747,
    "propaganda_score": 0.48008755,
    "threat_score": 0.08953477,
    "num_narratives": 65,
    "num_opinions": 120
  },
  "metrics_by_date": [
    {
      "recorded_on": "2018-09-06T00:00:00+00:00",
      "values": {
        "negative_stance_score": 0.41308644,
        "positive_stance_score": 0.48008755,
        "outlierness_score": 0.254146592,
        "popularity_score": 0.276934175,
        "propaganda_score": 0.090985734,
        "threat_score": 0.07531446
      }
    }
  ],
  "created_at": "2019-11-01T00:00:00+00:00",
  "oldest_webcontent_published_at": "2018-01-03T00:00:00+00:00",
  "newest_webcontent_published_at": "2018-04-23T00:00:00+00:00"
}
```

Return a theme by its id.


#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/theme/:themeId`

#### Query parameters
Name | Type | Description  
-----| ---- | ----------- | -
metrics_created_at_lt | ISO 8601 string | Filter metrics list by created_at,  less than
metrics_created_at_gt | ISO 8601 string | Filter metrics list by created_at,  greater than 
metrics_created_at_lte | ISO 8601 string | Filter metrics list by created_at,  less than or equal
metrics_created_at_gte | ISO 8601 string | Filter metrics list by created_at,  greater than or equal


## Narratives

Narrative - describes a story that emerges in a theme (e.g. narratives under the theme ‘flavour’ could be ‘plain taste’, ‘chalky flavour.’ For the theme 'vaccine', the narratives could be 'inefficient vaccination', 'vaccine availability', etc). Narratives are clusters of similar opinions and are generated automatically from the data.


## List narratives

```python
import requests

url = f"https://api-gw.staging.factmata.com/v1/intelligence/topic/{topic_id}/theme/{theme_id}/narrative"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/v1/intelligence/topic/$TOPIC_ID/theme/$THEME_ID/narrative' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN" 
```
> Response example

```json
[
  {
    "id": 8,
    "title": "Cream based cosmetics",
    "metrics": {
      "negative_stance_score": 0.32324123,
      "positive_stance_score": 0.22623175,
      "outlierness_score": 0.296705679,
      "popularity_score": 0.56848747,
      "propaganda_score": 0.48008755,
      "threat_score": 0.08953477,
      "num_opinions": 223
    },
    "metrics_by_date": [
      {
        "recorded_on": "2018-09-06T00:00:00+00:00",
        "values": {
          "negative_stance_score": 0.41308644,
          "positive_stance_score": 0.48008755,
          "outlierness_score": 0.254146592,
          "popularity_score": 0.276934175,
          "propaganda_score": 0.090985734,
          "threat_score": 0.07531446
        }
      }
    ],
    "created_at": "2019-11-01T00:00:00+00:00",
    "oldest_webcontent_published_at": "2018-01-03T00:00:00+00:00",
    "newest_webcontent_published_at": "2018-04-23T00:00:00+00:00"
  }
]
```

Returns a list of narratives for a theme.


#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/topic/:topic_id/theme/:theme_id/narrative`



#### Query parameters
Name | Type | Description  
-----| ---- | ----------- | -
created_at_lt | ISO 8601 string | Filter by created_at, less than 
created_at_gt | ISO 8601 string | Filter by created_at, greater than  
created_at_lte | ISO 8601 string | Filter by created_at, less than or equal 
created_at_gte | ISO 8601 string | Filter by created_at, greater than or equal 
metrics_created_at_lt | ISO 8601 string | Filter metrics list by created_at,  less than
metrics_created_at_gt | ISO 8601 string | Filter metrics list by created_at,  greater than 
metrics_created_at_lte | ISO 8601 string | Filter metrics list by created_at,  less than or equal
metrics_created_at_gte | ISO 8601 string | Filter metrics list by created_at,  greater than or equal
sort_by | string | Sorting key. The default value is `num_opinions`. Supported values include the ones returned in the metrics array. Items in `metrics_by_date array` are sorted in chronological order based on `recorded_on`.


## Detail narratives


```python
import requests

url = f"https://api-gw.staging.factmata.com/api/v1/intelligence/narrative/{narrative_id}"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/narrative/$NARRATIVE_ID' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN" 
```

> Response example

```json
{
  "id": 8,
  "title": "Cream based cosmetics",
  "metrics": {
    "negative_stance_score": 0.32324123,
    "positive_stance_score": 0.22623175,
    "outlierness_score": 0.296705679,
    "popularity_score": 0.56848747,
    "propaganda_score": 0.48008755,
    "threat_score": 0.08953477,
    "num_opinions": 223
  },
  "metrics_by_date": [
    {
      "recorded_on": "2018-09-06T00:00:00+00:00",
      "values": {
        "negative_stance_score": 0.41308644,
        "positive_stance_score": 0.48008755,
        "outlierness_score": 0.254146592,
        "popularity_score": 0.276934175,
        "propaganda_score": 0.090985734,
        "threat_score": 0.07531446,
        "num_opinions": 18
      }
    }
  ],
  "created_at": "2019-11-01T00:00:00+00:00",
  "oldest_webcontent_published_at": "2018-01-03T00:00:00+00:00",
  "newest_webcontent_published_at": "2018-04-23T00:00:00+00:00"
}
```

Return a narrative by its id.


#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/narrative/:narrativeId`

#### Query parameters
Name | Type | Description  
-----| ---- | ----------- | -
metrics_created_at_lt | ISO 8601 string | Filter metrics list by created_at,  less than
metrics_created_at_gt | ISO 8601 string | Filter metrics list by created_at,  greater than 
metrics_created_at_lte | ISO 8601 string | Filter metrics list by created_at,  less than or equal
metrics_created_at_gte | ISO 8601 string | Filter metrics list by created_at,  greater than or equal


## Opinions

Opinion - a statement made on a specific topic (e.g. 'The new range of Protein Powder X has a horrible chalky taste’ or 'It will take at least 18 months before Covid-19 vaccine is available for the public'). Opinions are automatically extracted from the data and then grouped into narratives.


## List narrative opinions

```python
import requests

url = f"https://api-gw.staging.factmata.com/v1/intelligence/narrative/{narrative_id}/opinion"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/v1/intelligence/narrative/$NARRATIVE_ID/opinion' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN" 
```
> Response example

```json
[
  {
    "id": 199,
    "text": "acquia has a high cost  at 100k annually",
    "metrics": {
      "negative_stance_score": 0.41308644,
      "positive_stance_score": 0.48008755,
      "bot_generated_score": 0.254146592,
      "popularity_score": 0.276934175
    },
    "created_at": "2019-11-01T00:00:00+00:00",
    "opinion_makers": [
      {
        "id": 8,
        "url": "https://twitter.com/@raybae689",
        "influencer_score": 0.32
      }
    ],
    "webcontents": [
      {
        "type": "ARTICLE",
        "download_url": "https://api.factmata.com/v1/downloads/webcontents/53374d64-223d-11ea-978f-2e728ce88125.json",
        "author": "Roydon Ng",
        "author_url": "https://twitter.com/RoydonNg",
        "source": "christiantoday.com.au",
        "page_url": "https://christiantoday.com.au/news/democracy-cant-save-hong-kong.html",
        "resolved_page_url": null,
        "published_at": "2019-10-11T00:00:00+00:00",
        "metrics": {
          "propaganda_score": 0.03
        }
      }
    ]
  },
]
```

Returns a list of opinions for a narrative.


#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/narrative/:narrativeId/opinion`


#### Query parameters
Name | Type | Description  
-----| ---- | ----------- | -
sort_by | string | Sorting key. The default value is popularity_score. Supported values include the ones returned in the metrics array.


## Detail opinions

```python
import requests

url = f"https://api-gw.staging.factmata.com/v1/opinion/{opinion_id}"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/opinion/$OPINION_ID' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN" 
```

> Response example

```json
{
  "id": 199,
  "text": "acquia has a high cost  at 100k annually",
  "metrics": {
    "negative_stance_score": 0.41308644,
    "positive_stance_score": 0.48008755,
    "bot_generated_score": 0.254146592,
    "popularity_score": 0.276934175
  },
  "created_at": "2019-11-01T00:00:00+00:00",
  "opinion_makers": [
    {
      "id": 8,
      "url": "https://twitter.com/@raybae689",
      "influencer_score": 0.32
    },
  ],
  "webcontents": [
    {
      "type": "ARTICLE",
      "download_url": "https://api.factmata.com/v1/downloads/webcontents/53374d64-223d-11ea-978f-2e728ce88125.json",
      "author": "Roydon Ng",
      "author_url": "https://twitter.com/RoydonNg",
      "source": "christiantoday.com.au",
      "page_url": "https://christiantoday.com.au/news/democracy-cant-save-hong-kong.html",
      "resolved_page_url": null,
      "published_at": "2019-10-11T00:00:00+00:00",
      "metrics": {
        "propaganda_score": 0.03
      }
    }
  ]
}
```

Returns an opinion by its ID.


#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/opinion/:opnionId`


## Opinions makers

Opinion Maker - author of the opinion (e.g. John Smith, the World Health Organisation). Opinion makers are automatically extracted from the data.

## List opinion makers

```python
import requests

url = f"https://api-gw.staging.factmata.com/v1/intelligence/opinion/{opinion_id}/opinion_maker"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/v1/intelligence/opinion/$OPINION_ID/opinion_maker' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN" 
```
> Response example

```json
[
  {
    "id": 8,
    "url": "https://twitter.com/@raybae689",
    "influencer_score": 0.32,
    "created_at": "2019-01-01T00:00:00+00:00"
  }
]
```

Returns a list of opinion makers for an opinion sorted by influencer_score in descending order.


#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/opinion/:opinion_id/opinion_maker`


## Detail opinion makers

```python
import requests

url = f"https://api-gw.staging.factmata.com/v1/intelligence/opinion_maker/{opinion_maker_id}"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/v1/intelligence/opinion_maker/$OPINION_MAKER_ID' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN" 
```

> Response example

```json
{
  "id": 8,
  "url": "https://twitter.com/@raybae689",
  "influencer_score": 0.32,
  "created_at": "2019-01-01T00:00:00+00:00"
}
```

Returns an opinion maker by its id.


#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/opinion_maker/:opinionMakerId`

## Comparisons

Comparison of insights across a number of topics (e.g. Adidas vs Nike). The topics are compared across their common themes.

## List comparisons

```python
import requests

url = f"https://api-gw.staging.factmata.com/api/v1/intelligence/report/{report_id}/version/{report_version_id}/comparison"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/report/$REPORT_ID/version/$REPORT_VERSION_ID/comparison' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN" 
```
> Response example

```json
[
  {
    "id": 1,
    "title": "Avon vs. Neutrogena",
    "created_at": "2019-01-01T00:00:00+00:00"
  }
]
```

Returns a list of comparisons.

*Note*: The current API doesn't allow user to create new comparisons. Only existing ones can be fetched from the DB.


#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/report/:reportId/version/:reportVersionId/comparison`


## Detail comparisons

```python
import requests

url = f"https://api-gw.staging.factmata.com/api/v1/intelligence/comparison/{comparison_id}"

headers = {
  'X-API-KEY': f'Bearer {JWT_TOKEN}'
}
res = requests.get(url, headers=headers)
```

```shell
curl 'https://api-gw.staging.factmata.com/api/v1/intelligence/comparison/$COMPARISON_ID' \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: Bearer $JWT_TOKEN" 
```

> Response example

```json
{
  "id": 1,
  "title": "Avon vs. Neutrogena",
  "created_at": "2019-01-01T00:00:00+00:00",
  "topics": [
    {
      "id": 12,
      "title": "Avon Hydra Fusion",
      "created_at": "2019-01-01T00:00:00+00:00",
      "themes": [
        {
          "id": 1,
          "title": "cost",
          "topic_id": 23,
          "metrics": {
            "negative_stance_score": 0.32324123,
            "positive_stance_score": 0.22623175,
            "outlierness_score": 0.296705679,
            "popularity_score": 0.56848747,
            "propaganda_score": 0.48008755,
            "threat_score": 0.08953477,
            "num_narratives": 65,
            "num_opinions": 120
          },
          "metrics_by_date": [
            {
              "recorded_on": "2018-09-06T00:00:00+00:00",
              "values": {
                "negative_stance_score": 0.41308644,
                "positive_stance_score": 0.48008755,
                "outlierness_score": 0.254146592,
                "popularity_score": 0.276934175,
                "propaganda_score": 0.090985734,
                "threat_score": 0.07531446
              }
            }
          ],
          "created_at": "2019-11-01T00:00:00+00:00",
          "oldest_webcontent_published_at": "2018-01-03T00:00:00+00:00",
          "newest_webcontent_published_at": "2018-04-23T00:00:00+00:00"
        }
      ]
    }
  ]
}
```

Returns a comparison by its ID.

*Note*: The current API doesn't allow user to create new comparisons. Only existing ones can be fetched from the DB.


#### HTTP Request

`GET https://api-gw.staging.factmata.com/api/v1/intelligence/comparison/:comparisonId`

#### Query parameters
Name | Type | Description  
-----| ---- | ----------- | -
metrics_created_at_lt | ISO 8601 string | Filter metrics list by created_at,  less than
metrics_created_at_gt | ISO 8601 string | Filter metrics list by created_at,  greater than 
metrics_created_at_lte | ISO 8601 string | Filter metrics list by created_at,  less than or equal
metrics_created_at_gte | ISO 8601 string | Filter metrics list by created_at,  greater than or equal

