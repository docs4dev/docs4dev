## 95.ContractDSL

Spring Cloud Contract支持开箱即用的2种DSL.一个写在 `Groovy` ，一个写在 `YAML` .

如果您决定在Groovy中编写Contract，如果您之前没有使用过Groovy，请不要惊慌.由于Contract DSL仅使用它的一小部分（仅文字，方法调用和闭包），因此并不真正需要语言知识.此外，DSL是静态类型的，以使其在不知道DSL本身的情况下使程序员可读.

|图片/ important.png |重要|
| ---- | ---- |
|请记住，在GroovyContract文件中，您必须为 `Contract` 类和 `make` 静态导入提供完全限定名称，例如 `org.springframework.cloud.spec.Contract.make { … }` .您还可以为 `Contract` 类提供导入： `import org.springframework.cloud.spec.Contract` 然后调用 `Contract.make { … }` . |

> Spring Cloud Contract支持在单个文件中定义多个Contract.

以下是Groovy合约定义的完整示例：

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

以下是YAML合约定义的完整示例：

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

> 您可以使用独立maven命令将Contract编译为存根映射： `mvn org.springframework.cloud:spring-cloud-contract-maven-plugin:convert` 

## 95.1限制

> Spring Cloud Contract Verifier不能正确支持XML.请使用JSON或帮助我们实现此功能.

> 对验证JSON数组大小的支持是实验性的.如果要将其打开，请将以下系统属性的值设置为 `true` ： `spring.cloud.contract.verifier.assert.size` .默认情况下，此功能设置为 `false` .您还可以在插件配置中提供 `assertJsonSize` 属性.

> B因为JSON结构可以有任何形式，所以在 `GString` 中使用Groovy DSL和 `value(consumer(…), producer(…))` 表示法时，无法正确解析它.这就是为什么你应该使用Groovy Map表示法.

## 95.2常见的顶级元素

以下部分描述了最常见的顶级内容：

