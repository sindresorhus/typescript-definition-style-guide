# TypeScript Definition Style Guide

> Style guide for adding type definitions to my npm packages

*Open an issue if anything is unclear or if you have ideas for other checklist items.*


## Checklist

- Use tab-indentation and semicolons.
- The definition should target the latest TypeScript version.
- Exported properties/methods should be documented (see below).
- The definition should be tested (see below).
- When you have to use Node.js types, install the `@types/node` package as a dev dependency and add the `/// <reference types="node"/>` triple-slash reference to the top of the definition file.
- When you have to use DOM types (`Window`, `Document`, …), add the `/// <reference lib="dom"/>` triple-slash reference to the top of the definition file.
- Third-party library types (everything in the `@types/*` namespace) must be installed as direct dependencies, if required. Prefer imports over triple-slash references. You usually only need a triple-slash reference for a third-party library if it exposes interfaces only in the global namespace.
- Ensure you're not falling for any of the [common mistakes](https://github.com/DefinitelyTyped/DefinitelyTyped/#common-mistakes).
- For packages with a default export, use `export = foo;` syntax. Only use `export foo ...` syntax if the package has no default export. Do not add a `namespace` unless you have to export types or interfaces. See more on this topic [here](https://github.com/DefinitelyTyped/DefinitelyTyped#should-i-add-an-empty-namespace-to-a-package-that-doesnt-export-a-module-to-use-es6-style-imports).
- Use the name `"types"` and not `"typings"` for the TypeScript definition field in package.json.
- If the entry file in the package is named `index.js`, name the type definition file `index.d.ts` and put it in root.<br>
	You don't need to add a `types` field to package.json as TypeScript will infer it from the name.
- Add the type definition file to the `files` field in package.json.
- The pull request should have the title `Add TypeScript definition`. *(Copy-paste it so you don't get it wrong)*
- Ignore any Travis linting failures. I will fix them after merging.
- **Help review [other pull requests](https://github.com/search?q=user%3Asindresorhus+is%3Apr+is%3Aopen+%22Add+TypeScript+definition%22&type=Issues) that adds a type definition.**

Check out [this](https://github.com/sindresorhus/write-json-file/blob/master/index.d.ts) and [this](https://github.com/sindresorhus/delay/blob/master/index.d.ts) example for how it should be done.

### Types

- All types used in the public interface should be added to an exported namespace or `export`'ed.
- Types should not have namespaced names; `interface Options {}`, not `interface FooOptions {}`, unless there are multiple `Options` interfaces.
- Use the array shorthand type notation; `number[]`, not `Array<number>`.
- Use the `readonly number[]` notation; not `ReadonlyArray<number>`.
- Prefer using the [`unknown` type](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html#new-unknown-top-type) instead of `any` whenever possible.
- Don't use abbreviation for type/variable/function names; `function foo(options: Options)`, not `function foo(opts: Opts)`.
- When there are more than one [generic type variable](https://www.typescriptlang.org/docs/handbook/generics.html#working-with-generic-type-variables) in a method, they should have descriptive names; `type Mapper<Element, NewElement> = …`, not `type Mapper<T, U> = …`.
- Don't prefix the name of interfaces with `I`; `Options`, not `IOptions`.
- Imports, destructuring, and object literals should *not* have spaces around the identifier; `{foo}`, not `{ foo }`.
- Don't use generic types like `object` or `Function`. Use specific type-signatures like `{[key: string]: number}` or `(input: string) => boolean;`.

#### Prefer read-only values

Make something read-only when it's not meant to be modified. This is usually the case for return values and option interfaces. Get familiar with the `readonly` keyword for [properties](https://www.typescriptlang.org/docs/handbook/interfaces.html#readonly-properties) and [array/tuple types](https://github.com/Microsoft/TypeScript/wiki/What's-new-in-TypeScript#improvements-for-readonlyarray-and-readonly-tuples). There's also a [`Readonly` type](https://basarat.gitbooks.io/typescript/docs/types/readonly.html) to mark all properties as `readonly`.

Before:

```ts
interface Point {
	x: number;
	y: number;
	children: Point[];
}
```

After:

```ts
interface Point {
	readonly x: number;
	readonly y: number;
	readonly children: readonly Point[];
}
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
import {Writable} from 'stream';
```

After:

```ts
import {Writable as WritableStream} from 'stream';
```

### Documentation

Exported definitions should be documented with [TSDoc](https://github.com/Microsoft/tsdoc). You can borrow text from the readme.

Example:

```ts
/// <reference lib="dom"/>

declare namespace add {
	interface Options {
		/**
		Allow negative numbers.

		@default true
		*/
		allowNegative?: boolean;

		/**
		Has the ultimate foo.

		Note: Only use this for good.

		@default false
		*/
		hasFoo?: boolean;

		/**
		Where to save.

		Default: [User's downloads directory](https://example.com)

		@example
		```
		import add = require('add');

		add(1, 2, {saveDirectory: '/my/awesome/dir'})
		```
		*/
		saveDirectory?: string;
	}
}

declare const add: {
	/**
	Add two numbers together.

	@param x - The first number to add.
	@param y - The second number to add.
	@returns The sum of `x` and `y`.
	*/
	(x: number, y: number, options?: add.Options): number;

	/**
	Reload the specified `BrowserWindow` instance or the focused one.

	@param window - Default: `BrowserWindow.getFocusedWindow()`
	*/
	refresh(window?: BrowserWindow): void;
}

export = add;
```

Note:

- Don't prefix lines with `*`.
- Don't [hard-wrap](https://stackoverflow.com/questions/319925/difference-between-hard-wrap-and-soft-wrap).
- Put an empty line between interface entries.
- Sentences should start with an uppercase character and end in a dot.
- There's an empty line after the function description.
- Parameters and the return value should be documented.
- There's a dash after the parameter name.
- `@param` should not include the parameter type.
- If the parameter description just repeats the parameter name, leave it out.
- If the parameter is `options` it doesn't need a description.
- If the function returns `void` or a wrapped `void` like `Promise<void>`, leave out `@returns`.
- If you include an `@example`, there should be a newline above it. The example itself should be wrapped with triple backticks (```` ``` ````).
- If the API accepts an options-object, define an `Options` interface as seen above. Document default option values using the [`@default` tag](https://jsdoc.app/tags-default.html) (since interfaces cannot have default values). If the default needs to be a description instead of a basic value, use the formatting `Default: Lorem Ipsum.`.
- Use `@returns`, not `@return`.
- Ambient declarations can't have default parameters, so in the case of a default method parameter, document it in the parameter docs instead, as seen in the above example.
- `@returns` should not duplicate the type information unless it's impossible to describe it without.
	- `@returns A boolean of whether it was enabled.` → `@returns Whether it was enabled.`

#### Code examples

- Include as many code examples as possible. Borrow from the readme.
- Code examples should be fully functional and should include the import statement.

### Testing

The type definition should be tested with [`tsd`](https://github.com/SamVerschueren/tsd). [Example of how to integrate it.](https://github.com/sindresorhus/delay/commit/594c42fa0f9f5f2997715d7e83dbd9e2e018e9aa)

Example:

```ts
import {expectType} from 'tsd';
import delay from '.';

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
