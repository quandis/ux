# Quandis Business Objects Roadmap

Quandis Business Objects is a suite of Fintech modules that our customers use to configure solutions specific to their needs.

Frequently, they request the ability to modify the UX in ways that are unique to their needs. QBO needs a composable UX strategy that enables the following use cases:

- Configuration of Single Page Applications to meet the needs of a specific business area
- Embedding QBO functionality in our client's public websites or intranets
- Customization of the UX on a per-user basis by end users

# Current UX

QBO's current UX is a combination of:

- API calls requesting data in `json`, `xml`, or `xhtml`
- Server-side rendering of `xhtml` via `xslt`
- Client-side implementation of behavior using the `Mootools` javascript framework
- Client-side implementation of styling using Bootstrap 2.3.2 (with additional styles and overrides added by Quandis)

QBO3 includes the following design widgets:

|Widget|Description|
|-|-|
|Home|Overview for a single module, including dashboards.|
|Summary|Composite view of a single record, including parents and children.|
|Select|Read-only details for a single record.|
|Edit|Edit details for a single record.|
|Search|Read-only details for multiple records.|
|Navbar|Navigation bar for a page.|
|TabGroup|Collection of tabbed panels.|
|Panel|Display of data from a single API call.|
|PanelMenu|Renders options that apply to multiple records.|
|ButtonMenu|Renders options that apply to a single record.|
|BreadCrumbs|Method suffixed with 'Home' or 'Summary'.|