- [Section 95.2.1, “Description”](multi_contract-dsl.html#contract-dsl-description)

- [Section 95.2.2, “Name”](multi_contract-dsl.html#contract-dsl-name)

- [Section 95.2.3, “Ignoring Contracts”](multi_contract-dsl.html#contract-dsl-ignoring-contracts)

- [Section 95.2.4, “Passing Values from Files”](multi_contract-dsl.html#contract-dsl-passing-values-from-files)

- [Section 95.2.5, “HTTP Top-Level Elements”](multi_contract-dsl.html#contract-dsl-http-top-level-elements)

### 95.2.1说明

您可以在Contract中添加 `description` .描述是任意文本.以下代码显示了一个示例：

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

### 95.2.2姓名

您可以为Contract提供名称.假设您提供了以下名称： `should register a user` .如果这样做，则自动生成的测试的名称为 `validate_should_register_a_user` .此外，WireMock存根中存根的名称是 `should_register_a_user.json` .

|图片/ important.png |重要|
| ---- | ---- |
|您必须确保该名称不包含任何使生成的测试无法编译的字符.另外，请记住，如果为多个Contract提供相同的名称，则自动生成的测试将无法编译，并且生成的存根将相互覆盖. |

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

### 95.2.3忽略Contract

如果要忽略Contract，可以在插件配置中设置忽略Contract的值，也可以在Contract本身上设置 `ignored` 属性：

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

### 95.2.4从文件传递值

从版本 `1.2.0` 开始，您可以传递文件中的值.假设您的项目中包含以下资源.

```java
└── src
└── test
└── resources
└── contracts
├── readFromFile.groovy
├── request.json
└── response.json
```

进一步假设您的Contract如下：

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

进一步假设JSON文件如下：

**request.json** 

```java
{ "status" : "REQUEST" }
```

**response.json** 

```java
{ "status" : "RESPONSE" }
```

当进行测试或存根生成时，文件的内容将传递给请求或响应的主体.文件名必须是一个文件，其位置相对于Contract所在的文件夹.

### 95.2.5 HTTP顶级元素

可以在Contract定义的顶级闭包中调用以下方法.  `request` 和 `response` 是强制性的.  `priority` 是可选的.

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

|图片/ important.png |重要|
| ---- | ---- |
|如果要使合约具有 **higher** 优先级值，则需要将 **lower** 数字传递给 `priority` 标记/方法.例如.  `priority` ，其值为 `5` ， **higher** 优先级高于 `priority` ，值为 `10` . |

## 95.3请求

HTTP协议仅需要在请求中指定 **method and url** .Contract的请求定义中必须使用相同的信息.

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

可以指定绝对而不是相对 `url` ，但使用 `urlPath` 是推荐的方法，因为这样做会使测试 **host-independent** .

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

`request` 可能包含 **query parameters** .

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

`request` 可能包含其他 **request headers** ，如以下示例所示：

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

`request` 可能包含其他 **request cookies** ，如以下示例所示：

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

`request` 可能包含 **request body** ：

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

`request` 可能包含 **multipart** 元素.要包含多部分元素，请使用 `multipart` 方法/部分，如以下示例所示

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

在前面的示例中，我们以两种方式之一定义参数：

**Groovy DSL** 

- Directly，通过使用Map表示法，其中值可以是动态属性（例如 `formParameter: $(consumer(…), producer(…))` ）.

- By使用允许您设置命名参数的 `named(…)` 方法.命名参数可以设置 `name` 和 `content` .您可以通过具有两个参数的方法（例如 `named("fileName", "fileContent")` ）或通过Map表示法（例如 `named(name: "fileName", content: "fileContent")` ）来调用它.

**YAML** 

- 多部分参数通过 `multipart.params` 部分设置

- 可以通过 `multipart.named` 部分设置命名参数（给定参数名称的 `fileName` 和 `fileContent` ）.该部分包含 `paramName` （参数名称）， `fileName` （文件名）， `fileContent` （文件内容）字段

- 动态位可以通过 `matchers.multipart` 部分设置

- for参数使用可以接受 `regex` 或 `predefined` 正则表达式的 `params` 部分

- for named params使用 `named` 部分，首先通过 `paramName` 定义参数名称然后您可以通过 `regex` 或 `predefined` 正则表达式传递 `fileName` 或 `fileContent` 的参数化

根据该Contract，生成的测试如下：

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

WireMock存根如下：

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

## 95.4回复

响应必须包含 **HTTP status code** ，并且可能包含其他信息.以下代码显示了一个示例：

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

除状态外，响应可能包含 **headers** ， **cookies** 和 **body** ，两者的指定方式与请求中的相同（请参阅上一段）.

> Via Groovy DSL您可以引用 `org.springframework.cloud.contract.spec.internal.HttpStatus` 方法来提供有意义的状态而不是数字.例如.您可以将 `OK()` 调用 `200` 或 `400` 作为 `400` .

## 95.5动态属性

Contract可以包含一些动态属性：时间戳，ID等.您不希望强制消费者将其时钟存根以始终返回相同的时间值以使其获得由存根匹配.

对于Groovy DSL，您可以通过两种方式在Contract中提供动态部分：直接在正文中传递它们或将它们设置在一个名为 `bodyMatchers` 的单独部分中.

> Be之前2.0.0这些是使用 `testMatchers` 和 `stubMatchers` 设置的，请查看[migration guide](https://github.com/spring-cloud/spring-cloud-contract/wiki/Spring-Cloud-Contract-2.0-Migration-Guide)以获取更多信息.

对于YAML，您只能使用 `matchers` 部分.

### 95.5.1身体内的动态属性

|图片/ important.png |重要|
| ---- | ---- |
|此部分仅适用于Groovy DSL.查看[Section 95.5.7, “Dynamic Properties in the Matchers Sections”](multi_contract-dsl.html#contract-matchers)部分，了解类似功能的YAML示例. |

您可以使用 `value` 方法在主体内设置属性，或者，如果使用GroovyMap表示法，则使用 `$()` .以下示例显示如何使用value方法设置动态属性：

```java
value(consumer(...), producer(...))
value(c(...), p(...))
value(stub(...), test(...))
value(client(...), server(...))
```

以下示例显示如何使用 `$()` 设置动态属性：

```java
$(consumer(...), producer(...))
$(c(...), p(...))
$(stub(...), test(...))
$(client(...), server(...))
```

两种方法都同样有效.  `stub` 和 `client` 方法是 `consumer` 方法的别名.后续部分将详细介绍您可以对这些值执行的操作.

### 95.5.2正则表达式

|图片/ important.png |重要|
| ---- | ---- |
|此部分仅适用于Groovy DSL.查看[Section 95.5.7, “Dynamic Properties in the Matchers Sections”](multi_contract-dsl.html#contract-matchers)部分，了解类似功能的YAML示例. |

您可以使用正则表达式在Contract DSL中编写请求.当您想要指示应为遵循给定模式的请求提供给定响应时，这样做特别有用.此外，当您需要为测试和服务器端测试使用模式而不是精确值时，可以使用正则表达式.

以下示例显示如何使用正则表达式编写请求：

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

您还可以仅使用正则表达式提供通信的一侧.如果这样做，则Contract引擎会自动提供与提供的正则表达式匹配的生成字符串.以下代码显示了一个示例：

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

在前面的示例中，通信的相对侧具有为请求和响应生成的相应数据.

Spring Cloud Contract附带了一系列可在Contract中使用的预定义正则表达式，如以下示例所示：

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

在您的Contract中，您可以使用它，如以下示例所示：

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

### 95.5.3传递可选参数

|图片/ important.png |重要|
| ---- | ---- |
|此部分仅适用于Groovy DSL.查看[Section 95.5.7, “Dynamic Properties in the Matchers Sections”](multi_contract-dsl.html#contract-matchers)部分，了解类似功能的YAML示例. |

可以在Contract中提供可选参数.但是，您只能为以下内容提供可选参数：

- STUB请求方

- TEST响应方面

以下示例显示了如何提供可选参数：

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

通过使用 `optional()` 方法包装正文的一部分，可以创建必须存在0次或更多次的正则表达式.

如果您使用Spock，将从前一个示例生成以下测试：

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

还将生成以下存根：

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

### 95.5.4在服务器端执行自定义方法

|图片/ important.png |重要|
| ---- | ---- |
|此部分仅适用于Groovy DSL.查看[Section 95.5.7, “Dynamic Properties in the Matchers Sections”](multi_contract-dsl.html#contract-matchers)部分，了解类似功能的YAML示例. |

您可以定义在测试期间在服务器端执行的方法调用.这种方法可以添加到配置中定义为“baseClassForTests”的类中.以下代码显示了测试用例的Contract部分的示例：

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

以下代码显示了测试用例的基类部分：

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

|图片/ important.png |重要|
| ---- | ---- |
|您不能同时使用String和 `execute` 来执行连接.例如，调用 `header('Authorization', 'Bearer ' + execute('authToken()'))` 会导致不正确的结果.相反，调用 `header('Authorization', execute('authToken()'))` 并确保 `authToken()` 方法返回您需要的所有内容. |

从JSON读取的对象的类型可以是以下之一，具体取决于JSON路径：

-  `String` ：如果您指向JSON中的 `String` 值.

-  `JSONArray` ：如果您指向JSON中的 `List` .

-  `Map` ：如果你指向JSON中的 `Map` .

-  `Number` ：如果您在JSON中指向 `Integer` ， `Double` 等.

-  `Boolean` ：如果你指向JSON中的 `Boolean` .

在Contract的请求部分中，您可以指定 `body` 应从方法中获取.

|图片/ important.png |重要|
| ---- | ---- |
|您必须同时提供消费者和生产环境者方面.  `execute` 部分适用于整个身体 - 不适用于部分身体. |

以下示例显示如何从JSON读取对象：

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

前面的示例导致在请求正文中调用 `hashCode()` 方法.它应该类似于以下代码：

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

### 95.5.5引用响应请求

最好的情况是提供固定值，但有时您需要在响应中引用请求.

如果你使用Groovy DSL编写Contract，你可以使用 `fromRequest()` 方法，它允许你引用一堆元素HTTP请求.您可以使用以下选项：

-  `fromRequest().url()` ：返回请求URL和查询参数.

-  `fromRequest().query(String key)` ：返回具有给定名称的第一个查询参数.

-  `fromRequest().query(String key, int index)` ：返回具有给定名称的第n个查询参数.

-  `fromRequest().path()` ：返回完整路径.

-  `fromRequest().path(int index)` ：返回第n个路径元素.

-  `fromRequest().header(String key)` ：返回具有给定名称的第一个标头.

-  `fromRequest().header(String key, int index)` ：返回具有给定名称的第n个标头.

-  `fromRequest().body()` ：返回完整的请求正文.

-  `fromRequest().body(String jsonPath)` ：从请求中返回与JSON路径匹配的元素.

如果您正在使用YAML合约定义，则必须使用带有自定义Spring Cloud Contract函数的[Handlebars](http://handlebarsjs.com/)  `{{{ }}}` 表示法来实现此目的.

-  `{{{ request.url }}}` ：返回请求URL和查询参数.

-  `{{{ request.query.key.[index] }}}` ：返回具有给定名称的第n个查询参数.例如.对于键 `foo` ，首次输入 `{{{ request.query.foo.[0] }}}` 

-  `{{{ request.path }}}` ：返回完整路径.

-  `{{{ request.path.[index] }}}` ：返回第n个路径元素.例如.第一次入境 ```  {{{request.path.[0] }}}

-  `{{{ request.headers.key }}}` ：返回具有给定名称的第一个标头.

-  `{{{ request.headers.key.[index] }}}` ：返回具有给定名称的第n个标头.

-  `{{{ request.body }}}` ：返回完整的请求正文.

-  `{{{ jsonpath this 'your.json.path' }}}` ：从请求中返回与JSON路径匹配的元素.例如.为json路径 `$.foo`   -   `{{{ jsonpath this '$.foo' }}}` 

考虑以下Contract：

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

运行JUnit测试生成会导致测试类似于以下示例：

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

如您所见，请求中的元素已在响应中正确引用.

生成的WireMock存根应类似于以下示例：

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

发送诸如Contract `request` 部分中提出的请求之后的请求会导致发送以下响应正文：

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

|图片/ important.png |重要|
| ---- | ---- |
|此功能仅适用于版本大于或等于2.5.1的WireMock. Spring Cloud Contract Verifier使用WireMock的 `response-template` 响应转换器.它使用Handlebars将Mustache  `{{{ }}}` 模板转换为适当的值.另外，它注册了两个辅助函数：|

-  `escapejsonbody` ：以可嵌入JSON格式的格式转义请求正文.

-  `jsonpath` ：对于给定参数，在请求正文中查找对象.

### 95.5.6注册您自己的WireMock扩展

WireMock允许您注册自定义扩展.默认情况下，Spring Cloud Contract会注册转换器，该转换器允许您引用响应中的请求.如果要提供自己的扩展，可以注册 `org.springframework.cloud.contract.verifier.dsl.wiremock.WireMockExtensions` 接口的实现.由于我们使用spring.factories扩展方法，您可以在 `META-INF/spring.factories` 文件中创建类似于以下内容的条目：

```java
org.springframework.cloud.contract.verifier.dsl.wiremock.WireMockExtensions=\
org.springframework.cloud.contract.stubrunner.provider.wiremock.TestWireMockExtensions
org.springframework.cloud.contract.spec.ContractConverter=\
org.springframework.cloud.contract.stubrunner.TestCustomYamlContractConverter
```

以下是自定义扩展的示例：

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

|图片/ important.png |重要|
| ---- | ---- |
|如果希望仅将转换应用于明确需要它的映射，请记住覆盖 `applyGlobally()` 方法并将其设置为 `false` . |

### 95.5.7匹配器部分中的动态属性

如果您使用[Pact](https://docs.pact.io/)，以下讨论可能看起来很熟悉.相当多的用户习惯于在身体之间进行分离并设置Contract的动态部分.

您可以使用 `bodyMatchers` 部分，原因有两个：

- 定义应该在存根中结束的动态值.您可以在Contract的 `request` 或 `inputMessage` 部分进行设置.

- 验证测试结果.本节存在于Contract的 `response` 或 `outputMessage` 方面.

目前，Spring Cloud Contract Verifier仅支持基于JSON路径的匹配器，具有以下匹配可能性：

**Groovy DSL** 

- 对于存根（在消费者方面的测试中）：

-  `byEquality()` ：通过提供的JSON路径从消费者请求中获取的值必须等于Contract中提供的值.

-  `byRegex(…)` ：通过提供的JSON路径从消费者请求中获取的值必须与正则表达式匹配.

-  `byDate()` ：通过提供的JSON路径从消费者请求获取的值必须与ISO Date值的正则表达式匹配.

-  `byTimestamp()` ：通过提供的JSON路径从消费者请求获取的值必须与ISO DateTime值的正则表达式匹配.

-  `byTime()` ：通过提供的JSON路径从消费者请求中获取的值必须与ISO时间值的正则表达式匹配.

- 用于验证（在生产环境者方面生成的测试中）：

-  `byEquality()` ：通过提供的JSON路径从生产环境者的响应中获取的值必须等于Contract中提供的值.

-  `byRegex(…)` ：通过提供的JSON路径从生产环境者的响应中获取的值必须与正则表达式匹配.

-  `byDate()` ：通过提供的JSON路径从生产环境者的响应中获取的值必须与ISO Date值的正则表达式匹配.

-  `byTimestamp()` ：通过提供的JSON路径从生产环境者的响应中获取的值必须与ISO DateTime值的正则表达式匹配.

-  `byTime()` ：取自的Value生产环境者通过提供的JSON路径的响应必须与ISO时间值的正则表达式匹配.

-  `byType()` ：通过提供的JSON路径从生产环境者的响应中获取的值需要与Contract中响应正文中定义的类型相同.  `byType` 可以关闭，您可以在其中设置 `minOccurrence` 和 `maxOccurrence` .这样，您可以断言展平集合的大小.要检查unflattened集合的大小，请使用带有 `byCommand(…)`  testMatcher的自定义方法.

-  `byCommand(…)` ：通过提供的JSON路径从生产环境者的响应中获取的值作为输入传递给您提供的自定义方法.例如， `byCommand('foo($it)')` 导致调用与JSON路径匹配的值传递到的 `foo` 方法.从JSON读取的对象的类型可以是以下之一，具体取决于JSON路径：

-  `String` ：如果指向 `String` 值.

-  `JSONArray` ：如果你指向 `List` .

-  `Map` ：如果你指向 `Map` .

-  `Number` ：如果您指向 `Integer` ， `Double` 或其他类型的数字.

-  `Boolean` ：如果你指向一个 `Boolean` .

-  `byNull()` ：通过提供的JSON路径从响应中获取的值必须为null

_15588请阅读Groovy部分，详细说明类型的含义

对于YAML，匹配器的结构看起来像这样

```java
- path: $.foo
type: by_regex
value: bar
```

或者，如果要使用其中一个预定义的正则表达式 `[only_alpha_unicode, number, any_boolean, ip_address, hostname, email, url, uuid, iso_date, iso_date_time, iso_time, iso_8601_with_offset, non_empty, non_blank]` ：

```java
- path: $.foo
type: by_regex
predefined: only_alpha_unicode
```

您可以在下面找到`type`s的允许列表.

- For  `stubMatchers` ：

-  `by_equality` 

-  `by_regex` 

-  `by_date` 

-  `by_timestamp` 

-  `by_time` 

- For  `testMatchers` ：

-  `by_equality` 

-  `by_regex` 

-  `by_date` 

-  `by_timestamp` 

-  `by_time` 

-  `by_type` 

- 还接受了2个附加字段： `minOccurrence` 和 `maxOccurrence` .

-  `by_command` 

-  `by_null` 

请考虑以下示例：

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

在前面的示例中，您可以在 `matchers` 部分中查看Contract的动态部分.对于请求部分，您可以看到，对于除 `valueWithoutAMatcher` 之外的所有字段，将显式设置存根应包含的正则表达式的值.对于 `valueWithoutAMatcher` ，验证的方式与不使用匹配器的方式相同.在这种情况下，测试执行相等性检查.

对于 `bodyMatchers` 部分中的响应方，我们以类似的方式定义动态部分.唯一的区别是 `byType` 匹配器也存在.验证程序引擎检查四个字段以验证测试的响应是否具有JSON路径与给定字段匹配的值，与响应正文中定义的类型相同，并通过以下检查（基于被调用的方法）：

- For  `$.valueWithTypeMatch` ，引擎检查类型是否相同.

- For  `$.valueWithMin` ，引擎检查类型并断言大小是否大于或等于最小值.

- For  `$.valueWithMax` ，引擎检查类型并断言大小是否小于或等于最大值.

- For  `$.valueWithMinMax` ，引擎检查类型并断言大小是否在最小和最大出现之间.

生成的测试类似于以下示例（请注意， `and` 部分将自动生成的断言与断言与匹配器分开）：

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

|图片/ important.png |重要|
| ---- | ---- |
|请注意，对于 `byCommand` 方法，该示例调用 `assertThatValueIsANumber` .必须在测试基类中定义此方法，或者将该方法静态导入到测试中.请注意， `byCommand` 调用已转换为 `assertThatValueIsANumber(parsedJson.read("$.duck"));` .这意味着引擎获取方法名称并将适当的JSON路径作为参数传递给它. |

生成的WireMock存根在以下示例中：

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

|图片/ important.png |重要|
| ---- | ---- |
|如果使用 `matcher` ，那么 `matcher` 与JSON路径一起寻址的请求和响应部分将从断言中删除.在验证集合的情况下，必须为 **all** 集合的元素创建匹配器. |

请考虑以下示例：

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

上面的代码导致创建以下测试（代码块仅显示断言部分）：

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

正如您所看到的，断言是错误的.只有数组的第一个元素被断言.为了解决这个问题，您应该将断言应用于整个 `$.events` 集合并使用 `byCommand(…)` 方法对其进行断言.

## 95.6 JAX-RS支持

Spring Cloud Contract Verifier支持JAX-RS 2 Client API.基类需要定义 `protected WebTarget webTarget` 和服务器初始化.测试JAX-RS API的唯一选择是启动Web服务器.此外，具有正文的请求需要设置内容类型.否则，将使用 `application/octet-stream` 的默认值.

要使用JAX-RS模式，请使用以下设置：

```java
testMode == 'JAXRSCLIENT'
```

以下示例显示了生成的测试API：

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

## 95.7异步支持

如果您在服务器端使用异步通信（您的控制器正在返回 `Callable` ， `DeferredResult` 等等）然后，在Contract中，您必须在 `response` 部分提供 `async()` 方法.以下代码显示了一个示例：

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

## 95.8使用上下文路径

Spring Cloud Contract支持上下文路径.

|图片/ important.png |重要|
| ---- | ---- |
|完全支持上下文路径所需的唯一更改是 **PRODUCER** 侧的切换.此外，自动生成的测试必须使用 **EXPLICIT** 模式.消费者方面保持不变.要生成的测试通过，必须使用 **EXPLICIT** 模式. |

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

这样，您生成了一个 **DOES NOT** 使用MockMvc的测试.这意味着您生成实际请求，并且您需要设置生成的测试的基类以在真正的套接字上工作.

考虑以下Contract：

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

以下示例显示如何设置基类和Rest Assured：

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

如果你这样做：

- 自动生成测试中的所有请求都将发送到包含上下文路径的真实endpoints（例如， `/my-context-path/url` ）.

- 您的Contract反映您有上下文路径.生成的存根也具有该信息（例如，在存根中，您必须调用 `/my-context-path/url` ）.

## 95.9使用Web Flux

Spring Cloud Contract要求在生成的测试中使用 `EXPLICIT` 模式以使用Web Flux.

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

以下示例显示如何为Web Flux设置基类和Rest Assured：

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

## 95.10消息传递顶级元素

用于消息传递的DSL看起来与关注HTTP的有点不同.以下部分解释了这些差异：

- [Section 95.10.1, “Output Triggered by a Method”](multi_contract-dsl.html#contract-dsl-output-triggered-method)

- [Section 95.10.2, “Output Triggered by a Message”](multi_contract-dsl.html#contract-dsl-output-triggered-message)

- [Section 95.10.3, “Consumer/Producer”](multi_contract-dsl.html#contract-dsl-consumer-producer)

- [Section 95.10.4, “Common”](multi_contract-dsl.html#contract-dsl-common)

### 95.10.1由方法触发的输出

可以通过调用方法（例如启动时发送 `Scheduler` 并发送消息）来触发输出消息，如以下示例所示：

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

在前面的示例中，如果执行了名为 `bookReturnedTriggered` 的方法，则输出消息将发送到 `output` .在消息 **publisher’s** 侧，我们生成一个测试，调用该方法来触发消息.在 **consumer** 侧，您可以使用 `some_label` 来触发消息.

### 95.10.2由消息触发的输出

可以通过接收消息来触发输出消息，如以下示例所示：

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

在前面的示例中，如果在 `input` 目标上收到正确的消息，则输出消息将发送到 `output` .在消息 **publisher’s** 侧，引擎生成一个测试，将输入消息发送到定义的目标.在 **consumer** 侧，您可以向输入目标发送消息或使用标签（示例中为 `some_label` ）来触发消息.

### 95.10.3消费者/制片人

|图片/ important.png |重要|
| ---- | ---- |
|此部分仅适用于Groovy DSL. |

在HTTP中，您有一个 `client`  /  `stub and `server`  /  `test` 表示法的概念.您还可以在消息传递中使用这些范例.此外，Spring Cloud Contract Verifier还提供 `consumer` 和 `producer` 方法，如以下示例所示（请注意，您可以使用 `$` 或 `value` 方法提供 `consumer` 和 `producer` 部分）：

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

### 95.10.4常见

在 `input` 或 `outputMessage` 部分中，您可以使用您在基类或静态导入中定义的 `method` （例如 `assertThatMessageIsOnTheQueue()` ）的名称调用 `assertThat` . Spring Cloud Contract将在生成的测试中执行该方法.

## 95.11一个文件中的多个Contract

您可以在一个文件中定义多个Contract.这样的Contract可能类似于以下示例：

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

在前面的示例中，一个Contract具有 `name` 字段而另一个不具有.15718_字段.这导致生成两个看起来或多或少像这样的测试：

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

请注意，对于具有 `name` 字段的Contract，生成的测试方法名为 `validate_should_post_a_user` .对于没有名称的那个，它被称为 `validate_withList_1` .它对应于文件 `WithList.groovy` 的名称和列表中Contract的索引.

生成的存根显示在以下示例中：

```java
should post a user.json
1_WithList.json
```

如您所见，第一个文件从合约中获得了 `name` 参数.第二个得到了以索引为前缀的Contract文件（ `WithList.groovy` ）的名称（在这种情况下，Contract在文件中的Contract列表中的索引为 `1` ）.

> 正如您所看到的，如果您为Contract命名会更好，因为这样做会使您的测试更有意义.

## 95.12从Contract中生成Spring REST Docs片段

当您想要使用Spring REST Docs包含API的请求和响应时，如果您使用的是MockMvc和RestAssuredMockMvc，则只需对设置进行一些小的更改.如果您还没有，请简单地包含以下依赖项.

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

接下来，您需要对基类进行一些更改，如下所示例.

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

如果您使用的是独立设置，您可以像这样设置RestAssuredMockMvc：

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

> 自Spring REST Docs版本1.2.0.RELEASE以来，您无需为生成的代码段指定输出目录.

