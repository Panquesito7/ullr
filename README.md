Functional Web Components

**Building Web Components with Functional Programming.**

[![Build Status](https://travis-ci.org/aggre/ullr.svg?branch=master)](https://travis-ci.org/aggre/ullr)

```
 __  __     __         __         ______
/\ \/\ \   /\ \       /\ \       /\  == \
\ \ \_\ \  \ \ \____  \ \ \____  \ \  __<
 \ \_____\  \ \_____\  \ \_____\  \ \_\ \_\
  \/_____/   \/_____/   \/_____/   \/_/ /_/
```

---

# Installation

```
npm i ullr lit-html rxjs
```

# APIs

## `component`

`component` is a lit-html directive.

Encapsulate the template with Shadow DOM.

```ts
import { html } from 'lit-html'
import { component } from 'ullr/directive'

export const main = (title: string, desc: string) => html`
	${component(html`
		<style>
			h1 {
				color: blue;
			}
		</style>
		<main>
			<h1>${title}</h1>
			<p>${desc}</p>
		</main>
	`)}
`
```

## `subscribe`

`subscribe` is a lit-html directive.

Subscribe to `Observable<T>` of RxJS and re-rendering with callback function.

```ts
import { html } from 'lit-html'
import { subscribe } from 'ullr/directive'
import { timer } from 'rxjs'

export const template = html`
	<main>
		${subscribe(timer(10, 1), x => html`<p>${x}</p>`, html`<p>Default content</p>`)}
	</main>
`
```

## `customElements`

`customElements` creates a class that can be passed to `customElements.define`.

```ts
import { customElements } from 'ullr'
import { main } from './main'

const xApp = customElements(([title, desc]) => main(title, desc), [
	'title',
	'desc'
])

window.customElements.define('x-app', xApp)
```

## `ssr`

Render the template with Node.js (server side).

The API receives a template and testing function. Continue rendering until the testing function returns `true`.

You can use the emulated DOM in the testing function. (Use `jsdom`)

```ts
import { ssr } from 'ullr/ssr'
import { main } from './main'
;(async () => {
	const [document, template] = await ssr(main, doc =>
		Boolean(doc.querySelector('main > p'))
	)

	console.log(document)
	// <html><head></head><body><main>...</main></body></html>

	console.log(template)
	// <main>...<p>...</p>...</main>
})()
```

You can optionally pass the rendering target and full-document.

```ts
import { ssr } from 'ullr/ssr'
import { main } from './main'
;(async () => {
	const [document, template] = await ssr(
		main,
		doc => Boolean(doc.querySelector('main > p')),
		{
			target: '#testApp',
			html: `
				<!DOCTYPE html><html><head></head><body><div id=myApp></div></body></html>
			`
		}
	)

	console.log(document)
	// <!DOCTYPE html><html><head></head><body><div id="myApp"><main>...</main></div></body></html>

	console.log(template)
	// <main>...<p>...</p>...</main>
})()
```

# Usage

Basic usage:

```ts
import { html, render } from 'lit-html'
import { component } from 'ullr/directive'

const template = (title: string, desc: string) => html`
	<h1>${title}</h1>
	<p>${desc}</p>
`

const app = (title: string, desc: string) => html`
	${component(html`
		<style>
			h1 {
				font-weight: 400;
			}
			p {
				font-size: 1rem;
			}
		</style>
		${template(title, desc)}
	`)}
`

render(app('The title', 'lorem ipsum'), document.getElementById('root'))
```

Create a Custom Elements:

```ts
import { html } from 'lit-html'
import { customElements } from 'ullr'
import { component } from 'ullr/directive'

const template = (title: string, desc: string) => html`
	<h1>${title}</h1>
	<p>${desc}</p>
`

const app = (title: string, desc: string) => html`
	${component(html`
		<style>
			h1 {
				font-weight: 400;
			}
			p {
				font-size: 1rem;
			}
		</style>
		${template(title, desc)}
	`)}
`

const xApp = customElements(([title, desc]) => app(title, desc), [
	'title',
	'desc'
])

window.customElements.define('x-app', xApp)
```

Template to subscribe RxJS and update:

```ts
import { subscribe } from 'ullr/directive'
import { timer as _timer } from 'rxjs'
import { take, filter } from 'rxjs/operators'

const timer = _timer(10, 1).pipe(
	filter(x => x > 0),
	take(10)
)

export const template = html`
	<main>
		${subscribe(timer, x => html`<p>${x}</p>`, html`<p>Default content</p>`)}
	</main>
`
```
