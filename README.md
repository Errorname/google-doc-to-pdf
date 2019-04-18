![logo](https://raw.githubusercontent.com/Errorname/google-docs-mustaches/master/logo.png)

# google-docs-mustaches

📝Interpolate Google Docs files using mustaches and formatters

[![](https://img.shields.io/npm/v/google-docs-mustaches.svg)](https://www.npmjs.com/package/google-docs-mustaches)
[![](https://img.shields.io/github/license/Errorname/google-docs-mustaches.svg)](https://github.com/Errorname/google-docs-mustaches/blob/master/LICENSE)

## How does this work?

**google-docs-mustaches** will execute requests to the [Google Drive](https://developers.google.com/drive/api/v3/about-sdk) and [Google Docs](https://developers.google.com/docs/api/how-tos/overview) APIs to copy the file and interpolate its placeholders using the given data.

## Installation

```sh
npm install google-docs-mustaches
```

## Basic usage

Create a new Google Doc file and write the following text:

```
Hello {{ firstname }} {{ lastname | uppercase }}!

You have {{ accounts[0].money }}€ in you account...
```

Then execute the following code

```js
import Mustaches from 'google-docs-mustaches'

const mustaches = new Mustaches({
  token: () => gapi.auth.getToken().access_token
})

// ID of the template
const source = '11rGORd6FRxOGERe7fh6LNQfyB48ZvOgQNH6GScK_FfA'

// ID of the destination folder
const destination = '18mcqwbaXS8NOqZjztB3OUQAc5_P8M6-l'

mustaches.interpolate({
  source,
  destination,
  data: {
    firstname: 'Thibaud',
    lastname: 'Courtoison',
    accounts: [{ money: 1500 }]
  }
})
```

## Documentation

### `new Mustaches(options: ConstructorOptions)`

```ts
type AccessToken = string

interface ConstructorOptions {
  token: () => AccessToken
}
```

- `token` will be called at every request to the Google apis.

> **AccessToken** must have the following scopes:
>
> - https://www.googleapis.com/auth/drive
> - https://www.googleapis.com/auth/documents

See also: [How to retrieve the Google token?](#how-to-retrieve-the-google-token)

### `mustaches.interpolate(options: InterpolationOptions): ID`

This method will interpolate from the `source` file and put the generated file into the `destination` folder.

```ts
type ID = string

export interface InterpolationOptions {
  source: ID
  destination?: ID
  name?: string
  data: Object
  formatters?: Formatters
  export?: MimeType
}

interface Formatters {
  [name: string]: Formatter
}

type Formatter = (value: any, ...params: any[]) => string

export enum MimeType {
  pdf = 'application/pdf',
  text = 'plain/text'
}
```

- `source` is the ID of the file which will be interpolated.
- `destination` is the ID of the destination folder where the new file will be put. If no destination is given, the new file will be put next to the `source` file.
- `name` is the name of the newly created and interpolated google doc file.
- `data` is the data given for the [interpolation](#interpolation)
- `formatters` will be used for [interpolation](#interpolation)
- `export` can be specified to export the file after the [interpolation](#interpolation)

### `mustaches.discovery(options: DiscoveryOptions): Placeholder[]`

This methods returns all the placeholders found in the `source` file. The placeholders will be interpolated to see what would have been the results if `interpolate` was called. This method will **not** mutate your source file, nor create a new one.

```ts
export interface DiscoveryOptions {
  source: ID
  data?: Object
  formatters?: Formatters
}
```

- `source` is the ID of the file which will be interpolated.
- `data` is the data given for the [interpolation](#interpolation)
- `formatters` will be used for [interpolation](#interpolation)

## Interpolation

### Mustaches

The **double brackets** notation (also known as **mustaches**) is used to define placeholders:

```
My name is {{ firstname }}. Nice to meet you!
```

During the interpolation, the placeholder will be replaced with the content of the `options.data` object.

```js
{
  firstname: 'Thibaud'
}
```

```
My name is Thibaud. Nice to meet you!
```

### Path notation

You can use **nested objects and arrays** for the interpolation:

```
{{ pokemons[1].name }}, I choose you!
```

With the following `options.data`

```js
{
  pokemons: [
    {
      name: 'Eevee',
      level: 12
    },
    {
      name: 'Pikachu',
      level: 25
    }
  ]
}
```

Will become:

```
Pikachu, I choose you!
```

_Warning: If you use an undefined variable as input, it will be resolved as an empty string._

### Formatters

You can use **formatters** to print your data and **more complex objects** any way you want.
In addition of the input variable, they can accept parameters which can be of the following primitive types: Number, Boolean, String or can be a variable which will be evaluated from `options.data`.

There is a number of available formatters, but you can also **write your owns** in `options.formatters`.

```
Hi {{ name | uppercase }}. Today is {{ today | printDay('en-US') }}, tomorrow is {{ tomorrow | printDay('en-US') }}.
```

With the following `options`:

```js
{
  data: {
    name: "Courtoison",
    today: new Date(),
    tomorrow: new Date(new Date().setDate(new Date().getDate()+1))
  },
  formatters: {
    printDay: (date, locale) => date.toLocaleDateString(locale,{weekday: 'long'})
  }
}
```

Will become:

```
Hi COURTOISON. Today is Tuesday, tomorrow is Wednesday.
```

**Available formatters:**

- **lowercase**: `HeLLo` => `hello`
- **uppercase**: `wOrLd` => `WORLD`
- **capitalize**: `heLLO wOrlD` => `Hello World`
- **money(locale, currencyISO, fractionDigits = 0)**: `1500 | money("us", "USD", 2)` => `$1500.00`. (Currency ISO Codes)[https://en.wikipedia.org/wiki/ISO_4217#Active_codes]
- _More to come..._

_Warning: If you use an undefined formatter it will be simply ignored, which could lead to unexpected results if you're chaining them._

## How to retrieve the Google token?

If you are using `google-docs-mustaches` from inside a browser, you can follow [this tutorial](https://developers.google.com/api-client-library/javascript/start/start-js).

If you are using `google-docs-mustaches` in Node.js, you can follow [this one](https://github.com/googleapis/google-api-nodejs-client#oauth2-client).

> **Note:** Your AccessToken must have the following scopes:
>
> - https://www.googleapis.com/auth/drive
> - https://www.googleapis.com/auth/documents

## Want to help?

Great! If you want to contribute to **google-docs-mustaches**, go check out the [Contributor documentation](/dev) to get started!

## Supported environments

We use `cross-fetch` for compatibility with most environment.

Those are the `cross-fetch` supported environments:

- Node 6+
- React-Native

[![Browser compat](https://saucelabs.com/browser-matrix/cross-fetch.svg)](https://github.com/lquixada/cross-fetch)
