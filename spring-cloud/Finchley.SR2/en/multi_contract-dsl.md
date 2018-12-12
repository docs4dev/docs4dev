## 95. Contract DSL

Spring Cloud Contract supports out of the box 2 types of DSL. One written in  `Groovy`  and one written in  `YAML` .

If you decide to write the contract in Groovy, do not be alarmed if you have not used Groovy before. Knowledge of the language is not really needed, as the Contract DSL uses only a tiny subset of it (only literals, method calls and closures). Also, the DSL is statically typed, to make it programmer-readable without any knowledge of the DSL itself.

|images/important.png|Important|
|----|----|
|Remember that, inside the Groovy contract file, you have to provide the fully qualified name to the  `Contract`  class and  `make`  static imports, such as  `org.springframework.cloud.spec.Contract.make { … }` . You can also provide an import to the  `Contract`  class:  `import org.springframework.cloud.spec.Contract`  and then call  `Contract.make { … }` . |

> Spring Cloud Contract supports defining multiple contracts in a single file.

The following is a complete example of a Groovy contract definition:

```java
org.springframework.cloud.contract.spec.Contract.make {
	request {
		method 'PUT'
		url '/api/12'
		headers {
			header 'Content-Type': 'application/vnd.org.springframework.cloud.contract.verifier.twitter-places-analyzer.v1+json'
		}
		body '''\
		[{
			"created_at": "Sat Jul 26 09:38:57 +0000 2014",
			"id": 492967299297845248,
			"id_str": "492967299297845248",
			"text": "Gonna see you at Warsaw",
			"place":
			{
				"attributes":{},
				"bounding_box":
				{
					"coordinates":
						[[
							[-77.119759,38.791645],
							[-76.909393,38.791645],
							[-76.909393,38.995548],
							[-77.119759,38.995548]
						]],
					"type":"Polygon"
				},
				"country":"United States",
				"country_code":"US",
				"full_name":"Washington, DC",
				"id":"01fbe706f872cb32",
				"name":"Washington",
				"place_type":"city",
				"url": "http://api.twitter.com/1/geo/id/01fbe706f872cb32.json"
			}
		}]
	'''
	}
	response {
		status OK()
	}
}
```

The following is a complete example of a YAML contract definition:

```java
description: Some description
name: some name
priority: 8
ignored: true
request:
url: /foo
queryParameters:
a: b
b: c
method: PUT
headers:
foo: bar
fooReq: baz
body:
foo: bar
matchers:
body:
- path: $.foo
type: by_regex
value: bar
headers:
- key: foo
regex: bar
response:
status: 200
headers:
foo2: bar
foo3: foo33
fooRes: baz
body:
foo2: bar
foo3: baz
nullValue: null
matchers:
body:
- path: $.foo2
type: by_regex
value: bar
- path: $.foo3
type: by_command
value: executeMe($it)
- path: $.nullValue
type: by_null
value: null
headers:
- key: foo2
regex: bar
- key: foo3
command: andMeToo($it)
```

> You can compile contracts to stubs mapping using standalone maven command:  `mvn org.springframework.cloud:spring-cloud-contract-maven-plugin:convert` 

## 95.1 Limitations

> Spring Cloud Contract Verifier does not properly support XML. Please use JSON or help us implement this feature.

> The support for verifying the size of JSON arrays is experimental. If you want to turn it on, please set the value of the following system property to  `true` :  `spring.cloud.contract.verifier.assert.size` . By default, this feature is set to  `false` . You can also provide the  `assertJsonSize`  property in the plugin configuration.

> Because JSON structure can have any form, it can be impossible to parse it properly when using the Groovy DSL and the  `value(consumer(…), producer(…))`  notation in  `GString` . That is why you should use the Groovy Map notation.

## 95.2 Common Top-Level elements

The following sections describe the most common top-level elements:

