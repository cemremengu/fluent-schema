# fluent-schema

A fluent API to generate JSON schemas (draft-07) for Node.js and browser.

## Install

    npm install fluent-schema --save

or

    yarn add fluent-schema

## Usage

```javascript
const { FluentSchema, FORMATS } = require('fluent-schema')
const schema = FluentSchema()
  .id('http://foo/user')
  .title('My First Fluent JSON Schema')
  .description('A simple user')
  .prop('email')
  .asString()
  .format(FORMATS.EMAIL)
  .required()
  .prop('password')
  .asString()
  .minLength(8)
  .required()
  .prop('role')
  .enum(['ADMIN', 'USER'])
  .default('USER')
  .definition(
    'address',
    FluentSchema()
      .prop('line1')
      .required()
      .prop('line2')
      .prop('country')
      .required()
      .prop('city')
      .required()
      .prop('zipcoce')
      .required()
  )
  .prop('address')
  .ref('#definitions/address')

console.log(JSON.stringify(schema.valueOf(), undefined, 2))
```

Schema generated:

```json
{
  "definitions": {
    "address": {
      "type": "object",
      "$id": "#definitions/address",
      "properties": {
        "line1": {
          "type": "string",
          "$id": "#definitions/address/properties/line1"
        },
        "line2": {
          "type": "string",
          "$id": "#definitions/address/properties/line2"
        },
        "country": {
          "type": "string",
          "$id": "#definitions/address/properties/country"
        },
        "city": {
          "type": "string",
          "$id": "#definitions/address/properties/city"
        },
        "zipcoce": {
          "type": "string",
          "$id": "#definitions/address/properties/zipcode"
        }
      },
      "required": ["line1", "country", "city", "zipcode"]
    }
  },
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["email", "password"],
  "$id": "http://foo/user",
  "title": "My First Fluent JSON Schema",
  "description": "A simple user",
  "properties": {
    "email": {
      "type": "string",
      "$id": "#properties/email",
      "format": "email"
    },
    "password": {
      "type": "string",
      "$id": "#properties/password",
      "minLength": 8
    },
    "role": {
      "type": "string",
      "$id": "#properties/role",
      "enum": ["ADMIN", "USER"],
      "default": "USER"
    },
    "address": {
      "$ref": "#definitions/address"
    }
  }
}
```

## Integration

Fluent schema **doesn't** validate a JSON schema. However there are many libraries that can do that for you.
Below a few examples using [AJV](https://ajv.js.org/)

    npm install ajv --save

or

    yarn add ajv

### Validate an empty model

Snippet

```javascript
const ajv = new Ajv({ allErrors: true })
const validate = ajv.compile(userSchema.valueOf())
let user = {}
let valid = validate(user)
console.log({ valid }) //=> {valid: false}
console.log(validate.errors) //=> {valid: false}
```

Output:

```
{valid: false}
errors: [
  {
    keyword: 'required',
    dataPath: '',
    schemaPath: '#/required',
    params: { missingProperty: 'email' },
    message: "should have required property 'email'",
  },
  {
    keyword: 'required',
    dataPath: '',
    schemaPath: '#/required',
    params: { missingProperty: 'password' },
    message: "should have required property 'password'",
  },
]

```

### Validate a partially filled model

Snippet

```javascript
user = { email: 'test', password: 'password' }
valid = validate(user)
console.log({ valid })
console.log(validate.errors)
```

Output:

```
{valid: false}
errors:
[ { keyword: 'format',
    dataPath: '.email',
    schemaPath: '#/properties/email/format',
    params: { format: 'email' },
    message: 'should match format "email"' } ]

```

### Validate a model with a wrong format attribute

Snippet

```javascript
user = { email: 'test@foo.com', password: 'password' }
valid = validate(user)
console.log({ valid })
console.log('errors:', validate.errors)
```

Output:

```
{valid: false}
errors: [ { keyword: 'required',
    dataPath: '.address',
    schemaPath: '#definitions/address/required',
    params: { missingProperty: 'country' },
    message: 'should have required property \'country\'' },
  { keyword: 'required',
    dataPath: '.address',
    schemaPath: '#definitions/address/required',
    params: { missingProperty: 'city' },
    message: 'should have required property \'city\'' },
  { keyword: 'required',
    dataPath: '.address',
    schemaPath: '#definitions/address/required',
    params: { missingProperty: 'zipcoce' },
    message: 'should have required property \'zipcode\'' } ]
```

### Valid model

Snippet

```javascript
user = { email: 'test@foo.com', password: 'password' }
valid = validate(user)
console.log({ valid })
```

Output:

     {valid: true}

## Validation Keywords Supported

[Reference](https://json-schema.org/latest/json-schema-validation.html):

1. Validation Keywords for Any Instance Type

- [x] types
  - [x] string
  - [x] boolean
  - [x] number
  - [x] integer
  - [x] array
  - [x] object
  - [x] null
- [x] enum
- [x] const

2. Validation Keywords for Numeric Instances (number and integer)

- [x] multipleOf
- [x] maximum
- [x] exclusiveMaximum
- [x] minimum
- [x] exclusiveMinimum

3. Validation Keywords for Strings

- [x] maxLength
- [x] minLength
- [x] pattern
- [x] format

4. Validation Keywords for Arrays

- [x] items
- [x] additionalItems
- [x] maxItems
- [x] minItems
- [x] uniqueItems
- [x] contains

5. Validation Keywords for Objects

- [x] maxProperties
- [x] minProperties
- [x] required
- [x] properties
- [x] patternProperties
- [x] additionalProperties
- [x] dependencies
- [x] propertyNames

6. Keywords for Applying Subschemas Conditionally

- [x] if
- [x] then
- [x] else

7. Keywords for Applying Subschemas With Boolean Logic

- [x] allOf
- [x] anyOf
- [x] oneOf
- [x] not

## Acknowledgments

Thank [Matteo Collina](https://twitter.com/matteocollina) for pushing me to implement this utility! 🙏

## related projects

- JSON Schema [Draft 7](http://json-schema.org/specification-links.html#draft-7)
- [Understanding JSON Schema](https://json-schema.org/understanding-json-schema/) (despite is referring to draft 6 the guide still good to grasp the main concepts)
- [AJV]() JSON Schema validator
- [jsonschema.net](https://www.jsonschema.net/) an online JSON Schema visual editor (it doesn't support advance features)

## Licence

Licensed under [MIT](./LICENSE).