Existing XSLTs can be founds in our [qbo3 repository](https://dev.azure.com/quandis/qbo3/_git/qbo3) under source > application tier > {module} > Templates (examples [here](https://dev.azure.com/quandis/qbo3/_git/qbo3?path=/src/application%20tier/decision/Templates/Decision) and [here](https://dev.azure.com/quandis/qbo3/_git/qbo3?path=/src/application%20tier/application/Templates/Application)).
Key callouts include:

|Item|Description|
|-|-|
|[Theme.core.xslt](https://dev.azure.com/quandis/qbo3/_git/qbo3?path=/src/application%20tier/application/Templates/Theme.Core.xslt)|Current implementation of rendering each of the widgets described above.|
|[Behaviors](https://dev.azure.com/quandis/qbo3/_git/qbo3?path=/src/web%20tier/application/Scripts)|Mootools-based javascript components that implement client-side behaviors.|
|[Custom Routes](https://dev.azure.com/quandisopensource/Documentation/_wiki/wikis/Documentation.wiki/179/Custom-Routes)|This functionality allows a power user to create a new endpoint in QBO via configuration.|
|[qbo4.UI](https://dev.azure.com/quandis/qbo4/_git/qbo4.UI)|A framework-agnostic rendering engine for configuration-driven UX.|

# Future UX

Maintenance of XSLT is beyond that of a typical qbo power users (often a business analyst). We needs to implement a composable UX strategy where power users can render a single page application.

At a minimum, the creation and maintenance of the UX can be done with some sort of XML markup, perhaps along these lines:

```xml
<ui>
    <navbar>
        <tab name="Summary">
        <tab name="Processes" linkto="Process">
        <tab name="Other">
    </navbar>
    <panelGroup name="Summary">
        <panel object="Loan" method="Select" parameters="ID={LoanID}"/> // dot-notation substitution: qbo4.Common
        <if "{PropertyID} > ''">
            <panel object="Property" method="Select" parameters="ID={PropertyID}"/>
        </if>
    </panelGroup>
    <panelGroup name="Process">
        <panel object="Foreclosure" method="Search" parameters="LoanID={LoanID}"/> // querystring format
        <panel object="Bankruptcy" method="Search" parameters="{Object: 'Loan', ObjectID: '{Loan.LoanID}'}"/> // or json format
        <panel object="Process">    // or parameters are child nodes
            <method>Search</method>
            <sibling></sibling> // only Sibling == NULL
        </panel>
    </panelGroup>
    <panelGroup name="Other">
        <for-each object="ImportFormTemplate" method="Search" parameters="ParentObject=Loan&Repeatable=0,1">
            <panel object="Task" name="ImportForm.ImportForm" method="Render" parameters="ID={ImportFormID}"/>
        </for-each>
        <panel object="NewModule" method="SomeApiCall" parameters="{...}">
            <search>
                <row>
                    <columns>
                        <NewModuleID/>
                        <NewModule>
                            <a href="https://www.google.com/q?={NewModule}">{NewModule}</a>
                        </NewModule>
                        <Labels object="NewModule" objectid="{NewModuleID}"/>
                    </columns>
                </row>
                <options></options>
            </search>
        </newModule>
        <myCustomUIPlugin param1="{LoanID}"/>   // extend our definition of UI widgets with a plugin
    </panelGroup>
</ui>
```

The `qbo4.UI.Web` project contains the web components to add sugar to any UI, including qbo3 and qbo4 UI.

This replace's the qbo3/Mootools `Behaviors` concept. Web components are reusable in any modern framework.

## Background Reading

- [HTML Accessiblilty](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/HTML)
- [Accessible Rich Internet Applications (ARIA)](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA)
- [Web components](https://developer.mozilla.org/en-US/docs/Web/Web_Components) ([examples](https://github.com/mdn/web-components-examples) on Github)
- [Form validation](https://www.w3.org/TR/2009/WD-html5-20090825/forms.html#attr-input-pattern)
- [Javascript Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [TSDoc](https://tsdoc.org/) for Typescript comments

## Custom Web components

There is a [debate](https://github.com/WebKit/standards-positions/issues/97) about customizing standard html elements.
Apple has refused to support customizing existing elements, and Quandis agrees with this approach.

This means that you can do:

```html
<qbo-select></qbo-select>
```

but you may not do:

```html
<select is="qbo-select"></select>
```

Components that customize `<select>` should do this:

```javascript
@customElement('qbo-select')
export class QboSelect extends QboFetch
{
    ...
    render() {
        ...
        return html`
            <select>
                ${this.options.map(option => html`...`)}
            </select>`;
    }
}
```

# Project Structure

While this project is a `Microsoft.NET.Sdk.Web` project, it does not include any meaningful server-side code.

## Technology Components

| Technology | Config File | Purpose |
|-|-|-|
|Typescript|`tsconfig.json`|Type safe version of Javascript. `tsc --build` is used to compile (transpile) .ts files to .js file.|
|Node package manager (npm)|`package.json`|Javascript equivalent to Nuget package manager.|
|Webpack|`webpack.config.cjs`|Javascript equivalent to MSBuild. Webpack products a single `.js` and `.css` file for use by other projects.|
|Lit|--|Google's web component library. `LitElement` is the base class for most qbo4 web components.|
|web-test-runner|--|A test runner that runs tests in browser(s)' javascript/DOM engine.|

## Folder Structure

|Folder|Purpose|
|-|-|
|`wwwroot`|Contains .html pages to enable testing of web components.|
|`wwwroot/scripts`|Contains `main.js` and `main.css` files to be linked to by `.html` pages.|
|`package`|Folder containing the files to be packaged by `npm` and pushed to an artifact feed. This is specified in `package.json`|
|`src`|Source `.ts` files that comprise qbo4.UI.Web's components and libaries.|
|`tests`|Test `*.test.ts` files for testing components and libraries.|

## Typescript (tsconfig.json)

This project includes the `Microsoft.Typecript.MSBuild` Nuget package, which includes the `tsc` compiler.
When the project is built, MSBuild automatically calls `tsc` to compile `.ts` files to `.js` files.

It is convention to place `.ts` files in a `src` folder, and `.test.ts` files in a `tests` folder.
However, when we publish the package, we want to include only the `src` files. 
To address this issue, we have three `tsconfig.json` files:

- `tsconfig.json`: in the root folder
- `src\tsconfig.json`: in the `src` folder, and instructs `tsc` where to output the files (`package` folder)
- `tests\tsconfig.json`: in the `tests` folder, and instructs `tsc` where to output the files (`tests` (same) folder)

The significant takeaway for our use of `tsconfig.json` is that we target `ESNext` for our output, 
meaning that we produce javascript files that use the latest `ECMA` standards, including `import` and `export` of `modules`.
This means that these libraries will only work with modern browsers, which comprise 94+% of all browsers currently in use.
**IE is right out**.

## Node (package.json)

The `package.json` is used instruct `npm` how to behave. While it has many options, 
the most relevent are the defined `scripts` that allow you to execute node command with a shortcut:

```console
// execute the tests using web-test-runner
npm run test
// package the files for deployment
npm run pack
```

The following `npm` commands are not scripts, but you will use frequently:

```console
// install all packages listed in package.json
npm install
// install a package and add it to package.json
npm install <package> --save
// install a package and add it to package.json as a dev dependency
npm install <package> --save-dev
// run any package installed globally
npx <package>
```

## Webpack (webpack.config.cjs)

Typescript projects must be transpiled into javascript, and will leverage modules via `import` and `export`.
Convention dictates that:

```typescript
import `lit`;   // This expects lit to be defined in the folder node_modules\lit
import `./qbo`  // This expects qbo to be defined in the current folder
```

Webpack's job is to look a one (or more) 'entry' javascript files, build a dependency graph of all the imports actually in use,
and create a single Javascript containing all referenced code (a dependency graph).
The entry point for qbo modules is `qbo.js`.

Webpack uses a `webpack.config.cjs` configuration file by default, that will product a `main.js` and `main.css` in the `wwwroot/scripts` folder.

> Webpack looks for a `webpack.config.js` first, but expects the json to be `CommonJs`, 
instead of a `module` as specified in `package.json` - as needed to tell node our modules use `import` instead of `require`. 
Naming the file with `.cjs` tells `node` that file is `CommonJS`, regardless of what `package.json's` `target` says.

The `qbo.UI.Web.csproj` includes a build target for `webpack`, so that every time the project is built, a new `main.js` and `main.css` is created.

```xml
<Target Name="NpmRunPack" AfterTargets="Build">
	<Exec Command="npm run pack" />
</Target>
```

paird with the `pack` script defined in `pacakge.json`:

```json
{ ...
  "scripts": {
    "pack": "webpack --mode=development --no-color"
  },
  ...
}
```

# HTML Coding Guidelines

When creating web pages, features should be implemented with:

- HTML, first, if possible,
- CSS, second, if possible,
- Javascript, otherwise

## Web Components

When designing reusable browser-based components, [web components](https://developer.mozilla.org/en-US/docs/Web/Web_Components) should be used. 
Web components use Javascript, HTML and CSS to define new tags ~~or extend existing tags~~ , and scope all functionality to a [shadow DOM](https://developers.google.com/web/fundamentals/web-components/shadowdom). 
This allows a given component to use javascript without worrying that it will interfere with other components.

The goal of creating web components is to strive to ensure QBO components can be **mixed into a variety of frameworks**, 
including Angular and React.

In most cases, Quandis recommends use of Google's [Lit library](https://lit.dev/) to speed the development of pure web components.

A sampling custom components includes:

|Component|Purpose|
|-|-|
|`qbo-fetch`|Acts as a base class for elements that need to do API calls (via Javascript's `fetch` statement).|
|`qbo-select`|Renders a `<select>` element with options from the results of an API call.|
|`qbo-popup`|Renders a popup, with content optionally coming from an API call (via `qbo-fetch`).|
|`qbo-logging`|Provides UI around handling errors.|

## Form validation

HTML5 supports [extensive form validation](https://www.w3.org/TR/2009/WD-html5-20090825/forms.html#attr-input-pattern); use it!  Examples:

```html
<input name="zipcode" type="text" pattern="\d{5}(-\d{4})?"/>
<input name="creditcard" type="text" pattern="[0-9]{13,16}"/>
```

# Typescrpit / Javascript Coding Guidelines

- Use the [MDN Style Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide) (which Microsoft also uses)
- Use `const` and `let` [instead of](https://stackoverflow.com/questions/40070631/v8-javascript-performance-implications-of-const-let-and-var/40070682#40070682) `var`
- Use camelcase for variables and function names

``` javascript
const myLongVariableName = "foo";
```

- Use template literals for code readability

``` javascript
const firstName = "Bob";
const lastName = "Jones";
const result = `Hello ${firstName}, ${lastName}`;
```

- Omit the protocol from links to other sites, if possible, to support `http` and `https`

``` javascript
<style src="//maps.googleapis.com/maps/api/js"/>
```

- Comment all methods using Microsoft standards

## Namespacing

Use [`modules`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) via the `export` and `import` features to build javascript libraries.
Related classes and functions should be imported as a module. For example, qbo4-related modules:

- src
  - qbo.ts
  - qbo-fetch.ts
  - qbo-json.ts
  - qbo-select.ts

where `qbo.js` [aggregates](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules#aggregating_modules) related classes an functions:

```javascript
export * from './qbo-fetch.js'
export * from './qbo-json.js'
export * from './qbo-select.js'
```
and the website's root includes a `main.js` created by `webpack`:

```javascript
import * as qbo4 from './main.js';
```
To use qbo4 from a browser debugger window:

```javascript
const qbo = await import('/scirpts/main.js'); 
// now you can access qbo.* functions
```

## Error handling in the browser

- Assume our code will fail
- Log errors to the server
- Our code, not the browser, should handle errors
- Identify errors that might occur (arguments, network)
  - use [custom errors](https://medium.com/@iaincollins/error-handling-in-javascript-a6172ccdf9af)
- `throw` at a low level, `catch` at a high level
- Distinguish between fatal and non-fatal
- Global debug mode: help the developer figure out the problem

> Credit [Nicholas Zakas](https://www.slideshare.net/nzakas/enterprise-javascript-error-handling-presentation/48-E_tcetera_M_y_blog)

## Comments

Quandis shall follow the [TSDoc](https://tsdoc.org/) to work with Visual Studio Intellisense.

``` javascript
/* @description These are sample comments
 * @param foo {string} Foo is used to ...
 * @param bar {array} Each item in bar is used to ...
 * @returns {object} An object representing ...
*/
qbo4.someMethod = function (foo, bar) {...}
```
