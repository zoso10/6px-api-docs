# 6px API Documentation

## OVERVIEW

The 6px API is built on HTTP. Our API is [RESTful](http://en.wikipedia.org/wiki/Representational_State_Transfer) and it:

* Has predictable, resource-oriented URLs
* Uses built-in HTTP capabilities for passing parameters
* Responds with standard HTTP response codes to indicate errors
* Sends and receives data as [JSON](http://en.wikipedia.org/wiki/JSON)

6px has published [SDKs](#sdks) for various languages.

To give you an idea of how to use the API, we have annotated our documentation with code samples written in curl.

* [Base URL](#base-url)
* [Authentication](#authentication)
* [Date Format](#date-format)
* [Errors](#errors)
* [Limits](#limits)
* [Pagination](#pagination)
* [Sorting](#sorting)
* [Search](#search)
	* [Number](#number)
	* [String](#string)
	* [Location](#location)
* [Methods](#methods)
    * [Rotate](#rotate)
    * [Resize](#resize)
    * [Crop](#crop)
    * [Filter](#filter)
    * [Layer](#layer)
* [Input JSON](#input-json)
* [Output JSON](#output-json)
* [Endpoints](#endpoints)
	* [Create](#create)
	* [Get](#get)
	* [List](#list)
* [Web Sockets](#sockets)
* [SDKs](#sdks)

## BASE URL

All API endpoints referenced in this documentation should start with the following base URL:

```bash
https://api.6px.io/v1/
```

## AUTHENTICATION

When you sign up for an account, you are given an API key and secret. You authenticate with the 6px API by providing your API key and secret in every request. You can manage your API key and secret in the `My Account` section of the [dashboard](http://6px.io/login).

```bash
$ curl https://api.6px.io/v1/users/:user_id/jobs?key=YOUR_KEY&secret=YOUR_SECRET
```

> **Authentication using an API secret should only be used in server to server scenarios. If you're making a request via from a browser via AJAX, please omit the secret and ensure that the requesting domain is whitelisted in the account section of your dashboard.**

## DATE FORMAT

6px returns JSON for all API calls. Dates are passed as an [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) formatted datetime.

```bash
2014-01-23T19:40:42+17:00
```

## ERRORS


| Code    | Error                 | Description                                                                        |
|---------|-----------------------|------------------------------------------------------------------------------------|
| `200`   | OK                    | Everything worked as expected                                                      |
| `400`   | Bad Request           | Often missing a required parameter                                                 |
| `401`   | Unauthorized          | Invalid API key or secret, or the domain is not allowed                            |
| `404`   | Not Found             | The requested resource doesn’t exist                                               |
| `429`   | Too Many Requests     | You are sending too many requests too quickly or you have met your plans API quota |
| `500`   | Internal Server Error | Something went wrong on our end                                                    |

## LIMITS

Rate limits are defined by the number of API calls allowed by your subscription. Only `POST` requests to create a job are metered.

## PAGINATION

Requests that return multiple results will be paginated to 10 resources by default. You can specify further resources with the `?page` parameter.

```bash
$ curl https://api.6px.io/v1/users/:user_id/jobs?page=2
```

To increase the number of resources returned, simply pass the `?per_page` parameter.

```bash
$ curl https://api.6px.io/v1/users/:user_id/jobs?page=2&per_page=100
```

## SORTING

**Sort by value:**
```bash
$ curl https://api.6px.io/v1/users/:user_id/jobs?sort_by=created
```

**Sort by value (descending):**
```bash
$ curl https://api.6px.io/v1/users/:user_id/jobs?sort_by=created,desc
```

**Sort by value (ascending):**
```bash
$ curl https://api.6px.io/v1/users/:user_id/jobs?sort_by=created,asc
```

## SEARCH `experimental`

The search functionality is optimized to help you find jobs using query parameters.

**For example:**
* Image(s) within a specific area
* Image(s) with a specific tag
* Image(s) with specific methods

### NUMBER

* `?number={all}123,456` Both 123 and 456 must be present
* `?number={nin}123,456` Neither 123 nor 456
* `?number={in}123,456` Either 123 or 456
* `?number={gt}123` > 123
* `?number={gte}123` >= 123
* `?number={lt}123` < 123
* `?number={lte}123` <= 123
* `?number={ne}123` Not 123
* `?number={mod}10,2` Where (number / 10) has remainder 2

**Example:**
```bash
$ curl https://api.6px.io/v1/users/:user_id/jobs?processed.bytes={gt}1048576
```

### STRING

* `?string={all}foo,bar` Both foo and bar must be present
* `?string={nin}foo,bar` Neither foo nor bar
* `?string={in}foo,bar` Either foo or bar
* `?string={not}foo` Not foo
* `?string={exact}fOoBaR` Case-sensitive exact match of fOoBaR

**Example:**
```bash
$ curl https://api.6px.io/v1/users/:user_id/jobs?status=processing
```

### LOCATION

* `?latlon={near}40.0176,-105.2797,5` Near 40.0176,-105.2797 (with a 5 mile radius)
* `?latlon={near}40.0176,-105.2797` Near 40.0176,-105.2797 (no radius limit, automatically sorts by distance)

**Example:**
```bash
$ curl https://api.6px.io/v1/users/:user_id/jobs?processed.images.latlon={near}40.0176,-105.2797,10
```

## METHODS

### ROTATE

| Options | Type      | Required | Description                                                           |
|---------|-----------|----------|-----------------------------------------------------------------------|
| `degrees` | Number  | True     | -                                                                     |

**Example:**
```json
{
	"method": "rotate",
	"options": {
		"degrees": 90
	}
}
```

### RESIZE

| Options | Type    | Required   | Description                                                          |
|---------|---------|------------|----------------------------------------------------------------------|
| `height`  | Number  | False    | -                                                                    |
| `width`   | Number  | False    | -                                                                    |

**Example 1:**
```json
{
	"method": "resize",
	"options": {
		"height": 200,
		"width" : 200
	}
}
```

**Example 2:**
```json
{
	"method": "resize",
	"options": {
		"width": 400
	}
}
```

> **Height and/or width are required parameters. Automatic proprotions provided if either parameter is omitted.**

### CROP

| Options   | Type    | Required | Description                                                                    |
|-----------|---------|----------|--------------------------------------------------------------------------------|
| `height`  | Number  | False    | -                                                                              |
| `width`   | Number  | False    | -                                                                              |
| `x`       | Number  | False    | -                                                                              |
| `y`       | Number  | False    | -                                                                              |
| `face`    | Boolean | False    | Crops to dominant face. Omit `x` and `y` coordinates when using this option.   |
| `padding` | Number  | False    | Sets specified padding when using `face` parameter.                            |

**Example 1:**
```json
{
	"method": "crop",
	"options": {
		"height": 200,
		"width": 200,
		"x": 150,
		"y" 300
	}
}
```

**Example 2:**
```json
{
	"method": "crop",
	"options": {
		"height": 100,
		"face": true
	}
}
```

**Example 3:**
```json
{
	"method": "crop",
	"options": {
		"height": 100,
		"width": 100,
		"face": true,
		"padding" 20
	}
}
```

### FILTER

| Options        | Type    | Required | Description                                                          											|
|----------------|---------|----------|-----------------------------------------------------------------------------------------------------------------|
| `sepia`        | Number  | False    | Expects a number between `0` to `100`.  The higher the number, the more sepia that is applied.   											|
| `invert`       | Number  | False    | Expects `true` or `false`. For best results, omit `invert` alltogether if you do not want the filter.  			|
| `brightness`   | Number  | False    | Default value is `0`. If you want to white-wash the image, pass `100` as the value.  Blaken the image by passing `-100` |
| `contrast`     | Number  | False    | Default value is `0`. If you want to double the contrast, pass `100` as the value.|
| `exposure` | Number | False | Adjust the exposure amount in the image.  `-100` to `100` are accepted values.  Defaults to `0`. |
| `noise` | Number | False | Add noise to an image.  Defaults to `0` (no noise).  Maxes out at `100` (a whole lot of noise). |
| `saturation` | Number | False | Range of `-100` to `100` (defaults at `0`).|
| `vibrance` | Number | False | Boosts colors if passed a value above `0`.  Will make colors more dull if passed in a value below `0`. |
| `hue` | Number | False | Range of `0` to `100`|
| `gamma` | Number | False | Range of `0` to `100` |
| `colorize` | String | False | Recolor an image with the passed in hex value.  Make the image blue, just pass in `#0000FF`. |
| `channels`     | Object | False | Pass in an object with at least one channel that you want to modify. `red`, `blue`, and `green` are your options and their default values are `0`.  To cancel a channel out, pass `-100` as the value.
| `sharpen`    | Number  | False    | Default value is `100`. If you want to double the sharpness, send the value `200`.|
| `stackBlur`    | Number  | False    | Pass in the  blue radius. `0` - `100` are accepted values. |

Undocumented: `curves`

**Example 1:**
```json
{
	"method": "filter",
	"options": {
		"sepia": 70
	}
}
```

**Example 2:**
```json
{
	"method": "filter",
	"options": {
		"brightness": 150,
		"stackBlur": 30
	}
}
```

**Example 3:**
```json
{
	"method": "filter",
	"options": {
		"sepia": true,
		"sharpness": 175,
		"blur": true
	}
}
```

### LAYER

| Options        | Type    | Required | Description                                                          											|
|----------------|---------|----------|-----------------------------------------------------------------------------------------------------------------|
| `x`            | Number  | False    | The `x` position for the layered image.  																		|
| `y`            | Number  | False    | The `y` position for the layered image.  																		|
| `opacity`      | Number  | False    | Default value is `1`. Can be omitted if opaque is desired effect.                 			       				|
| `ref`          | String  | False    | Image that you would like to layer on top. This is a reference from the `input` array.                  		|


**Example 1:**
```json
{
	"method": "layer",
	"options": {
		"x": 0,
		"y": 0,
		"opacity": 0.6,
		"ref": "img2"
	}
}
```

**Example 2:**
```json
{
	"method": "layer",
	"options": {
		"opacity": 0.6,
		"ref": "img2"
	}
}
```

> The `x` and `y` values can be omitted if you want to play your layered image at the top left corner (0,0)

## INPUT JSON

### SAMPLE

```json
{
	"data": {},
	"callback": {
		"url": "http://example.com/callback"
	},
	"input": {
		"img1": "http://example.com/path/to/image1.jpg",
		"img2": "http://example.com/path/to/image2.jpg"
	},
	"output": [
		{
			"methods": [
				{
					"method": "resize",
					"options": {
						"width": 200,
						"height": 200
					}
				}
			],
			"type": "image/png",
			"ref": [
				"img1"
			],
			"tag": "6e86e93b7c0f",
			"url": "s3://key:secret@bucket/path"
		}
	]
}
```
#### DATA `object`
An `object` that allows you to store custom data such as a database id, name or email.

**Example Store:**
```json
{
	"data": {
		"id": "1234",
		"email": "foo@example.com"
	}
}
```

You can then easily locate jobs using with a simple [search](#search):
```bash
$ curl https://api.6px.io/v1/users/:user_id/jobs?data.id=1234
```

#### CALLBACK `object`
When your job is complete a `POST` will be sent to an optional callback URL. The `POST` body will contain the full JSON payload from the processed job.

#### INPUT `array`
The `input` array specifies the images that are to be used when processing the job. Input should be a publicly accessable URL or a valid [data URI](https://developer.mozilla.org/en-US/docs/data_URIs).

**The following image formats are supported:**

* `JPG`
* `PNG`
* `GIF`

#### OUTPUT `array`
Specifies operations that are to be run against the images in the `input` array. Multiple outputs can be achieved by providing additional objects.

**Example:**
```json
{
	"output": [
		{
			"methods": [
				{
					"method": "resize",
					"options": {
						"width": 200,
						"height": 200
					}
				}
			],
			"type": "image/png",
			"ref": [
				"main"
			],
			"tag": "6e86e93b7c0f",
			"url": "s3://key:secret@bucket/path"
		},
		{
			"methods": [
				{
					"method": "filter",
					"options": {
						"color": 0
					}
				}
			],
			"type": "image/png",
			"ref": [
				"main"
			],
			"tag": "6e86e93b7c0f",
			"url": "s3://key:secret@bucket/path"
		}
	]
}
```

##### METHODS `array`
Specifies what [methods](#methods) to run against that particular output.

**Example:**
```json
{
	"methods": [
		{
			"method": "resize",
			"options": {
				"height": 400,
				"width": 400
			}
		},
		{
			"method": "filter",
			"options": {
				"color": 0
			}
		}
	]
}
```

##### TYPE `string`
MIME type to save the image as.

**The following MIME types are supported:**

* `image/png`
* `image/jpeg`
* `image/gif`

##### REF `array`
Specifies which input(s) to use. If we specify more than one ref in this block, it will duplicate whatever is in this block for each of the input indexes defined.

**For example, if we have the following input object containg 4 inputs:**
```json
{
	"input": {
		"img1": "http://example.com/path/to/image1.jpg",
		"img2": "http://example.com/path/to/image2.jpg",
		"img3": "http://example.com/path/to/image3.jpg",
		"img4": "http://example.com/path/to/image4.jpg"
	}
}
```

**We could then specify that we want to use the 3rd input image like so:**
```json
{
	"output": [
		{
			"methods": [
				{
					"method": "crop",
					"options": {
						"face": true,
						"padding": 50
					}
				}
			],
			"ref": [
				"img3"
			]
		}
	]
}
```

##### TAG `string`
An optional unique identifer and filename for the given output. If a tag is not provided, we will automatically assign a unique string.

##### URL `string`
A full URL without the filename to which the output image(s) will be uploaded. If an output URL is not specified, your image(s) will automatically be uploaded and served from the 6px CDN.

**The following output locations / connection strings are supported:**

* `s3://key:secret@bucket/path` Amazon S3
* `cf://username:api_key@container/path` Rackspace Cloud Files (US)
* `cf+uk://username:api_key@container/path` Rackspace Cloud Files (UK)
* `ftp://user:password@ftp.example.com/path` FTP
* `sftp://user:password@sftp.example.com/path` SFTP

> **FTP / SFTP**
>
> If the URL contains authentication with special characters, make sure that they are [properly escaped](http://en.wikipedia.org/wiki/Percent-encoding). We try to write from the root of your server, so use an absolute path for your URL to ensure that we can write to your server successfully.

## OUTPUT JSON

### SAMPLE

```json
{
	"__v": 0,
	"_id": "52e1f64007438cb08073d5e9",
	"user_id": "52c747dc04f452f766000001",
	"modified": "2014-01-24T05:12:34.232Z",
	"created": "2014-01-24T05:12:32.466Z",
	"processed": {
		"duration": 0.21689900000000006,
		"bytes": 317955
	},
	"input": {
		"main": "http://example.com/path/to/image.jpg"
	},
	"output": [
		{
			"methods": [
				{
					"method": "resize",
					"options": {
						"width": 200,
						"height": 200
					}
				}
			],
			"type": "image/png",
			"ref": [
				"main"
			],
			"tag": "6e86e93b7c0f",
			"url": "s3://key:secret@bucket/path"
		}
	],
	"status": "complete",
	"callback": {
	    "url": "http://example.com/callback"
	},
	"data": {}
}
```

The output from a job contains all of the fields that were specified in the input, along with general database values and useful metrics.

#### PROCESSED `object`
Contains processing information obtained while processing the image.

##### DURATION `object`
Benchmarks for various tasks for the job.

| Value        | Type     | Description
|--------------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `start`      | Datetime | The time shown as an ISO8601 Datetime that the job started processing.          																																									     |
| `end`        | Datetime | The time shown as an ISO8601 Datetime that the job completed processing.          																																								       |
| `total`      | Number   | The total time in milliseconds taken to complete a job. The timer starts as soon as a job is received and stopps when the job is complete. If you are submitting a batch job, this will be the total time required to process all of the jobs combined.  |
| `upload`     | Number   | The total time in milliseconds taken to upload all of the outputs in your job.   																		                                                                                                |
| `download`   | Number   | The total time in milliseconds taken to download all of the images referenced in the inputs array. 																																					  |                			       																													|

##### BYTES `number`
Size of processed data in bytes. If you are submitting a batch job, this number will be the sum of all processed images.

#### STATUS `string`
Will be one of the following:

* `pending` Default status before job is picked up by worker
* `processing` Worker has taken on the job
* `complete` Job is complete
* `failed` Job has failed

## ENDPOINTS

### CREATE

##### `POST` /users/:user_id/jobs

**Example Request:**
```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"output":[{"methods":[{"method": "resize","options":{"width":200,"height":200}}],"type":"image/png","ref":["main"]}],"input":{"hobbit":"http://0.tqn.com/d/scifi/1/0/n/1/1/-/HBT-008104r.jpg"}}' https://api.6px.io/v1/users/:user_id/jobs
```

**Example Response:**
```json
{
    "_id": "52e1f64007438cb08073d5e8"
}
```

### GET

##### `GET` /users/:user_id/jobs/:job_id

**Example Request:**
```bash
$ curl https://api.6px.io/v1/users/52c747dc04f452f766000001/jobs/52e1f64007438cb08073d5e8
```

**Example Response:**
```json
{
	"__v": 0,
	"_id": "52e1f64007438cb08073d5e8",
	"user_id": "52c747dc04f452f766000001",
	"modified": "2014-01-24T05:12:34.232Z",
	"created": "2014-01-24T05:12:32.466Z",
	"processed": {
		"duration": 0.21689900000000006,
		"bytes": 317955
	},
	"input": {
		"main": "http://example.com/path/to/image.jpg"
	},
	"output": [
		{
			"methods": [
				{
					"method": "resize",
					"options": {
						"width": 200,
						"height": 200
					}
				}
			],
			"type": "image/png",
			"ref": [
				"main"
			],
			"tag": "6e86e93b7c0f",
			"url": "s3://key:secret@bucket/path"
		}
	],
	"status": "complete",
	"callback": {
	    "url": "http://example.com/callback"
	},
	"data": {}
}
```

### LIST

##### `GET` /users/:user_id/jobs

**Example Request:**
```bash
$ curl https://api.6px.io/v1/users/52c747dc04f452f766000001/jobs
```

**Example Response:**
```json
[
	{
		"__v": 0,
		"_id": "52e1f64007438cb08073d5e8",
		"user_id": "52c747dc04f452f766000001",
		"modified": "2014-01-24T05:12:34.232Z",
		"created": "2014-01-24T05:12:32.466Z",
		"processed": {
			"duration": 0.21689900000000006,
			"bytes": 317955
		},
		"input": {
			"main": "http://example.com/path/to/image.jpg"
		},
		"output": [
			{
				"methods": [
					{
						"method": "resize",
						"options": {
							"width": 200,
							"height": 200
						}
					}
				],
				"type": "image/png",
				"ref": [
					"main"
				],
				"tag": "6e86e93b7c0f",
				"url": "s3://key:secret@bucket/path"
			}
		],
		"status": "complete",
		"callback": {
		    "url": "http://example.com/callback"
		},
		"data": {}
	},
	{
		"__v": 0,
		"_id": "52e1f64007438cb08073d5e9",
		"user_id": "52c747dc04f452f766000001",
		"modified": "2014-01-24T05:12:34.232Z",
		"created": "2014-01-24T05:12:32.466Z",
		"processed": {
			"duration": 0.21689900000000006,
			"bytes": 317955
		},
		"input": {
			"main": "http://example.com/path/to/image.jpg"
		},
		"output": [
			{
				"methods": [
					{
						"method": "resize",
						"options": {
							"width": 200,
							"height": 200
						}
					}
				],
				"type": "image/png",
				"ref": [
					"main"
				],
				"tag": "6e86e93b7c0f",
				"url": "s3://key:secret@bucket/path"
			}
		],
		"status": "complete",
		"callback": {
		    "url": "http://example.com/callback"
		},
		"data": {}
	}
]
```

## WEB SOCKETS

The 6px API has full support for web sockets, allowing client and server side SDKs to receive realtime notifications. A notification is currently sent when a job status changes to `processing`, `complete` or `failed`. The payload includes the job id. More documentation to come.

## SDKs

* [Javascript](https://github.com/6px-io/6px-js-sdk)

## HELP US HELP YOU

Please let us how we can make this API better. If you have a specific feature request or found a bug, please drop us a line at ops@6px.io.
