# TypeScript Definition Style Guide

> Style guide for adding type definitions to my npm packages

*Open an issue if anything is unclear or if you have ideas for other checklist items.*

This style guide assumes your package is native ESM.

## Checklist

- Use tab-indentation and semicolons.
- The definition should target the latest TypeScript version.
- Exported properties/methods should be documented (see below).
- The definition should be tested (see below).
- When you have to use Node.js types, install the `@types/node` package as a dev dependency. **Do not** add a `/// <reference types="node"/>` triple-slash reference to the top of the definition file.
- Third-party library types (everything in the `@types/*` namespace) must be installed as direct dependencies, if required. Use imports, not triple-slash references.
- Ensure you're not falling for any of the [common mistakes](https://github.com/DefinitelyTyped/DefinitelyTyped/#common-mistakes).
- For packages with a default export, use `export default function foo(…)` syntax.
- Do not use `namespace`.
- Use the name `"types"` and not `"typings"` for the TypeScript definition field in package.json.
- Place `"types"` in package.json after all official package properties, but before custom properties, preferably after `"dependencies"` and/or `"devDependencies"`.
- If the entry file in the package is named `index.js`, name the type definition file `index.d.ts` and put it in root.\
	You don't need to add a `types` field to package.json as TypeScript will infer it from the name.