- [Section 95.2.1, “Description”](multi_contract-dsl.html#contract-dsl-description)

- [Section 95.2.2, “Name”](multi_contract-dsl.html#contract-dsl-name)

- [Section 95.2.3, “Ignoring Contracts”](multi_contract-dsl.html#contract-dsl-ignoring-contracts)

- [Section 95.2.4, “Passing Values from Files”](multi_contract-dsl.html#contract-dsl-passing-values-from-files)

- [Section 95.2.5, “HTTP Top-Level Elements”](multi_contract-dsl.html#contract-dsl-http-top-level-elements)

### 95.2.1 Description

You can add a  `description`  to your contract. The description is arbitrary text. The following code shows an example:

**Groovy DSL.**  

```java
org.springframework.cloud.contract.spec.Contract.make {
			description('''
given:
	An input
when:
	Sth happens
then:
	Output
''')
		}
```

**YAML.**  

```java
description: Some description
name: some name
priority: 8
ignored: true
request:
url: /foo
queryParameters:
a: b
b: c
method: PUT
headers:
foo: bar
fooReq: baz
body:
foo: bar
matchers:
body:
- path: $.foo
type: by_regex
value: bar
headers:
- key: foo
regex: bar
response:
status: 200
headers:
foo2: bar
foo3: foo33
fooRes: baz
body:
foo2: bar
foo3: baz
nullValue: null
matchers:
body:
- path: $.foo2
type: by_regex
value: bar
- path: $.foo3
type: by_command
value: executeMe($it)
- path: $.nullValue
type: by_null
value: null
headers:
- key: foo2
regex: bar
- key: foo3
command: andMeToo($it)
```

### 95.2.2 Name

You can provide a name for your contract. Assume that you provided the following name:  `should register a user` . If you do so, the name of the autogenerated test is  `validate_should_register_a_user` . Also, the name of the stub in a WireMock stub is  `should_register_a_user.json` .

|images/important.png|Important|
|----|----|
|You must ensure that the name does not contain any characters that make the generated test not compile. Also, remember that, if you provide the same name for multiple contracts, your autogenerated tests fail to compile and your generated stubs override each other. |

**Groovy DSL.**  

```java
org.springframework.cloud.contract.spec.Contract.make {
	name("some_special_name")
}
```

**YAML.**  

```java
name: some name
```

### 95.2.3 Ignoring Contracts

If you want to ignore a contract, you can either set a value of ignored contracts in the plugin configuration or set the  `ignored`  property on the contract itself:

**Groovy DSL.**  

```java
org.springframework.cloud.contract.spec.Contract.make {
	ignored()
}
```

**YAML.**  

```java
ignored: true
```

### 95.2.4 Passing Values from Files

Starting with version  `1.2.0` , you can pass values from files. Assume that you have the following resources in our project.

```java
└── src
└── test
└── resources
└── contracts
├── readFromFile.groovy
├── request.json
└── response.json
```

Further assume that your contract is as follows:

**Groovy DSL.**  

```java
import org.springframework.cloud.contract.spec.Contract

Contract.make {
	request {
		method('PUT')
		headers {
			contentType(applicationJson())
		}
		body(file("request.json"))
		url("/1")
	}
	response {
		status OK()
		body(file("response.json"))
		headers {
			contentType(textPlain())
		}
	}
}
```

**YAML.**  

```java
request:
method: GET
url: /foo
bodyFromFile: request.json
response:
status: 200
bodyFromFile: response.json
```

Further assume that the JSON files is as follows:

**request.json** 

```java
{ "status" : "REQUEST" }
```

**response.json** 

```java
{ "status" : "RESPONSE" }
```

When test or stub generation takes place, the contents of the file is passed to the body of a request or a response. The name of the file needs to be a file with location relative to the folder in which the contract lays.

### 95.2.5 HTTP Top-Level Elements

The following methods can be called in the top-level closure of a contract definition.  `request`  and  `response`  are mandatory.  `priority`  is optional.

**Groovy DSL.**  

```java
org.springframework.cloud.contract.spec.Contract.make {
	// Definition of HTTP request part of the contract
	// (this can be a valid request or invalid depending
	// on type of contract being specified).
	request {
		//...
	}

	// Definition of HTTP response part of the contract
	// (a service implementing this contract should respond
	// with following response after receiving request
	// specified in "request" part above).
	response {
		//...
	}

	// Contract priority, which can be used for overriding
	// contracts (1 is highest). Priority is optional.
	priority 1
}
```

**YAML.**  

```java
priority: 8
request:
...
response:
...
```

|images/important.png|Important|
|----|----|
|If you want to make your contract have a  **higher**  value of priority you need to pass a  **lower**  number to the  `priority`  tag / method. E.g.  `priority`  with value  `5`  has  **higher**  priority than  `priority`  with value  `10` . |

## 95.3 Request

The HTTP protocol requires only  **method and url**  to be specified in a request. The same information is mandatory in request definition of the Contract.

**Groovy DSL.**  

```java
org.springframework.cloud.contract.spec.Contract.make {
	request {
		// HTTP request method (GET/POST/PUT/DELETE).
		method 'GET'

		// Path component of request URL is specified as follows.
		urlPath('/users')
	}

	response {
		//...
	}
}
```

**YAML.**  

```java
method: PUT
url: /foo
```

It is possible to specify an absolute rather than relative  `url` , but using  `urlPath`  is the recommended way, as doing so makes the tests  **host-independent** .

**Groovy DSL.**  

```java
org.springframework.cloud.contract.spec.Contract.make {
	request {
		method 'GET'

		// Specifying `url` and `urlPath` in one contract is illegal.
		url('http://localhost:8888/users')
	}

	response {
		//...
	}
}
```

**YAML.**  

```java
request:
method: PUT
urlPath: /foo
```

`request`  may contain  **query parameters** .

**Groovy DSL.**  

```java
org.springframework.cloud.contract.spec.Contract.make {
	request {
		//...

		urlPath('/users') {

			// Each parameter is specified in form
			// `'paramName' : paramValue` where parameter value
			// may be a simple literal or one of matcher functions,
			// all of which are used in this example.
			queryParameters {

				// If a simple literal is used as value
				// default matcher function is used (equalTo)
				parameter 'limit': 100

				// `equalTo` function simply compares passed value
				// using identity operator (==).
				parameter 'filter': equalTo("email")

				// `containing` function matches strings
				// that contains passed substring.
				parameter 'gender': value(consumer(containing("[mf]")), producer('mf'))

				// `matching` function tests parameter
				// against passed regular expression.
				parameter 'offset': value(consumer(matching("[0-9]+")), producer(123))

				// `notMatching` functions tests if parameter
				// does not match passed regular expression.
				parameter 'loginStartsWith': value(consumer(notMatching(".{0,2}")), producer(3))
			}
		}

		//...
	}

	response {
		//...
	}
}
```

**YAML.**  

```java
request:
...
queryParameters:
a: b
b: c
headers:
foo: bar
fooReq: baz
cookies:
foo: bar
fooReq: baz
body:
foo: bar
matchers:
body:
- path: $.foo
type: by_regex
value: bar
headers:
- key: foo
regex: bar
response:
status: 200
headers:
foo2: bar
foo3: foo33
fooRes: baz
body:
foo2: bar
foo3: baz
nullValue: null
matchers:
body:
- path: $.foo2
type: by_regex
value: bar
- path: $.foo3
type: by_command
value: executeMe($it)
- path: $.nullValue
type: by_null
value: null
headers:
- key: foo2
regex: bar
- key: foo3
command: andMeToo($it)
cookies:
- key: foo2
regex: bar
- key: foo3
predefined:
```

`request`  may contain additional  **request headers** , as shown in the following example:

**Groovy DSL.**  

```java
org.springframework.cloud.contract.spec.Contract.make {
	request {
		//...

		// Each header is added in form `'Header-Name' : 'Header-Value'`.
		// there are also some helper methods
		headers {
			header 'key': 'value'
			contentType(applicationJson())
		}

		//...
	}

	response {
		//...
	}
}
```

**YAML.**  

```java
request:
...
headers:
foo: bar
fooReq: baz
```

`request`  may contain additional  **request cookies** , as shown in the following example:

**Groovy DSL.**  

```java
org.springframework.cloud.contract.spec.Contract.make {
	request {
		//...

		// Each Cookies is added in form `'Cookie-Key' : 'Cookie-Value'`.
		// there are also some helper methods
		cookies {
			cookie 'key': 'value'
			cookie('another_key', 'another_value')
		}

		//...
	}

	response {
		//...
	}
}
```

**YAML.**  

```java
request:
...
cookies:
foo: bar
fooReq: baz
```

`request`  may contain a  **request body** :

**Groovy DSL.**  

```java
org.springframework.cloud.contract.spec.Contract.make {
	request {
		//...

		// Currently only JSON format of request body is supported.
		// Format will be determined from a header or body's content.
		body '''{ "login" : "john", "name": "John The Contract" }'''
	}

	response {
		//...
	}
}
```

**YAML.**  

```java
request:
...
body:
foo: bar
```

`request`  may contain  **multipart**  elements. To include multipart elements, use the  `multipart`  method/section, as shown in the following examples

**Groovy DSL.**  

```java
org.springframework.cloud.contract.spec.Contract contractDsl = org.springframework.cloud.contract.spec.Contract.make {
	request {
		method "PUT"
		url "/multipart"
		headers {
			contentType('multipart/form-data;boundary=AaB03x')
		}
		multipart(
				// key (parameter name), value (parameter value) pair
				formParameter: $(c(regex('".+"')), p('"formParameterValue"')),
				someBooleanParameter: $(c(regex(anyBoolean())), p('true')),
				// a named parameter (e.g. with `file` name) that represents file with
				// `name` and `content`. You can also call `named("fileName", "fileContent")`
				file: named(
						// name of the file
						name: $(c(regex(nonEmpty())), p('filename.csv')),
						// content of the file
						content: $(c(regex(nonEmpty())), p('file content')),
						// content type for the part
						contentType: $(c(regex(nonEmpty())), p('application/json')))
		)
	}
	response {
		status OK()
	}
}
org.springframework.cloud.contract.spec.Contract contractDsl = org.springframework.cloud.contract.spec.Contract.make {
	request {
		method "PUT"
		url "/multipart"
		headers {
			contentType('multipart/form-data;boundary=AaB03x')
		}
		multipart(
				file: named(
						name: value(stub(regex('.+')), test('file')),
						content: value(stub(regex('.+')), test([100, 117, 100, 97] as byte[]))
				)
		)
	}
	response {
		status 200
	}
}
```

**YAML.**  

```java
request:
method: PUT
url: /multipart
headers:
Content-Type: multipart/form-data;boundary=AaB03x
multipart:
params:
# key (parameter name), value (parameter value) pair
formParameter: '"formParameterValue"'
someBooleanParameter: true
named:
- paramName: file
fileName: filename.csv
fileContent: file content
matchers:
multipart:
params:
- key: formParameter
regex: ".+"
- key: someBooleanParameter
predefined: any_boolean
named:
- paramName: file
fileName:
predefined: non_empty
fileContent:
predefined: non_empty
response:
status: 200
```

In the preceding example, we define parameters in either of two ways:

**Groovy DSL** 

- Directly, by using the map notation, where the value can be a dynamic property (such as  `formParameter: $(consumer(…), producer(…))` ).

- By using the  `named(…)`  method that lets you set a named parameter. A named parameter can set a  `name`  and  `content` . You can call it either via a method with two arguments, such as  `named("fileName", "fileContent")` , or via a map notation, such as  `named(name: "fileName", content: "fileContent")` .

**YAML** 

- The multipart parameters are set via  `multipart.params`  section

- The named parameters (the  `fileName`  and  `fileContent`  for a given parameter name) can be set via the  `multipart.named`  section. That section contains the  `paramName`  (name of the parameter),  `fileName`  (name of the file),  `fileContent`  (content of the file) fields

- The dynamic bits can be set via the  `matchers.multipart`  section

- for parameters use the  `params`  section that can accept  `regex`  or a  `predefined`  regular expression

- for named params use the  `named`  section where first you define the parameter name via  `paramName`  and then you can pass the parametrization of either  `fileName`  or  `fileContent`  via  `regex`  or a  `predefined`  regular expression

From this contract, the generated test is as follows:

```java
// given:
MockMvcRequestSpecification request = given()
.header("Content-Type", "multipart/form-data;boundary=AaB03x")
.param("formParameter", "\"formParameterValue\"")
.param("someBooleanParameter", "true")
.multiPart("file", "filename.csv", "file content".getBytes());

// when:
ResponseOptions response = given().spec(request)
.put("/multipart");

// then:
assertThat(response.statusCode()).isEqualTo(200);
```

The WireMock stub is as follows:

```java
'''
{
"request" : {
	"url" : "/multipart",
	"method" : "PUT",
	"headers" : {
	  "Content-Type" : {
		"matches" : "multipart/form-data;boundary=AaB03x.*"
	  }
	},
	"bodyPatterns" : [ {
		"matches" : ".*--(.*)\\r\\nContent-Disposition: form-data; name=\\"formParameter\\"\\r\\n(Content-Type: .*\\r\\n)?(Content-Transfer-Encoding: .*\\r\\n)?(Content-Length: \\\\d+\\r\\n)?\\r\\n\\".+\\"\\r\\n--\\\\1.*"
		}, {
			"matches" : ".*--(.*)\\r\\nContent-Disposition: form-data; name=\\"someBooleanParameter\\"\\r\\n(Content-Type: .*\\r\\n)?(Content-Transfer-Encoding: .*\\r\\n)?(Content-Length: \\\\d+\\r\\n)?\\r\\n(true|false)\\r\\n--\\\\1.*"
		}, {
	  "matches" : ".*--(.*)\\r\\nContent-Disposition: form-data; name=\\"file\\"; filename=\\"[\\\\S\\\\s]+\\"\\r\\n(Content-Type: .*\\r\\n)?(Content-Transfer-Encoding: .*\\r\\n)?(Content-Length: \\\\d+\\r\\n)?\\r\\n[\\\\S\\\\s]+\\r\\n--\\\\1.*"
	} ]
},
"response" : {
	"status" : 200,
	"transformers" : [ "response-template", "foo-transformer" ]
}
}
	'''
```

## 95.4 Response

The response must contain an  **HTTP status code**  and may contain other information. The following code shows an example:

**Groovy DSL.**  

```java
org.springframework.cloud.contract.spec.Contract.make {
	request {
		//...
	}
	response {
		// Status code sent by the server
		// in response to request specified above.
		status OK()
	}
}
```

**YAML.**  

```java
response:
...
status: 200
```

Besides status, the response may contain  **headers** ,  **cookies**  and a  **body** , both of which are specified the same way as in the request (see the previous paragraph).

> Via the Groovy DSL you can reference the  `org.springframework.cloud.contract.spec.internal.HttpStatus`  methods to provide a meaningful status instead of a digit. E.g. you can call  `OK()`  for a status  `200`  or  `BAD_REQUEST()`  for  `400` .

## 95.5 Dynamic properties

The contract can contain some dynamic properties: timestamps, IDs, and so on. You do not want to force the consumers to stub their clocks to always return the same value of time so that it gets matched by the stub.

For Groovy DSL you can provide the dynamic parts in your contracts in two ways: pass them directly in the body or set them in a separate section called  `bodyMatchers` .

> Before 2.0.0 these were set using  `testMatchers`  and  `stubMatchers` , check out the [migration guide](https://github.com/spring-cloud/spring-cloud-contract/wiki/Spring-Cloud-Contract-2.0-Migration-Guide) for more information.

For YAML you can only use the  `matchers`  section.

### 95.5.1 Dynamic properties inside the body

|images/important.png|Important|
|----|----|
|This section is valid only for Groovy DSL. Check out the [Section 95.5.7, “Dynamic Properties in the Matchers Sections”](multi_contract-dsl.html#contract-matchers) section for YAML examples of a similar feature. |

You can set the properties inside the body either with the  `value`  method or, if you use the Groovy map notation, with  `$()` . The following example shows how to set dynamic properties with the value method:

```java
value(consumer(...), producer(...))
value(c(...), p(...))
value(stub(...), test(...))
value(client(...), server(...))
```

The following example shows how to set dynamic properties with  `$()` :

```java
$(consumer(...), producer(...))
$(c(...), p(...))
$(stub(...), test(...))
$(client(...), server(...))
```

Both approaches work equally well.  `stub`  and  `client`  methods are aliases over the  `consumer`  method. Subsequent sections take a closer look at what you can do with those values.

### 95.5.2 Regular expressions

|images/important.png|Important|
|----|----|
|This section is valid only for Groovy DSL. Check out the [Section 95.5.7, “Dynamic Properties in the Matchers Sections”](multi_contract-dsl.html#contract-matchers) section for YAML examples of a similar feature. |

You can use regular expressions to write your requests in Contract DSL. Doing so is particularly useful when you want to indicate that a given response should be provided for requests that follow a given pattern. Also, you can use regular expressions when you need to use patterns and not exact values both for your test and your server side tests.

The following example shows how to use regular expressions to write a request:

```java
org.springframework.cloud.contract.spec.Contract.make {
	request {
		method('GET')
		url $(consumer(~/\/[0-9]{2}/), producer('/12'))
	}
	response {
		status OK()
		body(
				id: $(anyNumber()),
				surname: $(
						consumer('Kowalsky'),
						producer(regex('[a-zA-Z]+'))
				),
				name: 'Jan',
				created: $(consumer('2014-02-02 12:23:43'), producer(execute('currentDate(it)'))),
				correlationId: value(consumer('5d1f9fef-e0dc-4f3d-a7e4-72d2220dd827'),
						producer(regex('[a-fA-F0-9]{8}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{12}'))
				)
		)
		headers {
			header 'Content-Type': 'text/plain'
		}
	}
}
```

You can also provide only one side of the communication with a regular expression. If you do so, then the contract engine automatically provides the generated string that matches the provided regular expression. The following code shows an example:

```java
org.springframework.cloud.contract.spec.Contract.make {
	request {
		method 'PUT'
		url value(consumer(regex('/foo/[0-9]{5}')))
		body([
			requestElement: $(consumer(regex('[0-9]{5}')))
		])
		headers {
			header('header', $(consumer(regex('application\\/vnd\\.fraud\\.v1\\+json;.*'))))
		}
	}
	response {
		status OK()
		body([
			responseElement: $(producer(regex('[0-9]{7}')))
		])
		headers {
			contentType("application/vnd.fraud.v1+json")
		}
	}
}
```

In the preceding example, the opposite side of the communication has the respective data generated for request and response.

Spring Cloud Contract comes with a series of predefined regular expressions that you can use in your contracts, as shown in the following example:

```java
protected static final Pattern TRUE_OR_FALSE = Pattern.compile(/(true|false)/)
protected static final Pattern ALPHA_NUMERIC = Pattern.compile('[a-zA-Z0-9]+')
protected static final Pattern ONLY_ALPHA_UNICODE = Pattern.compile(/[\p{L}]*/)
protected static final Pattern NUMBER = Pattern.compile('-?(\\d*\\.\\d+|\\d+)')
protected static final Pattern INTEGER = Pattern.compile('-?(\\d+)')
protected static final Pattern POSITIVE_INT = Pattern.compile('([1-9]\\d*)')
protected static final Pattern DOUBLE = Pattern.compile('-?(\\d*\\.\\d+)')
protected static final Pattern HEX = Pattern.compile('[a-fA-F0-9]+')
protected static final Pattern IP_ADDRESS = Pattern.compile('([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\.([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\.([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\.([01]?\\d\\d?|2[0-4]\\d|25[0-5])')
protected static final Pattern HOSTNAME_PATTERN = Pattern.compile('((http[s]?|ftp):/)/?([^:/\\s]+)(:[0-9]{1,5})?')
protected static final Pattern EMAIL = Pattern.compile('[a-zA-Z0-9._%+-][emailprotected][a-zA-Z0-9.-]+\\.[a-zA-Z]{2,6}')
protected static final Pattern URL = UrlHelper.URL
protected static final Pattern HTTPS_URL = UrlHelper.HTTPS_URL
protected static final Pattern UUID = Pattern.compile('[a-f0-9]{8}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{12}')
protected static final Pattern ANY_DATE = Pattern.compile('(\\d\\d\\d\\d)-(0[1-9]|1[012])-(0[1-9]|[12][0-9]|3[01])')
protected static final Pattern ANY_DATE_TIME = Pattern.compile('([0-9]{4})-(1[0-2]|0[1-9])-(3[01]|0[1-9]|[12][0-9])T(2[0-3]|[01][0-9]):([0-5][0-9]):([0-5][0-9])')
protected static final Pattern ANY_TIME = Pattern.compile('(2[0-3]|[01][0-9]):([0-5][0-9]):([0-5][0-9])')
protected static final Pattern NON_EMPTY = Pattern.compile(/[\S\s]+/)
protected static final Pattern NON_BLANK = Pattern.compile(/^\s*\S[\S\s]*/)
protected static final Pattern ISO8601_WITH_OFFSET = Pattern.compile(/([0-9]{4})-(1[0-2]|0[1-9])-(3[01]|0[1-9]|[12][0-9])T(2[0-3]|[01][0-9]):([0-5][0-9]):([0-5][0-9])(\.\d{3})?(Z|[+-][01]\d:[0-5]\d)/)

protected static Pattern anyOf(String... values){
	return Pattern.compile(values.collect({"^$it\$"}).join("|"))
}

Pattern onlyAlphaUnicode() {
	return ONLY_ALPHA_UNICODE
}

Pattern alphaNumeric() {
	return ALPHA_NUMERIC
}

Pattern number() {
	return NUMBER
}

Pattern positiveInt() {
	return POSITIVE_INT
}

Pattern anyBoolean() {
	return TRUE_OR_FALSE
}

Pattern anInteger() {
	return INTEGER
}

Pattern aDouble() {
	return DOUBLE
}

Pattern ipAddress() {
	return IP_ADDRESS
}

Pattern hostname() {
	return HOSTNAME_PATTERN
}

Pattern email() {
	return EMAIL
}

Pattern url() {
	return URL
}

Pattern httpsUrl() {
	return HTTPS_URL
}

Pattern uuid(){
	return UUID
}

Pattern isoDate() {
	return ANY_DATE
}

Pattern isoDateTime() {
	return ANY_DATE_TIME
}

Pattern isoTime() {
	return ANY_TIME
}

Pattern iso8601WithOffset() {
	return ISO8601_WITH_OFFSET
}

Pattern nonEmpty() {
	return NON_EMPTY
}

Pattern nonBlank() {
	return NON_BLANK
}
```

In your contract, you can use it as shown in the following example:

```java
Contract dslWithOptionalsInString = Contract.make {
	priority 1
	request {
		method POST()
		url '/users/password'
		headers {
			contentType(applicationJson())
		}
		body(
				email: $(consumer(optional(regex(email()))), producer('[emailprotected]')),
				callback_url: $(consumer(regex(hostname())), producer('http://partners.com'))
		)
	}
	response {
		status 404
		headers {
			contentType(applicationJson())
		}
		body(
				code: value(consumer("123123"), producer(optional("123123"))),
				message: "User not found by email = [${value(producer(regex(email())), consumer('[emailprotected]'))}]"
		)
	}
}
```

### 95.5.3 Passing Optional Parameters

|images/important.png|Important|
|----|----|
|This section is valid only for Groovy DSL. Check out the [Section 95.5.7, “Dynamic Properties in the Matchers Sections”](multi_contract-dsl.html#contract-matchers) section for YAML examples of a similar feature. |

It is possible to provide optional parameters in your contract. However, you can provide optional parameters only for the following:

- STUB side of the Request

- TEST side of the Response

The following example shows how to provide optional parameters:

```java
org.springframework.cloud.contract.spec.Contract.make {
	priority 1
	request {
		method 'POST'
		url '/users/password'
		headers {
			contentType(applicationJson())
		}
		body(
				email: $(consumer(optional(regex(email()))), producer('[emailprotected]')),
				callback_url: $(consumer(regex(hostname())), producer('http://partners.com'))
		)
	}
	response {
		status 404
		headers {
			header 'Content-Type': 'application/json'
		}
		body(
				code: value(consumer("123123"), producer(optional("123123")))
		)
	}
}
```

By wrapping a part of the body with the  `optional()`  method, you create a regular expression that must be present 0 or more times.

If you use Spock for, the following test would be generated from the previous example:

```java
"""
given:
def request = given()
.header("Content-Type", "application/json")
.body('''{"email":"[emailprotected]","callback_url":"http://partners.com"}''')

when:
def response = given().spec(request)
.post("/users/password")

then:
response.statusCode == 404
response.header('Content-Type')  == 'application/json'
and:
DocumentContext parsedJson = JsonPath.parse(response.body.asString())
assertThatJson(parsedJson).field("['code']").matches("(123123)?")
"""
```

The following stub would also be generated:

```java
'''
{
"request" : {
	"url" : "/users/password",
	"method" : "POST",
	"bodyPatterns" : [ {
	  "matchesJsonPath" : "$[?(@.['email'] =~ /([a-zA-Z0-9._%+-][emailprotected][a-zA-Z0-9.-]+\\\\.[a-zA-Z]{2,6})?/)]"
	}, {
	  "matchesJsonPath" : "$[?(@.['callback_url'] =~ /((http[s]?|ftp):\\\\/)\\\\/?([^:\\\\/\\\\s]+)(:[0-9]{1,5})?/)]"
	} ],
	"headers" : {
	  "Content-Type" : {
		"equalTo" : "application/json"
	  }
	}
},
"response" : {
	"status" : 404,
	"body" : "{\\"code\\":\\"123123\\",\\"message\\":\\"User not found by email == [not.existing@user.com]\\"}",
	"headers" : {
	  "Content-Type" : "application/json"
	}
},
"priority" : 1
}
'''
```

### 95.5.4 Executing Custom Methods on the Server Side

|images/important.png|Important|
|----|----|
|This section is valid only for Groovy DSL. Check out the [Section 95.5.7, “Dynamic Properties in the Matchers Sections”](multi_contract-dsl.html#contract-matchers) section for YAML examples of a similar feature. |

You can define a method call that executes on the server side during the test. Such a method can be added to the class defined as "baseClassForTests" in the configuration. The following code shows an example of the contract portion of the test case:

```java
org.springframework.cloud.contract.spec.Contract.make {
	request {
		method 'PUT'
		url $(consumer(regex('^/api/[0-9]{2}$')), producer('/api/12'))
		headers {
			header 'Content-Type': 'application/json'
		}
		body '''\
				[{
					"text": "Gonna see you at Warsaw"
				}]
			'''
	}
	response {
		body (
				path: $(consumer('/api/12'), producer(regex('^/api/[0-9]{2}$'))),
				correlationId: $(consumer('1223456'), producer(execute('isProperCorrelationId($it)')))
		)
		status OK()
	}
}
```

The following code shows the base class portion of the test case:

```java
abstract class BaseMockMvcSpec extends Specification {

	def setup() {
		RestAssuredMockMvc.standaloneSetup(new PairIdController())
	}

	void isProperCorrelationId(Integer correlationId) {
		assert correlationId == 123456
	}

	void isEmpty(String value) {
		assert value == null
	}

}
```

|images/important.png|Important|
|----|----|
|You cannot use both a String and  `execute`  to perform concatenation. For example, calling  `header('Authorization', 'Bearer ' + execute('authToken()'))`  leads to improper results. Instead, call  `header('Authorization', execute('authToken()'))`  and ensure that the  `authToken()`  method returns everything you need. |

The type of the object read from the JSON can be one of the following, depending on the JSON path:

-  `String` : If you point to a  `String`  value in the JSON.

-  `JSONArray` : If you point to a  `List`  in the JSON.

-  `Map` : If you point to a  `Map`  in the JSON.

-  `Number` : If you point to  `Integer` ,  `Double`  etc. in the JSON.

-  `Boolean` : If you point to a  `Boolean`  in the JSON.

In the request part of the contract, you can specify that the  `body`  should be taken from a method.

|images/important.png|Important|
|----|----|
|You must provide both the consumer and the producer side. The  `execute`  part is applied for the whole body - not for parts of it. |

The following example shows how to read an object from JSON:

```java
Contract contractDsl = Contract.make {
	request {
		method 'GET'
		url '/something'
		body(
				$(c("foo"), p(execute("hashCode()")))
		)
	}
	response {
		status OK()
	}
}
```

The preceding example results in calling the  `hashCode()`  method in the request body. It should resemble the following code:

```java
// given:
MockMvcRequestSpecification request = given()
.body(hashCode());

// when:
ResponseOptions response = given().spec(request)
.get("/something");

// then:
assertThat(response.statusCode()).isEqualTo(200);
```

### 95.5.5 Referencing the Request from the Response

The best situation is to provide fixed values, but sometimes you need to reference a request in your response.

If you’re writing contracts using Groovy DSL, you can use the  `fromRequest()`  method, which lets you reference a bunch of elements from the HTTP request. You can use the following options:

-  `fromRequest().url()` : Returns the request URL and query parameters.

-  `fromRequest().query(String key)` : Returns the first query parameter with a given name.

-  `fromRequest().query(String key, int index)` : Returns the nth query parameter with a given name.

-  `fromRequest().path()` : Returns the full path.

-  `fromRequest().path(int index)` : Returns the nth path element.

-  `fromRequest().header(String key)` : Returns the first header with a given name.

-  `fromRequest().header(String key, int index)` : Returns the nth header with a given name.

-  `fromRequest().body()` : Returns the full request body.

-  `fromRequest().body(String jsonPath)` : Returns the element from the request that matches the JSON Path.

If you’re using the YAML contract definition you have to use the [Handlebars](http://handlebarsjs.com/)  `{{{ }}}`  notation with custom, Spring Cloud Contract functions to achieve this.

-  `{{{ request.url }}}` : Returns the request URL and query parameters.

-  `{{{ request.query.key.[index] }}}` : Returns the nth query parameter with a given name. E.g. for key  `foo` , first entry  `{{{ request.query.foo.[0] }}}` 

-  `{{{ request.path }}}` : Returns the full path.

-  `{{{ request.path.[index] }}}` : Returns the nth path element. E.g. for first entry  ``` {{{ request.path.[0] }}}

-  `{{{ request.headers.key }}}` : Returns the first header with a given name.

-  `{{{ request.headers.key.[index] }}}` : Returns the nth header with a given name.

-  `{{{ request.body }}}` : Returns the full request body.

-  `{{{ jsonpath this 'your.json.path' }}}` : Returns the element from the request that matches the JSON Path. E.g. for json path  `$.foo`  -  `{{{ jsonpath this '$.foo' }}}` 

Consider the following contract:

**Groovy DSL.**  

```java
Contract contractDsl = Contract.make {
	request {
		method 'GET'
		url('/api/v1/xxxx') {
			queryParameters {
				parameter("foo", "bar")
				parameter("foo", "bar2")
			}
		}
		headers {
			header(authorization(), "secret")
			header(authorization(), "secret2")
		}
		body(foo: "bar", baz: 5)
	}
	response {
		status OK()
		headers {
			header(authorization(), "foo ${fromRequest().header(authorization())} bar")
		}
		body(
				url: fromRequest().url(),
				path: fromRequest().path(),
				pathIndex: fromRequest().path(1),
				param: fromRequest().query("foo"),
				paramIndex: fromRequest().query("foo", 1),
				authorization: fromRequest().header("Authorization"),
				authorization2: fromRequest().header("Authorization", 1),
				fullBody: fromRequest().body(),
				responseFoo: fromRequest().body('$.foo'),
				responseBaz: fromRequest().body('$.baz'),
				responseBaz2: "Bla bla ${fromRequest().body('$.foo')} bla bla"
		)
	}
}
Contract contractDsl = Contract.make {
	request {
		method 'GET'
		url('/api/v1/xxxx') {
			queryParameters {
				parameter("foo", "bar")
				parameter("foo", "bar2")
			}
		}
		headers {
			header(authorization(), "secret")
			header(authorization(), "secret2")
		}
		body(foo: "bar", baz: 5)
	}
	response {
		status OK()
		headers {
			contentType(applicationJson())
		}
		body('''
				{
					"responseFoo": "{{{ jsonPath request.body '$.foo' }}}",
					"responseBaz": {{{ jsonPath request.body '$.baz' }}},
					"responseBaz2": "Bla bla {{{ jsonPath request.body '$.foo' }}} bla bla"
				}
		'''.toString())
	}
}
```

**YAML.**  

```java
request:
method: GET
url: /api/v1/xxxx
queryParameters:
foo:
- bar
- bar2
headers:
Authorization:
- secret
- secret2
body:
foo: bar
baz: 5
response:
status: 200
headers:
Authorization: "foo {{{ request.headers.Authorization.0 }}} bar"
body:
url: "{{{ request.url }}}"
path: "{{{ request.path }}}"
pathIndex: "{{{ request.path.1 }}}"
param: "{{{ request.query.foo }}}"
paramIndex: "{{{ request.query.foo.1 }}}"
authorization: "{{{ request.headers.Authorization.0 }}}"
authorization2: "{{{ request.headers.Authorization.1 }}"
fullBody: "{{{ request.body }}}"
responseFoo: "{{{ jsonpath this '$.foo' }}}"
responseBaz: "{{{ jsonpath this '$.baz' }}}"
responseBaz2: "Bla bla {{{ jsonpath this '$.foo' }}} bla bla"
```

Running a JUnit test generation leads to a test that resembles the following example:

```java
// given:
MockMvcRequestSpecification request = given()
.header("Authorization", "secret")
.header("Authorization", "secret2")
.body("{\"foo\":\"bar\",\"baz\":5}");

// when:
ResponseOptions response = given().spec(request)
.queryParam("foo","bar")
.queryParam("foo","bar2")
.get("/api/v1/xxxx");

// then:
assertThat(response.statusCode()).isEqualTo(200);
assertThat(response.header("Authorization")).isEqualTo("foo secret bar");
// and:
DocumentContext parsedJson = JsonPath.parse(response.getBody().asString());
assertThatJson(parsedJson).field("['fullBody']").isEqualTo("{\"foo\":\"bar\",\"baz\":5}");
assertThatJson(parsedJson).field("['authorization']").isEqualTo("secret");
assertThatJson(parsedJson).field("['authorization2']").isEqualTo("secret2");
assertThatJson(parsedJson).field("['path']").isEqualTo("/api/v1/xxxx");
assertThatJson(parsedJson).field("['param']").isEqualTo("bar");
assertThatJson(parsedJson).field("['paramIndex']").isEqualTo("bar2");
assertThatJson(parsedJson).field("['pathIndex']").isEqualTo("v1");
assertThatJson(parsedJson).field("['responseBaz']").isEqualTo(5);
assertThatJson(parsedJson).field("['responseFoo']").isEqualTo("bar");
assertThatJson(parsedJson).field("['url']").isEqualTo("/api/v1/xxxx?foo=bar&foo=bar2");
assertThatJson(parsedJson).field("['responseBaz2']").isEqualTo("Bla bla bar bla bla");
```

As you can see, elements from the request have been properly referenced in the response.

The generated WireMock stub should resemble the following example:

```java
{
"request" : {
"urlPath" : "/api/v1/xxxx",
"method" : "POST",
"headers" : {
"Authorization" : {
"equalTo" : "secret2"
}
},
"queryParameters" : {
"foo" : {
"equalTo" : "bar2"
}
},
"bodyPatterns" : [ {
"matchesJsonPath" : "$[?(@.['baz'] == 5)]"
}, {
"matchesJsonPath" : "$[?(@.['foo'] == 'bar')]"
} ]
},
"response" : {
"status" : 200,
"body" : "{\"authorization\":\"{{{request.headers.Authorization.[0]}}}\",\"path\":\"{{{request.path}}}\",\"responseBaz\":{{{jsonpath this '$.baz'}}} ,\"param\":\"{{{request.query.foo.[0]}}}\",\"pathIndex\":\"{{{request.path.[1]}}}\",\"responseBaz2\":\"Bla bla {{{jsonpath this '$.foo'}}} bla bla\",\"responseFoo\":\"{{{jsonpath this '$.foo'}}}\",\"authorization2\":\"{{{request.headers.Authorization.[1]}}}\",\"fullBody\":\"{{{escapejsonbody}}}\",\"url\":\"{{{request.url}}}\",\"paramIndex\":\"{{{request.query.foo.[1]}}}\"}",
"headers" : {
"Authorization" : "{{{request.headers.Authorization.[0]}}};foo"
},
"transformers" : [ "response-template" ]
}
}
```

Sending a request such as the one presented in the  `request`  part of the contract results in sending the following response body:

```java
{
"url" : "/api/v1/xxxx?foo=bar&foo=bar2",
"path" : "/api/v1/xxxx",
"pathIndex" : "v1",
"param" : "bar",
"paramIndex" : "bar2",
"authorization" : "secret",
"authorization2" : "secret2",
"fullBody" : "{\"foo\":\"bar\",\"baz\":5}",
"responseFoo" : "bar",
"responseBaz" : 5,
"responseBaz2" : "Bla bla bar bla bla"
}
```

|images/important.png|Important|
|----|----|
|This feature works only with WireMock having a version greater than or equal to 2.5.1. The Spring Cloud Contract Verifier uses WireMock’s  `response-template`  response transformer. It uses Handlebars to convert the Mustache  `{{{ }}}`  templates into proper values. Additionally, it registers two helper functions: |

-  `escapejsonbody` : Escapes the request body in a format that can be embedded in a JSON.

-  `jsonpath` : For a given parameter, find an object in the request body.

### 95.5.6 Registering Your Own WireMock Extension

WireMock lets you register custom extensions. By default, Spring Cloud Contract registers the transformer, which lets you reference a request from a response. If you want to provide your own extensions, you can register an implementation of the  `org.springframework.cloud.contract.verifier.dsl.wiremock.WireMockExtensions`  interface. Since we use the spring.factories extension approach, you can create an entry in  `META-INF/spring.factories`  file similar to the following:

```java
org.springframework.cloud.contract.verifier.dsl.wiremock.WireMockExtensions=\
org.springframework.cloud.contract.stubrunner.provider.wiremock.TestWireMockExtensions
org.springframework.cloud.contract.spec.ContractConverter=\
org.springframework.cloud.contract.stubrunner.TestCustomYamlContractConverter
```

The following is an example of a custom extension:

**TestWireMockExtensions.groovy.**  

```java
package org.springframework.cloud.contract.verifier.dsl.wiremock

import com.github.tomakehurst.wiremock.extension.Extension

/**
* Extension that registers the default transformer and the custom one
*/
class TestWireMockExtensions implements WireMockExtensions {
	@Override
	List<Extension> extensions() {
		return [
				new DefaultResponseTransformer(),
				new CustomExtension()
		]
	}
}

class CustomExtension implements Extension {

	@Override
	String getName() {
		return "foo-transformer"
	}
}
```

|images/important.png|Important|
|----|----|
|Remember to override the  `applyGlobally()`  method and set it to  `false`  if you want the transformation to be applied only for a mapping that explicitly requires it. |

### 95.5.7 Dynamic Properties in the Matchers Sections

If you work with [Pact](https://docs.pact.io/), the following discussion may seem familiar. Quite a few users are used to having a separation between the body and setting the dynamic parts of a contract.

You can use the  `bodyMatchers`  section for two reasons:

- Define the dynamic values that should end up in a stub. You can set it in the  `request`  or  `inputMessage`  part of your contract.

- Verify the result of your test. This section is present in the  `response`  or  `outputMessage`  side of the contract.

Currently, Spring Cloud Contract Verifier supports only JSON Path-based matchers with the following matching possibilities:

**Groovy DSL** 

- For the stubs(in tests on the Consumer’s side):

-  `byEquality()` : The value taken from the consumer’s request via the provided JSON Path must be equal to the value provided in the contract.

-  `byRegex(…)` : The value taken from the consumer’s request via the provided JSON Path must match the regex.

-  `byDate()` : The value taken from the consumer’s request via the provided JSON Path must match the regex for an ISO Date value.

-  `byTimestamp()` : The value taken from the consumer’s request via the provided JSON Path must match the regex for an ISO DateTime value.

-  `byTime()` : The value taken from the consumer’s request via the provided JSON Path must match the regex for an ISO Time value.

- For the verification(in generated tests on the Producer’s side):

-  `byEquality()` : The value taken from the producer’s response via the provided JSON Path must be equal to the provided value in the contract.

-  `byRegex(…)` : The value taken from the producer’s response via the provided JSON Path must match the regex.

-  `byDate()` : The value taken from the producer’s response via the provided JSON Path must match the regex for an ISO Date value.

-  `byTimestamp()` : The value taken from the producer’s response via the provided JSON Path must match the regex for an ISO DateTime value.

-  `byTime()` : The value taken from the producer’s response via the provided JSON Path must match the regex for an ISO Time value.

-  `byType()` : The value taken from the producer’s response via the provided JSON Path needs to be of the same type as the type defined in the body of the response in the contract.  `byType`  can take a closure, in which you can set  `minOccurrence`  and  `maxOccurrence` . That way, you can assert the size of the flattened collection. To check the size of an unflattened collection, use a custom method with the  `byCommand(…)`  testMatcher.

-  `byCommand(…)` : The value taken from the producer’s response via the provided JSON Path is passed as an input to the custom method that you provide. For example,  `byCommand('foo($it)')`  results in calling a  `foo`  method to which the value matching the JSON Path gets passed. The type of the object read from the JSON can be one of the following, depending on the JSON path:

-  `String` : If you point to a  `String`  value.

-  `JSONArray` : If you point to a  `List` .

-  `Map` : If you point to a  `Map` .

-  `Number` : If you point to  `Integer` ,  `Double` , or other kind of number.

-  `Boolean` : If you point to a  `Boolean` .

-  `byNull()` : The value taken from the response via the provided JSON Path must be null

**YAML.** Please read the Groovy section for detailed explanation of what the types mean

For YAML the structure of a matcher looks like this

```java
- path: $.foo
type: by_regex
value: bar
```

Or if you want to use one of the predefined regular expressions  `[only_alpha_unicode, number, any_boolean, ip_address, hostname, email, url, uuid, iso_date, iso_date_time, iso_time, iso_8601_with_offset, non_empty, non_blank]` :

```java
- path: $.foo
type: by_regex
predefined: only_alpha_unicode
```

Below you can find the allowed list of `type`s.

- For  `stubMatchers` :

-  `by_equality` 

-  `by_regex` 

-  `by_date` 

-  `by_timestamp` 

-  `by_time` 

- For  `testMatchers` :

-  `by_equality` 

-  `by_regex` 

-  `by_date` 

-  `by_timestamp` 

-  `by_time` 

-  `by_type` 

- there are 2 additional fields accepted:  `minOccurrence`  and  `maxOccurrence` .

-  `by_command` 

-  `by_null` 

Consider the following example:

**Groovy DSL.**  

```java
Contract contractDsl = Contract.make {
	request {
		method 'GET'
		urlPath '/get'
		body([
				duck: 123,
				alpha: "abc",
				number: 123,
				aBoolean: true,
				date: "2017-01-01",
				dateTime: "2017-01-01T01:23:45",
				time: "01:02:34",
				valueWithoutAMatcher: "foo",
				valueWithTypeMatch: "string",
				key: [
						'complex.key' : 'foo'
				]
		])
		bodyMatchers {
			jsonPath('$.duck', byRegex("[0-9]{3}"))
			jsonPath('$.duck', byEquality())
			jsonPath('$.alpha', byRegex(onlyAlphaUnicode()))
			jsonPath('$.alpha', byEquality())
			jsonPath('$.number', byRegex(number()))
			jsonPath('$.aBoolean', byRegex(anyBoolean()))
			jsonPath('$.date', byDate())
			jsonPath('$.dateTime', byTimestamp())
			jsonPath('$.time', byTime())
			jsonPath("\$.['key'].['complex.key']", byEquality())
		}
		headers {
			contentType(applicationJson())
		}
	}
	response {
		status OK()
		body([
				duck: 123,
				alpha: "abc",
				number: 123,
				positiveInteger: 1234567890,
				negativeInteger: -1234567890,
				positiveDecimalNumber: 123.4567890,
				negativeDecimalNumber: -123.4567890,
				aBoolean: true,
				date: "2017-01-01",
				dateTime: "2017-01-01T01:23:45",
				time: "01:02:34",
				valueWithoutAMatcher: "foo",
				valueWithTypeMatch: "string",
				valueWithMin: [
					1,2,3
				],
				valueWithMax: [
					1,2,3
				],
				valueWithMinMax: [
					1,2,3
				],
				valueWithMinEmpty: [],
				valueWithMaxEmpty: [],
				key: [
				        'complex.key' : 'foo'
				],
				nullValue: null
		])
		bodyMatchers {
			// asserts the jsonpath value against manual regex
			jsonPath('$.duck', byRegex("[0-9]{3}"))
			// asserts the jsonpath value against the provided value
			jsonPath('$.duck', byEquality())
			// asserts the jsonpath value against some default regex
			jsonPath('$.alpha', byRegex(onlyAlphaUnicode()))
			jsonPath('$.alpha', byEquality())
			jsonPath('$.number', byRegex(number()))
			jsonPath('$.positiveInteger', byRegex(anInteger()))
			jsonPath('$.negativeInteger', byRegex(anInteger()))
			jsonPath('$.positiveDecimalNumber', byRegex(aDouble()))
			jsonPath('$.negativeDecimalNumber', byRegex(aDouble()))
			jsonPath('$.aBoolean', byRegex(anyBoolean()))
			// asserts vs inbuilt time related regex
			jsonPath('$.date', byDate())
			jsonPath('$.dateTime', byTimestamp())
			jsonPath('$.time', byTime())
			// asserts that the resulting type is the same as in response body
			jsonPath('$.valueWithTypeMatch', byType())
			jsonPath('$.valueWithMin', byType {
				// results in verification of size of array (min 1)
				minOccurrence(1)
			})
			jsonPath('$.valueWithMax', byType {
				// results in verification of size of array (max 3)
				maxOccurrence(3)
			})
			jsonPath('$.valueWithMinMax', byType {
				// results in verification of size of array (min 1 & max 3)
				minOccurrence(1)
				maxOccurrence(3)
			})
			jsonPath('$.valueWithMinEmpty', byType {
				// results in verification of size of array (min 0)
				minOccurrence(0)
			})
			jsonPath('$.valueWithMaxEmpty', byType {
				// results in verification of size of array (max 0)
				maxOccurrence(0)
			})
			// will execute a method `assertThatValueIsANumber`
			jsonPath('$.duck', byCommand('assertThatValueIsANumber($it)'))
			jsonPath("\$.['key'].['complex.key']", byEquality())
			jsonPath('$.nullValue', byNull())
		}
		headers {
			contentType(applicationJson())
			header('Some-Header', $(c('someValue'), p(regex('[a-zA-Z]{9}'))))
		}
	}
}
```

**YAML.**  

```java
request:
method: GET
urlPath: /get
body:
duck: 123
alpha: "abc"
number: 123
aBoolean: true
date: "2017-01-01"
dateTime: "2017-01-01T01:23:45"
time: "01:02:34"
valueWithoutAMatcher: "foo"
valueWithTypeMatch: "string"
key:
"complex.key": 'foo'
nullValue: null
matchers:
headers:
- key: Content-Type
regex: "application/json.*"
body:
- path: $.duck
type: by_regex
value: "[0-9]{3}"
- path: $.duck
type: by_equality
- path: $.alpha
type: by_regex
predefined: only_alpha_unicode
- path: $.alpha
type: by_equality
- path: $.number
type: by_regex
predefined: number
- path: $.aBoolean
type: by_regex
predefined: any_boolean
- path: $.date
type: by_date
- path: $.dateTime
type: by_timestamp
- path: $.time
type: by_time
- path: "$.['key'].['complex.key']"
type: by_equality
- path: $.nullvalue
type: by_null
headers:
Content-Type: application/json
response:
status: 200
body:
duck: 123
alpha: "abc"
number: 123
aBoolean: true
date: "2017-01-01"
dateTime: "2017-01-01T01:23:45"
time: "01:02:34"
valueWithoutAMatcher: "foo"
valueWithTypeMatch: "string"
valueWithMin:
- 1
- 2
- 3
valueWithMax:
- 1
- 2
- 3
valueWithMinMax:
- 1
- 2
- 3
valueWithMinEmpty: []
valueWithMaxEmpty: []
key:
'complex.key' : 'foo'
nulValue: null
matchers:
headers:
- key: Content-Type
regex: "application/json.*"
body:
- path: $.duck
type: by_regex
value: "[0-9]{3}"
- path: $.duck
type: by_equality
- path: $.alpha
type: by_regex
predefined: only_alpha_unicode
- path: $.alpha
type: by_equality
- path: $.number
type: by_regex
predefined: number
- path: $.aBoolean
type: by_regex
predefined: any_boolean
- path: $.date
type: by_date
- path: $.dateTime
type: by_timestamp
- path: $.time
type: by_time
- path: $.valueWithTypeMatch
type: by_type
- path: $.valueWithMin
type: by_type
minOccurrence: 1
- path: $.valueWithMax
type: by_type
maxOccurrence: 3
- path: $.valueWithMinMax
type: by_type
minOccurrence: 1
maxOccurrence: 3
- path: $.valueWithMinEmpty
type: by_type
minOccurrence: 0
- path: $.valueWithMaxEmpty
type: by_type
maxOccurrence: 0
- path: $.duck
type: by_command
value: assertThatValueIsANumber($it)
- path: $.nullValue
type: by_null
value: null
headers:
Content-Type: application/json
```

In the preceding example, you can see the dynamic portions of the contract in the  `matchers`  sections. For the request part, you can see that, for all fields but  `valueWithoutAMatcher` , the values of the regular expressions that the stub should contain are explicitly set. For the  `valueWithoutAMatcher` , the verification takes place in the same way as without the use of matchers. In that case, the test performs an equality check.

For the response side in the  `bodyMatchers`  section, we define the dynamic parts in a similar manner. The only difference is that the  `byType`  matchers are also present. The verifier engine checks four fields to verify whether the response from the test has a value for which the JSON path matches the given field, is of the same type as the one defined in the response body, and passes the following check (based on the method being called):

- For  `$.valueWithTypeMatch` , the engine checks whether the type is the same.

- For  `$.valueWithMin` , the engine check the type and asserts whether the size is greater than or equal to the minimum occurrence.

- For  `$.valueWithMax` , the engine checks the type and asserts whether the size is smaller than or equal to the maximum occurrence.

- For  `$.valueWithMinMax` , the engine checks the type and asserts whether the size is between the min and maximum occurrence.

The resulting test would resemble the following example (note that an  `and`  section separates the autogenerated assertions and the assertion from matchers):

```java
// given:
MockMvcRequestSpecification request = given()
.header("Content-Type", "application/json")
.body("{\"duck\":123,\"alpha\":\"abc\",\"number\":123,\"aBoolean\":true,\"date\":\"2017-01-01\",\"dateTime\":\"2017-01-01T01:23:45\",\"time\":\"01:02:34\",\"valueWithoutAMatcher\":\"foo\",\"valueWithTypeMatch\":\"string\",\"key\":{\"complex.key\":\"foo\"}}");

// when:
ResponseOptions response = given().spec(request)
.get("/get");

// then:
assertThat(response.statusCode()).isEqualTo(200);
assertThat(response.header("Content-Type")).matches("application/json.*");
// and:
DocumentContext parsedJson = JsonPath.parse(response.getBody().asString());
assertThatJson(parsedJson).field("['valueWithoutAMatcher']").isEqualTo("foo");
// and:
assertThat(parsedJson.read("$.duck", String.class)).matches("[0-9]{3}");
assertThat(parsedJson.read("$.duck", Integer.class)).isEqualTo(123);
assertThat(parsedJson.read("$.alpha", String.class)).matches("[\\p{L}]*");
assertThat(parsedJson.read("$.alpha", String.class)).isEqualTo("abc");
assertThat(parsedJson.read("$.number", String.class)).matches("-?(\\d*\\.\\d+|\\d+)");
assertThat(parsedJson.read("$.aBoolean", String.class)).matches("(true|false)");
assertThat(parsedJson.read("$.date", String.class)).matches("(\\d\\d\\d\\d)-(0[1-9]|1[012])-(0[1-9]|[12][0-9]|3[01])");
assertThat(parsedJson.read("$.dateTime", String.class)).matches("([0-9]{4})-(1[0-2]|0[1-9])-(3[01]|0[1-9]|[12][0-9])T(2[0-3]|[01][0-9]):([0-5][0-9]):([0-5][0-9])");
assertThat(parsedJson.read("$.time", String.class)).matches("(2[0-3]|[01][0-9]):([0-5][0-9]):([0-5][0-9])");
assertThat((Object) parsedJson.read("$.valueWithTypeMatch")).isInstanceOf(java.lang.String.class);
assertThat((Object) parsedJson.read("$.valueWithMin")).isInstanceOf(java.util.List.class);
assertThat((java.lang.Iterable) parsedJson.read("$.valueWithMin", java.util.Collection.class)).as("$.valueWithMin").hasSizeGreaterThanOrEqualTo(1);
assertThat((Object) parsedJson.read("$.valueWithMax")).isInstanceOf(java.util.List.class);
assertThat((java.lang.Iterable) parsedJson.read("$.valueWithMax", java.util.Collection.class)).as("$.valueWithMax").hasSizeLessThanOrEqualTo(3);
assertThat((Object) parsedJson.read("$.valueWithMinMax")).isInstanceOf(java.util.List.class);
assertThat((java.lang.Iterable) parsedJson.read("$.valueWithMinMax", java.util.Collection.class)).as("$.valueWithMinMax").hasSizeBetween(1, 3);
assertThat((Object) parsedJson.read("$.valueWithMinEmpty")).isInstanceOf(java.util.List.class);
assertThat((java.lang.Iterable) parsedJson.read("$.valueWithMinEmpty", java.util.Collection.class)).as("$.valueWithMinEmpty").hasSizeGreaterThanOrEqualTo(0);
assertThat((Object) parsedJson.read("$.valueWithMaxEmpty")).isInstanceOf(java.util.List.class);
assertThat((java.lang.Iterable) parsedJson.read("$.valueWithMaxEmpty", java.util.Collection.class)).as("$.valueWithMaxEmpty").hasSizeLessThanOrEqualTo(0);
assertThatValueIsANumber(parsedJson.read("$.duck"));
assertThat(parsedJson.read("$.['key'].['complex.key']", String.class)).isEqualTo("foo");
```

|images/important.png|Important|
|----|----|
|Notice that, for the  `byCommand`  method, the example calls the  `assertThatValueIsANumber` . This method must be defined in the test base class or be statically imported to your tests. Notice that the  `byCommand`  call was converted to  `assertThatValueIsANumber(parsedJson.read("$.duck"));` . That means that the engine took the method name and passed the proper JSON path as a parameter to it. |

The resulting WireMock stub is in the following example:

```java
'''
{
"request" : {
	"urlPath" : "/get",
	"method" : "POST",
	"headers" : {
	  "Content-Type" : {
		"matches" : "application/json.*"
	  }
	},
	"bodyPatterns" : [ {
	  "matchesJsonPath" : "$[?(@.['valueWithoutAMatcher'] == 'foo')]"
	}, {
	  "matchesJsonPath" : "$[?(@.['valueWithTypeMatch'] == 'string')]"
	}, {
	  "matchesJsonPath" : "$.['list'].['some'].['nested'][?(@.['anothervalue'] == 4)]"
	}, {
	  "matchesJsonPath" : "$.['list'].['someother'].['nested'][?(@.['anothervalue'] == 4)]"
	}, {
	  "matchesJsonPath" : "$.['list'].['someother'].['nested'][?(@.['json'] == 'with value')]"
	}, {
	  "matchesJsonPath" : "$[?(@.duck =~ /([0-9]{3})/)]"
	}, {
	  "matchesJsonPath" : "$[?(@.duck == 123)]"
	}, {
	  "matchesJsonPath" : "$[?(@.alpha =~ /([\\\\p{L}]*)/)]"
	}, {
	  "matchesJsonPath" : "$[?(@.alpha == 'abc')]"
	}, {
	  "matchesJsonPath" : "$[?(@.number =~ /(-?(\\\\d*\\\\.\\\\d+|\\\\d+))/)]"
	}, {
	  "matchesJsonPath" : "$[?(@.aBoolean =~ /((true|false))/)]"
	}, {
	  "matchesJsonPath" : "$[?(@.date =~ /((\\\\d\\\\d\\\\d\\\\d)-(0[1-9]|1[012])-(0[1-9]|[12][0-9]|3[01]))/)]"
	}, {
	  "matchesJsonPath" : "$[?(@.dateTime =~ /(([0-9]{4})-(1[0-2]|0[1-9])-(3[01]|0[1-9]|[12][0-9])T(2[0-3]|[01][0-9]):([0-5][0-9]):([0-5][0-9]))/)]"
	}, {
	  "matchesJsonPath" : "$[?(@.time =~ /((2[0-3]|[01][0-9]):([0-5][0-9]):([0-5][0-9]))/)]"
	}, {
	  "matchesJsonPath" : "$.list.some.nested[?(@.json =~ /(.*)/)]"
	} ]
},
"response" : {
	"status" : 200,
	"body" : "{\\"date\\":\\"2017-01-01\\",\\"dateTime\\":\\"2017-01-01T01:23:45\\",\\"number\\":123,\\"aBoolean\\":true,\\"duck\\":123,\\"alpha\\":\\"abc\\",\\"valueWithMin\\":[1,2,3],\\"time\\":\\"01:02:34\\",\\"valueWithTypeMatch\\":\\"string\\",\\"valueWithMax\\":[1,2,3],\\"valueWithMinMax\\":[1,2,3],\\"valueWithoutAMatcher\\":\\"foo\\"}",
	"headers" : {
	  "Content-Type" : "application/json"
	}
}
}
'''
```

|images/important.png|Important|
|----|----|
|If you use a  `matcher` , then the part of the request and response that the  `matcher`  addresses with the JSON Path gets removed from the assertion. In the case of verifying a collection, you must create matchers for  **all**  the elements of the collection. |

Consider the following example:

```java
Contract.make {
request {
method 'GET'
url("/foo")
}
response {
status OK()
body(events: [[
operation          : 'EXPORT',
eventId            : '16f1ed75-0bcc-4f0d-a04d-3121798faf99',
status             : 'OK'
], [
operation          : 'INPUT_PROCESSING',
eventId            : '3bb4ac82-6652-462f-b6d1-75e424a0024a',
status             : 'OK'
]
]
)
bodyMatchers {
jsonPath('$.events[0].operation', byRegex('.+'))
jsonPath('$.events[0].eventId', byRegex('^([a-fA-F0-9]{8}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{12})$'))
jsonPath('$.events[0].status', byRegex('.+'))
}
}
}
```

The preceding code leads to creating the following test (the code block shows only the assertion section):

```java
and:
	DocumentContext parsedJson = JsonPath.parse(response.body.asString())
	assertThatJson(parsedJson).array("['events']").contains("['eventId']").isEqualTo("16f1ed75-0bcc-4f0d-a04d-3121798faf99")
	assertThatJson(parsedJson).array("['events']").contains("['operation']").isEqualTo("EXPORT")
	assertThatJson(parsedJson).array("['events']").contains("['operation']").isEqualTo("INPUT_PROCESSING")
	assertThatJson(parsedJson).array("['events']").contains("['eventId']").isEqualTo("3bb4ac82-6652-462f-b6d1-75e424a0024a")
	assertThatJson(parsedJson).array("['events']").contains("['status']").isEqualTo("OK")
and:
	assertThat(parsedJson.read("\$.events[0].operation", String.class)).matches(".+")
	assertThat(parsedJson.read("\$.events[0].eventId", String.class)).matches("^([a-fA-F0-9]{8}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{12})\$")
	assertThat(parsedJson.read("\$.events[0].status", String.class)).matches(".+")
```

As you can see, the assertion is malformed. Only the first element of the array got asserted. In order to fix this, you should apply the assertion to the whole  `$.events`  collection and assert it with the  `byCommand(…)`  method.

## 95.6 JAX-RS Support

The Spring Cloud Contract Verifier supports the JAX-RS 2 Client API. The base class needs to define  `protected WebTarget webTarget`  and server initialization. The only option for testing JAX-RS API is to start a web server. Also, a request with a body needs to have a content type set. Otherwise, the default of  `application/octet-stream`  gets used.

In order to use JAX-RS mode, use the following settings:

```java
testMode == 'JAXRSCLIENT'
```

The following example shows a generated test API:

```java
'''
// when:
Response response = webTarget
.path("/users")
.queryParam("limit", "10")
.queryParam("offset", "20")
.queryParam("filter", "email")
.queryParam("sort", "name")
.queryParam("search", "55")
.queryParam("age", "99")
.queryParam("name", "Denis.Stepanov")
.queryParam("email", "[emailprotected]")
.request()
.method("GET");

String responseAsString = response.readEntity(String.class);

// then:
assertThat(response.getStatus()).isEqualTo(200);
// and:
DocumentContext parsedJson = JsonPath.parse(responseAsString);
assertThatJson(parsedJson).field("['property1']").isEqualTo("a");
'''
```

## 95.7 Async Support

If you’re using asynchronous communication on the server side (your controllers are returning  `Callable` ,  `DeferredResult` , and so on), then, inside your contract, you must provide an  `async()`  method in the  `response`  section. The following code shows an example:

**Groovy DSL.**  

```java
org.springframework.cloud.contract.spec.Contract.make {
request {
method GET()
url '/get'
}
response {
status OK()
body 'Passed'
async()
}
}
```

**YAML.**  

```java
response:
async: true
```

## 95.8 Working with Context Paths

Spring Cloud Contract supports context paths.

|images/important.png|Important|
|----|----|
|The only change needed to fully support context paths is the switch on the  **PRODUCER**  side. Also, the autogenerated tests must use  **EXPLICIT**  mode. The consumer side remains untouched. In order for the generated test to pass, you must use  **EXPLICIT**  mode. |

**Maven.**  

```xml
<plugin>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-contract-maven-plugin</artifactId>
<version>${spring-cloud-contract.version}</version>
<extensions>true</extensions>
<configuration>
<testMode>EXPLICIT</testMode>
</configuration>
</plugin>
```

**Gradle.**  

```java
contracts {
		testMode = 'EXPLICIT'
}
```

That way, you generate a test that  **DOES NOT**  use MockMvc. It means that you generate real requests and you need to setup your generated test’s base class to work on a real socket.

Consider the following contract:

```java
org.springframework.cloud.contract.spec.Contract.make {
	request {
		method 'GET'
		url '/my-context-path/url'
	}
	response {
		status OK()
	}
}
```

The following example shows how to set up a base class and Rest Assured:

```java
import io.restassured.RestAssured;
import org.junit.Before;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest(classes = ContextPathTestingBaseClass.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ContextPathTestingBaseClass {

	@LocalServerPort int port;

	@Before
	public void setup() {
		RestAssured.baseURI = "http://localhost";
		RestAssured.port = this.port;
	}
}
```

If you do it this way:

- All of your requests in the autogenerated tests are sent to the real endpoint with your context path included (for example,  `/my-context-path/url` ).

- Your contracts reflect that you have a context path. Your generated stubs also have that information (for example, in the stubs, you have to call  `/my-context-path/url` ).

## 95.9 Working with Web Flux

Spring Cloud Contract requires the usage of  `EXPLICIT`  mode in your generated tests to work with Web Flux.

**Maven.**  

```xml
<plugin>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-contract-maven-plugin</artifactId>
<version>${spring-cloud-contract.version}</version>
<extensions>true</extensions>
<configuration>
<testMode>EXPLICIT</testMode>
</configuration>
</plugin>
```

**Gradle.**  

```java
contracts {
		testMode = 'EXPLICIT'
}
```

The following example shows how to set up a base class and Rest Assured for Web Flux:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = BeerRestBase.Config.class,
		webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT,
		properties = "server.port=0")
public abstract class BeerRestBase {

// your tests go here

// in this config class you define all controllers and mocked services
@Configuration
@EnableAutoConfiguration
static class Config {

	@Bean
	PersonCheckingService personCheckingService()  {
		return personToCheck -> personToCheck.age >= 20;
	}

	@Bean
	ProducerController producerController() {
		return new ProducerController(personCheckingService());
	}
}

}
```

## 95.10 Messaging Top-Level Elements

The DSL for messaging looks a little bit different than the one that focuses on HTTP. The following sections explain the differences:

- [Section 95.10.1, “Output Triggered by a Method”](multi_contract-dsl.html#contract-dsl-output-triggered-method)

- [Section 95.10.2, “Output Triggered by a Message”](multi_contract-dsl.html#contract-dsl-output-triggered-message)

- [Section 95.10.3, “Consumer/Producer”](multi_contract-dsl.html#contract-dsl-consumer-producer)

- [Section 95.10.4, “Common”](multi_contract-dsl.html#contract-dsl-common)

### 95.10.1 Output Triggered by a Method

The output message can be triggered by calling a method (such as a  `Scheduler`  when a was started and a message was sent), as shown in the following example:

**Groovy DSL.**  

```java
def dsl = Contract.make {
	// Human readable description
	description 'Some description'
	// Label by means of which the output message can be triggered
	label 'some_label'
	// input to the contract
	input {
		// the contract will be triggered by a method
		triggeredBy('bookReturnedTriggered()')
	}
	// output message of the contract
	outputMessage {
		// destination to which the output message will be sent
		sentTo('output')
		// the body of the output message
		body('''{ "bookName" : "foo" }''')
		// the headers of the output message
		headers {
			header('BOOK-NAME', 'foo')
		}
	}
}
```

**YAML.**  

```java
# Human readable description
description: Some description
# Label by means of which the output message can be triggered
label: some_label
input:
# the contract will be triggered by a method
triggeredBy: bookReturnedTriggered()
# output message of the contract
outputMessage:
# destination to which the output message will be sent
sentTo: output
# the body of the output message
body:
bookName: foo
# the headers of the output message
headers:
BOOK-NAME: foo
```

In the previous example case, the output message is sent to  `output`  if a method called  `bookReturnedTriggered`  is executed. On the message  **publisher’s**  side, we generate a test that calls that method to trigger the message. On the  **consumer**  side, you can use the  `some_label`  to trigger the message.

### 95.10.2 Output Triggered by a Message

The output message can be triggered by receiving a message, as shown in the following example:

**Groovy DSL.**  

```java
def dsl = Contract.make {
	description 'Some Description'
	label 'some_label'
	// input is a message
	input {
		// the message was received from this destination
		messageFrom('input')
		// has the following body
		messageBody([
		        bookName: 'foo'
		])
		// and the following headers
		messageHeaders {
			header('sample', 'header')
		}
	}
	outputMessage {
		sentTo('output')
		body([
		        bookName: 'foo'
		])
		headers {
			header('BOOK-NAME', 'foo')
		}
	}
}
```

**YAML.**  

```java
# Human readable description
description: Some description
# Label by means of which the output message can be triggered
label: some_label
# input is a message
input:
messageFrom: input
# has the following body
messageBody:
bookName: 'foo'
# and the following headers
messageHeaders:
sample: 'header'
# output message of the contract
outputMessage:
# destination to which the output message will be sent
sentTo: output
# the body of the output message
body:
bookName: foo
# the headers of the output message
headers:
BOOK-NAME: foo
```

In the preceding example, the output message is sent to  `output`  if a proper message is received on the  `input`  destination. On the message  **publisher’s**  side, the engine generates a test that sends the input message to the defined destination. On the  **consumer**  side, you can either send a message to the input destination or use a label ( `some_label`  in the example) to trigger the message.

### 95.10.3 Consumer/Producer

|images/important.png|Important|
|----|----|
|This section is valid only for Groovy DSL. |

In HTTP, you have a notion of  `client` / `stub and `server` / `test`  notation. You can also use those paradigms in messaging. In addition, Spring Cloud Contract Verifier also provides the  `consumer`  and  `producer`  methods, as presented in the following example (note that you can use either  `$`  or  `value`  methods to provide  `consumer`  and  `producer`  parts):

```java
Contract.make {
	label 'some_label'
	input {
		messageFrom value(consumer('jms:output'), producer('jms:input'))
		messageBody([
				bookName: 'foo'
		])
		messageHeaders {
			header('sample', 'header')
		}
	}
	outputMessage {
		sentTo $(consumer('jms:input'), producer('jms:output'))
		body([
				bookName: 'foo'
		])
	}
}
```

### 95.10.4 Common

In the  `input`  or  `outputMessage`  section you can call  `assertThat`  with the name of a  `method`  (e.g.  `assertThatMessageIsOnTheQueue()` ) that you have defined in the base class or in a static import. Spring Cloud Contract will execute that method in the generated test.

## 95.11 Multiple Contracts in One File

You can define multiple contracts in one file. Such a contract might resemble the following example:

**Groovy DSL.**  

```java
import org.springframework.cloud.contract.spec.Contract

[
Contract.make {
name("should post a user")
request {
method 'POST'
url('/users/1')
}
response {
status OK()
}
},
Contract.make {
request {
method 'POST'
url('/users/2')
}
response {
status OK()
}
}
]
```

**YAML.**  

```java
---
name: should post a user
request:
method: POST
url: /users/1
response:
status: 200

---
request:
method: POST
url: /users/2
response:
status: 200
```

In the preceding example, one contract has the  `name`  field and the other does not. This leads to generation of two tests that look more or less like this:

```java
package org.springframework.cloud.contract.verifier.tests.com.hello;

import com.example.TestBase;
import com.jayway.jsonpath.DocumentContext;
import com.jayway.jsonpath.JsonPath;
import com.jayway.restassured.module.mockmvc.specification.MockMvcRequestSpecification;
import com.jayway.restassured.response.ResponseOptions;
import org.junit.Test;

import static com.jayway.restassured.module.mockmvc.RestAssuredMockMvc.*;
import static com.toomuchcoding.jsonassert.JsonAssertion.assertThatJson;
import static org.assertj.core.api.Assertions.assertThat;

public class V1Test extends TestBase {

	@Test
	public void validate_should_post_a_user() throws Exception {
		// given:
			MockMvcRequestSpecification request = given();

		// when:
			ResponseOptions response = given().spec(request)
					.post("/users/1");

		// then:
			assertThat(response.statusCode()).isEqualTo(200);
	}

	@Test
	public void validate_withList_1() throws Exception {
		// given:
			MockMvcRequestSpecification request = given();

		// when:
			ResponseOptions response = given().spec(request)
					.post("/users/2");

		// then:
			assertThat(response.statusCode()).isEqualTo(200);
	}

}
```

Notice that, for the contract that has the  `name`  field, the generated test method is named  `validate_should_post_a_user` . For the one that does not have the name, it is called  `validate_withList_1` . It corresponds to the name of the file  `WithList.groovy`  and the index of the contract in the list.

The generated stubs is shown in the following example:

```java
should post a user.json
1_WithList.json
```

As you can see, the first file got the  `name`  parameter from the contract. The second got the name of the contract file ( `WithList.groovy` ) prefixed with the index (in this case, the contract had an index of  `1`  in the list of contracts in the file).

> As you can see, it is much better if you name your contracts because doing so makes your tests far more meaningful.

## 95.12 Generating Spring REST Docs snippets from the contracts

When you want to include the requests and responses of your API using Spring REST Docs, you only need to make some minor changes to your setup if you are using MockMvc and RestAssuredMockMvc. Simply include the following dependencies if you haven’t already.

**Maven.**  

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-contract-verifier</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.springframework.restdocs</groupId>
	<artifactId>spring-restdocs-mockmvc</artifactId>
	<optional>true</optional>
</dependency>
```

**Gradle.**  

```java
testCompile 'org.springframework.cloud:spring-cloud-starter-contract-verifier'
testCompile 'org.springframework.restdocs:spring-restdocs-mockmvc'
```

Next you need to make some changes to your base class like the following example.

```java
package com.example.fraud;

import io.restassured.module.mockmvc.RestAssuredMockMvc;

import org.junit.Before;
import org.junit.Rule;
import org.junit.rules.TestName;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.restdocs.JUnitRestDocumentation;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.documentationConfiguration;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public abstract class FraudBaseWithWebAppSetup {

	private static final String OUTPUT = "target/generated-snippets";

	@Rule
	public JUnitRestDocumentation restDocumentation = new JUnitRestDocumentation(OUTPUT);

	@Rule public TestName testName = new TestName();

	@Autowired
	private WebApplicationContext context;

	@Before
	public void setup() {
	RestAssuredMockMvc.mockMvc(MockMvcBuilders.webAppContextSetup(this.context)
			.apply(documentationConfiguration(this.restDocumentation))
			.alwaysDo(document(getClass().getSimpleName() + "_" + testName.getMethodName()))
			.build());
	}

	protected void assertThatRejectionReasonIsNull(Object rejectionReason) {
		assert rejectionReason == null;
	}
}
```

In case you are using the standalone setup, you can set up RestAssuredMockMvc like this:

```java
package com.example.fraud;

import io.restassured.module.mockmvc.RestAssuredMockMvc;
import org.junit.Before;
import org.junit.Rule;
import org.junit.rules.TestName;
import org.springframework.restdocs.JUnitRestDocumentation;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.documentationConfiguration;

public abstract class FraudBaseWithStandaloneSetup {

	private static final String OUTPUT = "target/generated-snippets";

	@Rule
	public JUnitRestDocumentation restDocumentation = new JUnitRestDocumentation(OUTPUT);

	@Rule public TestName testName = new TestName();

	@Before
	public void setup() {
		RestAssuredMockMvc.standaloneSetup(MockMvcBuilders.standaloneSetup(new FraudDetectionController())
				.apply(documentationConfiguration(this.restDocumentation))
				.alwaysDo(document(getClass().getSimpleName() + "_" + testName.getMethodName())));
	}

}
```

> You don’t need to specify the output directory for the generated snippets since version 1.2.0.RELEASE of Spring REST Docs.