- Add the type definition file to the `files` field in package.json.
- The pull request should have the title `Add TypeScript definition`. *(Copy-paste it so you don't get it wrong)*
- **Help review [other pull requests](https://github.com/search?q=user%3Asindresorhus+is%3Apr+is%3Aopen+%22Add+TypeScript+definition%22&type=Issues) that adds a type definition.**

Check out [this](https://github.com/sindresorhus/filled-array/commit/aae7539cb32f163cb063499664b012d0b04b3104), [this](https://github.com/sindresorhus/write-json-file/blob/main/index.d.ts), and [this](https://github.com/sindresorhus/delay/blob/main/index.d.ts) example for how it should be done.

### Types

- Types should not have namespaced names; `type Options {}`, not `type FooOptions {}`, unless there are multiple `Options` interfaces.
- Use the array shorthand type notation; `number[]`, not `Array<number>`.
- Use the `readonly number[]` notation; not `ReadonlyArray<number>`.
- Prefer using the [`unknown` type](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html#new-unknown-top-type) instead of `any` whenever possible.
- Don't use abbreviation for type/variable/function names; `function foo(options: Options)`, not `function foo(opts: Opts)`.
- When there are more than one [generic type variable](https://www.typescriptlang.org/docs/handbook/generics.html#working-with-generic-type-variables) in a method, they should have descriptive names; `type Mapper<Element, NewElement> = …`, not `type Mapper<T, U> = …`.
- Don't prefix the name of interfaces with `I`; `Options`, not `IOptions`.
- Imports, destructuring, and object literals should *not* have spaces around the identifier; `{foo}`, not `{ foo }`.
- Don't use permissive types like `object` or `Function`. Use specific type-signatures like `Record<string, number>` or `(input: string) => boolean;`.
- Use `Record<string, any>` for accepting objects with string index type and `Record<string, unknown>` for returning such objects. The reason `any` is used for assignment is that TypeScript has special behavior for it:
	> The index signature `Record<string, any>` in TypeScript behaves specially: it’s a valid assignment target for any object type. This is a special rule, since types with index signatures don’t normally produce this behavior.

#### Prefer read-only values

Make something read-only when it's not meant to be modified. This is usually the case for return values and option interfaces. Get familiar with the `readonly` keyword for [properties](https://www.typescriptlang.org/docs/handbook/interfaces.html#readonly-properties) and [array/tuple types](https://github.com/Microsoft/TypeScript/wiki/What's-new-in-TypeScript#improvements-for-readonlyarray-and-readonly-tuples). There's also a [`Readonly` type](https://basarat.gitbooks.io/typescript/docs/types/readonly.html) to mark all properties as `readonly`.

Before:

```ts
type Point = {
	x: number;
	y: number;
	children: Point[];
};
```

After:

```ts
type Point = {
	readonly x: number;
	readonly y: number;
	readonly children: readonly Point[];
};
```

#### Import types explicitly

Don't use implicit global types except for built-ins or when they can't be imported.

Before:

```ts
export function getWindow(): Electron.BrowserWindow;
```

After:

```ts
import {BrowserWindow} from 'electron';

export function getWindow(): BrowserWindow;
```

#### Readable named imports

Use a readable name when using named imports.

Before:

```ts
import {Writable} from 'node:stream';
```

After:

```ts
import {Writable as WritableStream} from 'node:stream';
```

### Documentation

Exported definitions should be documented with [TSDoc](https://github.com/Microsoft/tsdoc). You can borrow text from the readme.

Example:

```ts
export type Options = {
	/**
	Allow negative numbers.

	@default true
	*/
	readonly allowNegative?: boolean;

	/**
	Has the ultimate foo.

	Note: Only use this for good.

	@default false
	*/
	readonly hasFoo?: boolean;

	/**
	Where to save.

	Default: [User's downloads directory](https://example.com)

	@example
	```
	import add from 'add';

	add(1, 2, {saveDirectory: '/my/awesome/dir'})
	```
	*/
	readonly saveDirectory?: string;
};

/**
Add two numbers together.

@param x - The first number to add.
@param y - The second number to add.
@returns The sum of `x` and `y`.
*/
export default function add(x: number, y: number, options?: Options): number;
```

Note:

- Don't prefix lines with `*`.
- Don't [hard-wrap](https://stackoverflow.com/questions/319925/difference-between-hard-wrap-and-soft-wrap).
- Put an empty line between type entries.
- Sentences should start with an uppercase character and end in a dot.
- There's an empty line after the function description.
- Parameters and the return value should be documented.
- There's a dash after the parameter name.
- `@param` should not include the parameter type.
- If the parameter description just repeats the parameter name, leave it out.
- If the parameter is `options` it doesn't need a description.
- If the function returns `void` or a wrapped `void` like `Promise<void>`, leave out `@returns`.
- If you include an `@example`, there should be a newline above it. The example itself should be wrapped with triple backticks (```` ``` ````).
- If the API accepts an options-object, define an `Options` type as seen above. Document default option values using the [`@default` tag](https://jsdoc.app/tags-default.html) (since type cannot have default values). If the default needs to be a description instead of a basic value, use the formatting `Default: Lorem Ipsum.`.
- Use `@returns`, not `@return`.
- Ambient declarations can't have default parameters, so in the case of a default method parameter, document it in the parameter docs instead, as seen in the above example.
- `@returns` should not duplicate the type information unless it's impossible to describe it without.
	- `@returns A boolean of whether it was enabled.` → `@returns Whether it was enabled.`

#### Code examples

- Include as many code examples as possible. Borrow from the readme.
- Code examples should be fully functional and should include the import statement.

### Testing

The type definition should be tested with [`tsd`](https://github.com/SamVerschueren/tsd). [Example of how to integrate it.](https://github.com/sindresorhus/filled-array/commit/aae7539cb32f163cb063499664b012d0b04b3104)

Example:

```ts
import {expectType} from 'tsd';
import delay from './index.js';

expectType<Promise<void>>(delay(200));

expectType<Promise<string>>(delay(200, {value: '🦄'}));
expectType<Promise<number>>(delay(200, {value: 0}));

expectType<Promise<never>>(delay.reject(200, {value: '🦄'}));
expectType<Promise<never>>(delay.reject(200, {value: 0}));
```

When it makes sense, also add a negative test using [`expectError()`](https://github.com/SamVerschueren/tsd#expecterrorfunction).

Note:

- The test file should be named `index.test-d.ts`.
- `tsd` supports top-level `await`.
- When testing promise-returning functions, don't use the `await` keyword. Instead, directly assert for a `Promise`, like in the example above. When you use `await`, your function can potentially return a bare value without being wrapped in a `Promise`, since `await` will happily accept non-`Promise` values, rendering your test meaningless.
- Use [`const` assertions](https://github.com/Microsoft/TypeScript/wiki/What's-new-in-TypeScript#const-assertions) when you need to pass literal or readonly typed values to functions in your tests.
