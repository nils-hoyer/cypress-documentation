---
layout: toc-top
title: Using Cypress
containerClass: faq
---

## {% fa fa-angle-right %} How do I get an element's text contents?

Cypress commands yield jQuery objects, so you can call methods on them.

If you're trying to assert on an element's text content:

```javascript
cy.get('div').should('have.text', 'foobarbaz')
```

If the text contains a {% url "non-breaking space" https://en.wikipedia.org/wiki/Non-breaking_space %} entity `&nbsp;` then use the Unicode character `\u00a0` instead of `&nbsp;`.

```html
<div>Hello&nbsp;world</div>
```

```javascript
cy.get('div').should('have.text', 'Hello\u00a0world')
```

If you'd like to work with the text prior to an assertion:

```javascript
cy.get('div').should(($div) => {
  const text = $div.text()

  expect(text).to.match(/foo/)
  expect(text).to.include('foo')
  expect(text).not.to.include('bar')
})
```

If you need to convert text to a number before checking if it is greater than 10:

```javascript
cy.get('div').invoke('text').then(parseFloat).should('be.gt', 10)
```

If you need to hold a reference or compare values of text:

```javascript
cy.get('div').invoke('text').then((text1) => {
  // do more work here

  // click the button which changes the div's text
  cy.get('button').click()

  // grab the div again and compare its previous text
  // to the current text
  cy.get('div').invoke('text').should((text2) => {
    expect(text1).not.to.eq(text2)
  })
})
```

jQuery's `.text()` method automatically calls `elem.textContent` under the hood. If you'd like to instead use `innerText` you can do the following:

```javascript
cy.get('div').should(($div) => {
  // access the native DOM element
  expect($div.get(0).innerText).to.eq('foobarbaz')
})
```

This is the equivalent of Selenium's `getText()` method, which returns the innerText of a visible element.

## {% fa fa-angle-right %} How do I get an input's value?

Cypress yields you jQuery objects, so you can call methods on them.

If you're trying to assert on an input's value:

```javascript
// make an assertion on the value
cy.get('input').should('have.value', 'abc')
```

If you'd like to massage or work with the text prior to an assertion:

```javascript
cy.get('input').should(($input) => {
  const val = $input.val()

  expect(val).to.match(/foo/)
  expect(val).to.include('foo')
  expect(val).not.to.include('bar')
})
```

If you need to hold a reference or compare values of text:

```javascript
cy.get('input').invoke('val').then((val1) => {
  // do more work here

  // click the button which changes the input's value
  cy.get('button').click()

  // grab the input again and compare its previous value
  // to the current value
  cy.get('input').invoke('val').should((val2) => {
    expect(val1).not.to.eq(val2)
  })
})
```

## {% fa fa-angle-right %} How do I compare the value or state of one thing to another?

Our {% url 'Variables and Aliases guide' variables-and-aliases %} gives you examples of doing exactly that.

## {% fa fa-angle-right %} Can I store an attribute's value in a constant or a variable for later use?

Yes, and there are a couple of ways to do this. One way to hold a value or reference is with {% url '`closures`' variables-and-aliases#Closures %}.
Commonly, users believe they have a need to store a value in a `const`, `var`, or `let`. Cypress recommends doing this only when dealing with mutable objects (that change state).

For examples how to do this, please read our {% url 'Variables and Aliases guide' variables-and-aliases %}.

## {% fa fa-angle-right %} How do I get the native DOM reference of an element found using Cypress?

Cypress wraps elements in jQuery so you'd get the native element from there within a {% url "`.then()`" then %} command.

```javascript
cy.get('button').then(($el) => {
  $el.get(0)
})
```

## {% fa fa-angle-right %} How do I do something different if an element doesn't exist?

What you're asking about is conditional testing and control flow.

Please read our extensive {% url 'Conditional Testing Guide' conditional-testing %} which explains this in detail.

## {% fa fa-angle-right %} How can I make Cypress wait until something is visible in the DOM?

{% note info Remember %}
DOM based commands will automatically {% url "retry" retry-ability %} and wait for their corresponding elements to exist before failing.
{% endnote %}

Cypress offers you many robust ways to {% url 'query the DOM' introduction-to-cypress#Querying-Elements %}, all wrapped with retry-and-timeout logic.

Other ways to wait for an element's presence in the DOM is through `timeouts`. Cypress commands have a default timeout of 4 seconds, however, most Cypress commands have {% url 'customizable timeout options' configuration#Timeouts %}. Timeouts can be configured globally or on a per-command basis.

In {% url 'some cases' interacting-with-elements#Visibility %}, your DOM element will not be actionable. Cypress gives you a powerful {%url '`{force:true}`' interacting-with-elements#Forcing %} option you can pass to most action commands.

**Please read** our {% url 'Core Concepts Introduction to Cypress' introduction-to-cypress %}. This is the single most important guide for understanding how to test with Cypress.

## {% fa fa-angle-right %} How do I wait for my application to load?

We have seen many different iterations of this question. The answers can be varied depending on how your application behaves and the circumstances under which you are testing it. Here are a few of the most common versions of this question.

**_How do I know if my page is done loading?_**

When you load your application using `cy.visit()`, Cypress will wait for the `load` event to fire. The {% url '`cy.visit()`' visit#Usage %} command loads a remote page and does not resolve until all of the external resources complete their loading phase. Because we expect your applications to observe differing load times, this command's default timeout is set to 60000ms. If you visit an invalid url or a {% url 'second unique domain' web-security#Same-superdomain-per-test %}, Cypress will log a verbose yet friendly error message.

**_In CI, how do I make sure my server has started?_**

We recommend these great modules for this use case:

* {% url '`wait-on`' https://www.npmjs.com/package/wait-on %}
* {% url '`start-server-and-test`' https://github.com/bahmutov/start-server-and-test %}

**_How can I wait for my requests to be complete?_**

The prescribed way to do this is to define your routes using {% url '`cy.intercept()`' intercept %}, create {% url '`aliases`' variables-and-aliases#Aliases %} for these routes prior to the visit, and _then_ you can explicitly tell Cypress which routes you want to wait on using {% url '`cy.wait()`' wait#Syntax %}. **There is no magical way to wait for all of your XHRs or Ajax requests.** Because of the asynchronous nature of these requests, Cypress cannot intuitively know to wait for them. You must define these routes and be able to unambiguously tell Cypress which requests you want to wait on.

## {% fa fa-angle-right %} Can I test the HTML `<head>` element?

Yes, you sure can. While executing tests in the Test Runner, you can view the entire `window.document` object in your open console using {% url '`cy.document()`' document %}. You can even make assertions on the `<head>` element. Check out this example.

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <meta http-equiv="Content-Security-Policy" content="default-src 'self'">
  <meta name="description" content="This description is so meta">
  <title>Test the HEAD content</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
</body>
</html>
```

```js
describe('The Document Metadata', () => {
  beforeEach(() => {
    cy.visit('/')
  })

  it('looks inside the head content using `cy.document()`', () => {
    // this will yield the entire window.document object
    // if you click on DOCUMENT from the command log,
    // it will output the entire #document to the console
    cy.document()
  })

  // or make assertions on any of the metadata in the head element

  it('looks inside <title> tag', () => {
    cy.get('head title')
      .should('contain', 'Test the HEAD content')
  })

  it('looks inside <meta> tag for description', () => {
    cy.get('head meta[name="description"]')
      .should('have.attr', 'content', 'This description is so meta')
  })
})
```

## {% fa fa-angle-right %} Can I check that a form's HTML form validation is shown when an input is invalid?

You certainly can.

```javascript
it('check validation message on invalid input', () => {
  cy.get('input:invalid').should('have.length', 0)
  cy.get('[type="email"]').type('not_an_email')
  cy.get('[type="submit"]').click()
  cy.get('input:invalid').should('have.length', 1)
  cy.get('[type="email"]').then(($input) => {
    expect($input[0].validationMessage).to.eq('I expect an email!')
  })
})
```


## {% fa fa-angle-right %} Can I throttle network speeds using Cypress?

You can throttle your network connection by accessing your Developer Tools Network panel. Additionally, you can add your own custom presets by selecting **Custom > Add** from the Network Conditions drawer.

We do not currently offer any options to simulate this during `cypress run`.

## {% fa fa-angle-right %} Can I use the new ES7 async / await syntax?

No. The Command API is not designed in a way that makes this possible. This is not a limitation of Cypress - it is a very conscious and important design decision.

Async / await is sugar around promises and Cypress commands are a mixture of both promises and streams.

If you're curious please read:

- Our {% url 'Introduction to Cypress guide' introduction-to-cypress#Commands-Are-Asynchronous %} which explains how the Commands are designed
- Our {% url 'Variables and Aliases guide' variables-and-aliases %} which talks about patterns dealing with async code

## {% fa fa-angle-right %} How do I select or query for elements if my application uses dynamic classes or dynamic IDs?

Don't use classes or ID's. You add `data-*` attributes to your elements and target them that way.

Read more about the {% url 'best practices for selecting elements here' best-practices#Selecting-Elements %}.

## {% fa fa-angle-right %} I want to run tests only within one specific folder. How do I do this?

You can specify which test files to run during {% url "`cypress run`" command-line#cypress-run %} by {% url "passing a glob to the `--spec` flag" command-line#cypress-run-spec-lt-spec-gt %} matching the files you want to run. You should be able to pass a glob matching the specific folder where the tests are you want to run.

This feature is not available when using {% url "`cypress open`" command-line#cypress-open %} however.

## {% fa fa-angle-right %} Is there a suggested way or best practice for how I should target elements or write element selectors?

Yes. Read more about the {% url 'best practices for selecting elements here' best-practices#Selecting-Elements %}.

## {% fa fa-angle-right %} Can I prevent Cypress from failing my test when my application throws an uncaught exception error?

Yes.

By default Cypress will automatically fail tests whenever an uncaught exception bubbles up out of your app.

Cypress exposes an event for this (amongst many others) that you can listen for to either:

- Debug the error instance itself
- Prevent Cypress from failing the test

This is documented in detail on the {% url "Catalog Of Events" catalog-of-events %} page and the recipe {% url 'Handling errors' recipes#Fundamentals %}.

## {% fa fa-angle-right %} Will Cypress fail the test when an application has unhandled rejected promise?

By default no, Cypress does not listen to the unhandled promise rejection event in your application, and thus does not fail the test. You can set up your own listener though and fail the test, see our recipe {% url 'Handling errors' recipes#Fundamentals %}:

```js
// register listener during cy.visit
it('fails on unhandled rejection', () => {
  cy.visit('/', {
    onBeforeLoad (win) {
      win.addEventListener('unhandledrejection', (event) => {
        const msg = `UNHANDLED PROMISE REJECTION: ${event.reason}`

        // fail the test
        throw new Error(msg)
      })
    },
  })
})

// ALTERNATIVE: register listener for this test
it('fails on unhandled rejection', () => {
  cy.on('window:before:load', (win) => {
    win.addEventListener('unhandledrejection', (event) => {
      const msg = `UNHANDLED PROMISE REJECTION: ${event.reason}`

      // fail the test
      throw new Error(msg)
    })
  })

  cy.visit('/')
})

// ALTERNATIVE: register listener in every test
before(() => {
  Cypress.on('window:before:load', (win) => {
    win.addEventListener('unhandledrejection', (event) => {
      const msg = `UNHANDLED PROMISE REJECTION: ${event.reason}`

      // fail the test
      throw new Error(msg)
    })
  })
})

it('fails on unhandled rejection', () => {
  cy.visit('/')
})
```

## {% fa fa-angle-right %} Can I override environment variables or create configuration for different environments?

Yes, you can pass configuration to Cypress via environment variables, CLI arguments, JSON files and other means.

{% url "Read the Environment Variables guide." environment-variables %}

## {% fa fa-angle-right %} Can I override or change the default user agent the browser uses?

Yes. {% url "You can override this with `userAgent` in your configuration file (`cypress.json` by default)." configuration#Browser %}

## {% fa fa-angle-right %} Can I block traffic going to specific domains? I want to block Google Analytics or other providers.

Yes. {% url "You can set this with `blockHosts` in your configuration file (`cypress.json` by default)." configuration#Browser %}

Also, check out our {% url 'Stubbing Google Analytics Recipe' recipes#Stubbing-and-spying %}.

## {% fa fa-angle-right %} How can I verify that calls to analytics like Google Analytics are being made correct?

You can stub their functions and then ensure they're being called.

Check out our {% url 'Stubbing Google Analytics Recipe' recipes#Stubbing-and-spying %}.

## {% fa fa-angle-right %} I'm trying to test a chat application. Can I run more than one browser at a time with Cypress?

{% url "We've answered this question in detail here." trade-offs#Multiple-browsers-open-at-the-same-time %}

## {% fa fa-angle-right %} Can I test a chrome extension? How do I load my chrome extension?

Yes. You can test your extensions by {% url 'loading them when we launch the browser.' browser-launch-api %}.

## {% fa fa-angle-right %} How can I modify or pass arguments used to launch the browser?

You use the {% url `before:browser:launch` browser-launch-api %} plugin event.

## {% fa fa-angle-right %} Can I make cy.request() poll until a condition is met?

Yes. You do it the {% url 'same way as any other recursive loop' request#Request-Polling %}.

## {% fa fa-angle-right %} Can I use the Page Object pattern?

Yes.

The page object pattern isn't actually anything "special". If you're coming from Selenium you may be accustomed to creating instances of classes, but this is completely unnecessary and irrelevant.

The "Page Object Pattern" should really be renamed to: "Using functions and creating custom commands".

If you're looking to abstract behavior or roll up a series of actions you can create reusable {% url 'Custom Commands with our API' custom-commands %}. You can also use regular ol' JavaScript functions without any of the ceremony typical with "Page Objects".

For those wanting to use page objects, we've highlighted the {% url 'best practices ' custom-commands#Best-Practices %} for replicating the page object pattern.

## {% fa fa-angle-right %} Why do my Cypress tests pass locally but not in CI?

There are many reasons why tests may fail in CI but pass locally. Some of these include:

- There is a problem isolated to the Electron browser (`cypress run` by default runs in the Electron browser)
- A test failure in CI could be highlighting a bug in your CI build process
- Variability in timing when running your application in CI (For example, network requests that resolve within the timeout locally may take longer in CI)
- Machine differences in CI versus your local machine -- CPU resources, environment variables, etc.

To troubleshoot why tests are failing in CI but passing locally, you can try these strategies:

- Test locally with Electron to identify if the issue is specific to the browser.
- You can also identify browser-specific issues by running in a different browser in CI with the `--browser` flag.
- Review your CI build process to ensure nothing is changing with your application that would result in failing tests.
- Remove time-sensitive variability in your tests. For example, ensure a network request has finished before looking for the DOM element that relies on the data from that network request. You can leverage {% url "aliasing" variables-and-aliases#Aliases %} for this.
- Ensure video recording and/or screenshots are enabled for the CI run and compare the recording to the Command Log when running the test locally.

## {% fa fa-angle-right %} Why are my video recordings freezing or dropping frames when running in CI?

Videos recorded on Continuous Integration may have frozen or dropped frames if there are not enough resources available when running the tests in your CI container. Like with any application, there needs to be the required CPU to run Cypress and record video. You can run your tests with {% url 'memory and CPU logs enabled' troubleshooting#Log-memory-and-CPU-usage %} to identify and evaluate the resource utilization within your CI.

If you are experiencing this issue, we recommend switching to a more powerful CI container or provider.

## {% fa fa-angle-right %} What can I do if my tests crash or hang on CI?

As some users have noted, a longer test has a higher chance of hanging or even crashing when running on CI. When a test runs for a long period of time, its commands and the application itself might allocate more memory than available, causing the crash. The exact risk of crashing depends on the application and the available hardware resources. While there is no single time limit that would solve this problem, in general we recommend splitting spec files to run in under one minute each. You can read the blog post {% url "Make Cypress Run Faster by Splitting Specs" https://glebbahmutov.com/blog/split-spec/ %} to learn how to split a spec file.

You can further split individual long-running tests. For example, you can verify parts of the longer user feature in the separate tests as described in {% url "Split a very long Cypress test into shorter ones using App Actions" https://www.cypress.io/blog/2019/10/29/split-a-very-long-cypress-test-into-shorter-ones-using-app-actions/ %}.

## {% fa fa-angle-right %} How can I parallelize my runs?

You can read more about parallelization {% url 'here' parallelization %}.

## {% fa fa-angle-right %} Can I run a single test or group of tests?

You can run a group of tests or a single test by placing an {% url `.only` writing-and-organizing-tests#Excluding-and-Including-Tests %} on a test suite or specific test.

You can run a single test file or group of tests by passing the `--spec` flag to {% url '`cypress run`' command-line#cypress-run %}.

## {% fa fa-angle-right %} How do I test uploading a file?

It is possible to upload files in your application but it's different based on how you've written your own upload code. Many people had success by using the community plugin {% url cypress-file-upload https://github.com/abramenal/cypress-file-upload %}. This plugin adds a custom child command `.attachFile` that you call from the test.

```javascript
// attaches the file cypress/fixtures/data.json
cy.get('[data-cy="file-input"]').attachFile('data.json')
```

You can read more about uploading files {% issue 170 'here' %}.

## {% fa fa-angle-right %} What is the projectId for?

The `projectId` is a 6 character string that helps identify your project once you've {% url "set up your tests to record" runs %}. It's generated by Cypress and typically is found in your {% url "configuration file (`cypress.json` by default)" configuration %} file.

```json
{
  "projectId": "a7bq2k"
}
```

For further detail see the {% url Identification projects#Identification %} section of the {% url "Dashboard Service" dashboard-introduction%} docs.

## {% fa fa-angle-right %} What is a Record Key?

A *Record Key* is a GUID that's generated automatically by Cypress once you've {% url "set up your tests to record" runs %}. It helps identify your project and authenticate that your project is even *allowed* to record tests.

You can find your project's record key inside of the *Settings* tab in the Test Runner.

{% imgTag /img/dashboard/record-key-shown-in-desktop-gui-configuration.jpg "Record Key in Configuration Tab" %}

For further detail see the {% url Identification projects#Identification %} section of the {% url "Dashboard Service" dashboard-introduction%} docs.

## {% fa fa-angle-right %} How do I check that an email was sent out?

{% note warning 'Anti-Pattern' %}
Don't try to use your UI to check email. Instead opt to programmatically use 3rd party APIs or talk directly to your server. Read about this {% url 'best practice' best-practices#Visiting-external-sites %} here.
{% endnote %}

## {% fa fa-angle-right %} How do I wait for multiple requests to the same url?

You should set up an alias (using {% url `.as()` as %}) to a single {% url `cy.intercept()` intercept %} that matches all of the XHRs. You can then {% url `cy.wait()` wait %} on it multiple times. Cypress keeps track of how many matching requests there are.

```javascript
cy.intercept('users').as('getUsers')
cy.wait('@getUsers')  // Wait for first GET to /users/
cy.get('#list>li').should('have.length', 10)
cy.get('#load-more-btn').click()
cy.wait('@getUsers')  // Wait for second GET to /users/
cy.get('#list>li').should('have.length', 20)
```

## {% fa fa-angle-right %} How do I seed / reset my database?

You can use {% url `cy.request()` request %}, {% url `cy.exec()` exec %}, or {% url `cy.task()` task %} to talk to your back end to seed data.

You could also stub requests directly using {% url `cy.intercept()` intercept %} which avoids ever even needing to fuss with your database.

## {% fa fa-angle-right %} How do I test elements inside an iframe?

As of {% issue 136#issuecomment-328100955 `0.20.0` %} you can now wrap the elements of an iframe and work with them.

We have an {% issue 685 'open proposal' %} to expand the APIs to support "switching into" an iframe and then back out of them.

## {% fa fa-angle-right %} How do I preserve cookies / localStorage in between my tests?

By default, Cypress automatically {% url "clears all cookies **before** each test" clearcookies %} to prevent state from building up.

You can preserve specific cookies across tests using the {% url "Cypress.Cookies api" cookies %}:

```javascript
// now any cookie with the name 'session_id' will
// not be cleared before each test runs
Cypress.Cookies.defaults({
  preserve: 'session_id'
})
```

You **cannot** currently preserve localStorage across tests and can read more {% issue '461#issuecomment-325402086' 'here' %}.

## {% fa fa-angle-right %} Some of my elements animate in; how do I work around that?

Oftentimes you can usually account for animation by asserting {% url "`.should('be.visible')`" should %} or {% url "another assertion" introduction-to-cypress#Assertions %} on one of the elements you expect to be animated in.

```javascript
// assuming a click event causes the animation
cy.get('.element').click().should('not.have.class', 'animating')
```

If the animation is especially long, you could extend the time Cypress waits for the assertion to pass by increasing the `timeout` of the previous command before the assertion.

```javascript
cy.get('button', { timeout: 10000 }) // wait up to 10 seconds for this 'button' to exist
  .should('be.visible')              // and to be visible

cy.get('.element').click({ timeout: 10000 }).should('not.have.class', 'animating')
// wait up to 10 seconds for the .element to not have 'animating' class
```

However, most of the time you don't even have to worry about animations. Why not?  Cypress will {% url "automatically wait" interacting-with-elements %} for elements to stop animating prior to interacting with them via action commands like `.click()` or `.type()`.

## {% fa fa-angle-right %} Can I test anchor links that open in a new tab?

Cypress does not and may never have multi-tab support for various reasons.

Luckily there are lots of clear and safe workarounds that enable you to test this behavior in your application.

{% url 'Read through the recipe on tab handling and links to see how to test anchor links.' recipes#Testing-the-DOM %}

## {% fa fa-angle-right %} Can I dynamically test multiple viewports?

Yes, you can. We provide an {% url 'example here' viewport#Width-Height %}.

## {% fa fa-angle-right %} Can I run the same tests on multiple subdomains?

Yes. In this example, we loop through an array of urls and make assertions on the logo.

```javascript
const urls = ['https://docs.cypress.io', 'https://www.cypress.io']

describe('Logo', () => {
  urls.forEach((url) => {
    it(`Should display logo on ${url}`, () => {
      cy.visit(url)
      cy.get('#logo img')
        .should('have.attr', 'src')
        .and('include', 'logo')
    })
  })
})
```

{% imgTag /img/faq/questions/command-log-of-dynamic-url-test.png "Command Log multiple urls" %}

## {% fa fa-angle-right %} How do I require or import node modules in Cypress?

The code you write in Cypress is executed in the browser, so you can import or require JS modules, *but* only those that work in a browser.

You can `require` or `import` them as you're accustomed to. We preprocess your spec files with webpack and Babel.

We recommend utilizing one of the following to execute code outside of the browser. Furthermore, you can use your own Node version during code excecution by setting the {% url "`nodeVersion`" configuration#Node-version %} in your configuration.

- {% url `cy.task()` task %} to run code in Node via the {% url "`pluginsFile`" configuration#Folders-Files %}
- {% url `cy.exec()` exec %} to execute a shell command

{% url 'Check out the "Node Modules" example recipe.' recipes#Fundamentals %}

## {% fa fa-angle-right %} Is there a way to give a proper SSL certificate to your proxy so the page doesn't show up as "not secure"?

No, Cypress modifies network traffic in real time and therefore must sit between your server and the browser. There is no other way for us to achieve that.

## {% fa fa-angle-right %} Is there any way to detect if my app is running under Cypress?

You can check for the existence of `window.Cypress`, in your **application code**.

Here's an example:

```javascript
if (window.Cypress) {
  // we are running in Cypress
  // so do something different here
  window.env = 'test'
} else {
  // we are running in a regular ol' browser
}
```

## {% fa fa-angle-right %} Do you allow before, beforeEach, after, or afterEach hooks?

Yes. You can read more {% url "here" writing-and-organizing-tests#Hooks %}.

## {% fa fa-angle-right %} I tried to install Cypress in my CI, but I get the error: `EACCES: permission denied`.

First, make sure you have {% url "Node" https://nodejs.org %} installed on your system. `npm` is a Node package that is installed globally by default when you install Node and is required to install our {% url "`cypress` npm  package" command-line %}.

Next, you'd want to check that you have the proper permissions for installing on your system or you may need to run `sudo npm install cypress`.

## {% fa fa-angle-right %} Is there a way to test that a file got downloaded? I want to test that a button click triggers a download.

There are a lot of ways to test this, so it depends. You'll need to be aware of what actually causes the download, then think of a way to test that mechanism.

If your server sends specific disposition headers which cause a browser to prompt for download, you can figure out what URL this request is made to, and use {% url "cy.request()" request %} to hit that directly. Then you can test that the server send the right response headers.

If it's an anchor that initiates the download, you could test that it has the right `href` property. As long as you can verify that clicking the button is going to make the right HTTP request, that might be enough to test for.

Finally, if you want to really download the file and verify its contents, see our {% url "File download" https://github.com/cypress-io/cypress-example-recipes#testing-the-dom %} recipe.

In the end, it's up to you to know your implementation and to test enough to cover everything.

## {% fa fa-angle-right %} Is it possible to catch the promise chain in Cypress?

No. You cannot add a `.catch` error handler to a failed command. {% url "Read more about how the Cypress commands are not Promises" introduction-to-cypress#Commands-Are-Not-Promises %}

## {% fa fa-angle-right %} Is there a way to modify the screenshots/video resolution?

Not at the moment. {% issue 587 "There is an open issue for this." %}

## {% fa fa-angle-right %} Does Cypress support ES7?

Yes. You can customize how specs are processed by using one of our {% url 'preprocessor plugins' plugins %} or by {% url 'writing your own custom preprocessor' preprocessors-api %}.

Typically you'd reuse your existing Babel and webpack configurations.

## {% fa fa-angle-right %} How does one determine what the latest version of Cypress is?

There are a few ways.

- The easiest way is probably to check our {% url "changelog" changelog %}.
- You can also check the latest version {% url "here" https://download.cypress.io/desktop.json %}.
- It's also always in our {% url "repo" https://github.com/cypress-io/cypress %}.

## {% fa fa-angle-right %} Is there an ESLint plugin for Cypress or a list of globals?

Yes! Check out our {% url "ESLint plugin" https://github.com/cypress-io/eslint-plugin-cypress %}. It will set up all the globals you need for running Cypress, including browser globals and {% url "Mocha" https://mochajs.org/ %} globals.

## {% fa fa-angle-right %} When I visit my site directly, the certificate is verified, however the browser launched through Cypress is showing it as "Not Secure". Why?

This is normal. Cypress modifies the traffic between your server and the browser. The browser notices this and displays a certificate warning. However, this is purely cosmetic and does not alter the way your application under test runs in any way, so you can safely ignore this warning.

## {% fa fa-angle-right %} Is there an option to run Cypress in CI with Developer Tools open? We want to track network and console issues.

No. This is definitely the motivation behind {% issue 448 "this open issue" %}, but there is not a way to run Cypress in `cypress run` with Developer Tools open.

You may try running the tests locally and {% url "select the Electron browser" launching-browsers#Electron-Browser %}, that is as close as you will get with Developer Tools open and replicating the environment that was run during `cypress run`.

## {% fa fa-angle-right %} How do I run the server and tests together and then shutdown the server?

To start the server, run the tests and then shutdown the server we recommend {% url "these npm tools" continuous-integration#Boot-your-server %}.

## {% fa fa-angle-right %} Can I test my Electron app?

Testing your Electron app will not 'just work', as Cypress is designed to test anything that runs in a browser and Electron is a browser + Node.

That being said, we use Cypress to test our own Desktop app's front end - by stubbing events from Electron. These tests are open source so you can check them out {% url "here" https://github.com/cypress-io/cypress/tree/develop/packages/desktop-gui/cypress/integration %}.

## {% fa fa-angle-right %} I found a bug! What do I do?

- Search existing {% url "open issues" https://github.com/cypress-io/cypress/issues %}, it may already be reported!
- Update Cypress. Your issue may have {% url "already been fixed" changelog %}.
- {% open_an_issue %}. Your best chance of getting a bug looked at quickly is to provide a repository with a reproducible bug that can be cloned and run.

## {% fa fa-angle-right %} What are your best practices for organizing tests?

We see organizations _starting_ with Cypress by placing end-to-end tests in a separate repo. This is a great practice that allows someone on the team to prototype a few tests and evaluate Cypress within minutes. As the time passes and the number of tests grows, we _strongly suggest_ moving end-to-end tests to live right alongside your front end code. This brings many benefits:

- engages developers in writing end-to-end tests sooner
- keeps tests and the features they test in sync
- tests can be run every time the code changes
- allows code sharing between the application code and the tests (like selectors)

## {% fa fa-angle-right %} What is the right balance between custom commands and utility functions?

There is already a great section in {% url "Custom Commands" custom-commands#Best-Practices %} guide that talks about trade-offs between custom commands and utility functions. We feel reusable functions in general are a way to go. Plus they do not confuse {% issue 1065 'IntelliSense like custom commands do' %}.

## {% fa fa-angle-right %} Can I print the list of commands from a test in the terminal?

If a test fails, Cypress takes a screenshot image, but does not print the list of commands in the terminal, only the failed assertion. There is a user space plugin {% url cypress-failed-log https://github.com/bahmutov/cypress-failed-log %} that saves a JSON file with all commands from a failed test. We are also working on mirroring `console.log` messages from the browser in the terminal, see {% issue 2078 %}.

## {% fa fa-angle-right %} Can my tests interact with Redux / Vuex data store?

Usually your end-to-end tests interact with the application through public browser APIs: DOM, network, storage, etc. But sometimes you might want to make assertions against the data held inside the application's data store. Cypress helps you do this. Tests run right in the same browser instance and can reach into the application's context using {% url `cy.window` window %}. By conditionally exposing the application reference and data store from the application's code, you can allow the tests to make assertions about the data store, and even drive the application via Redux actions.

- see {% url "Testing Redux Store" https://www.cypress.io/blog/2018/11/14/testing-redux-store/ %} blog post and {% url "Redux Testing" recipes#Blogs %} recipe.
- see {% url "Testing Vue web applications with Vuex data store & REST back end" https://www.cypress.io/blog/2017/11/28/testing-vue-web-application-with-vuex-data-store-and-rest-backend/ %} blog post and {% url 'Vue + Vuex + REST Testing' recipes#Blogs %} recipe.

## {% fa fa-angle-right %} How do I spy on console.log?

To spy on ```console.log``` you should use {% url "`cy.stub()`" stub %}.

```javascript
cy.visit('/', {
  onBeforeLoad(win) {
    cy.stub(win.console, 'log').as('consoleLog')
  }
})

//...
cy.get('@consoleLog').should('be.calledWith', 'Hello World!')
```

Also, check out our {% url 'Stubbing `console` Receipe' recipes#Stubbing-and-spying %}.

## {% fa fa-angle-right %} How do I use special characters with `cy.get()`?

Special characters like `/`, `.` are valid characters for ids {% url "according to the CSS spec" https://www.w3.org/TR/html50/dom.html#the-id-attribute %}.

To test elements with those characters in ids, they need to be escaped with {% url "`CSS.escape`" https://developer.mozilla.org/en-US/docs/Web/API/CSS/escape %} or {% url "`Cypress.$.escapeSelector`" https://api.jquery.com/jQuery.escapeSelector/ %}.

```html
<!doctype html>
<html lang="en">
<body>
  <div id="Configuration/Setup/TextField.id">Hello World</div>
</body>
</html>
```

```js
it('test', () => {
  cy.visit('index.html')
  cy.get(`#${CSS.escape('Configuration/Setup/TextField.id')}`)
    .contains('Hello World')

  cy.get(`#${Cypress.$.escapeSelector('Configuration/Setup/TextField.id')}`)
    .contains('Hello World')
})
```

Note that `cy.$$.escapeSelector()` doesn't work. `cy.$$` doesn't refer to `jQuery`. It only queries DOM. {% url "Learn more about why" $#Notes %}

## {% fa fa-angle-right %} Can I use Cypress to test charts and graphs?

Yes. You can leverage visual testing tools to test that charts and graphs are rendering as expected. For more information, check out the {% url "Visual Testing guide" visual-testing %} and the following blog posts.

- see {% url "Testing a chart with Cypress and Applitools" https://glebbahmutov.com/blog/testing-a-chart/ %}
- see {% url "Testing how an application renders a drawing with Cypress and Percy.io" https://glebbahmutov.com/blog/testing-visually/ %}

## {% fa fa-angle-right %} Why doesn't the `instanceof Event` work?

It might be because of the 2 different windows in Cypress Test Runner. For more information, please check {% url "the note here" window#Cypress-uses-2-different-windows %}.

## {% fa fa-angle-right %} Can I use Cucumber to write tests?

Yes, you can. You can write feature files containing Cucumber scenarios and then use Cypress to write your step definitions in your spec files. A special preprocessor then converts the scenarios and step definitions into "regular" JavaScript Cypress tests.

- try using the {% url "Cucumber preprocessor" https://github.com/TheBrainFamily/cypress-cucumber-preprocessor %} and search our {% url "Plugins" plugins %} page for additional helper plugins
- read {% url "Cypress Super-patterns: How to elevate the quality of your test suite" https://dev.to/wescopeland/cypress-super-patterns-how-to-elevate-the-quality-of-your-test-suite-1lcf %} for best practices when writing Cucumber tests
- take a look at {% url "briebug/bba-cypress-quickstart" https://github.com/briebug/bba-cypress-quickstart %} example application

## {% fa fa-angle-right %} Can I test Next.js sites using Cypress?

Yes, absolutely. See an example in the {% url next-and-cypress-example https://github.com/bahmutov/next-and-cypress-example %} repository where we show how to instrument the application's source code to get {% url "code coverage" code-coverage %} from tests. You can learn how to set good Cypress tests for a Next.js application in this {% url tutorial https://getstarted.sh/bulletproof-next/e2e-testing-with-cypress %}.

## {% fa fa-angle-right %} Can I test Gatsby.js sites using Cypress?

Yes, as you can read in the official {% url "Gatsby docs" https://www.gatsbyjs.com/docs/end-to-end-testing/ %}. You can also watch the "Cypress + Gatsby webinar" {% url recording https://www.youtube.com/watch?v=Tx6Lg9mwcCE %} and browse the webinar's {% url slides https://cypress.slides.com/amirrustam/cypress-gatsby-confidently-fast-web-development %}.

## {% fa fa-angle-right %} Can I test React applications using Cypress?

Yes, absolutely. A good example of a fully tested React application is our {% url "Cypress RealWorld App" https://github.com/cypress-io/cypress-example-realworld %} and {% url "TodoMVC Redux App" https://github.com/cypress-io/cypress-example-todomvc-redux %}. You can even use React DevTools while testing your application, read {% url "The easiest way to connect Cypress and React DevTools" https://dev.to/dmtrkovalenko/the-easiest-way-to-connect-cypress-and-react-devtools-5hgm %}. If you really need to select React components by their name, props, or state, check out {% url cypress-react-selector https://github.com/abhinaba-ghosh/cypress-react-selector %}.

Finally, you might want to check out the {% url "React Component Testing" component-testing-introduction %} adaptor that allows you to test your React components right inside Cypress.

## {% fa fa-angle-right %} Can I check the GraphQL network calls using Cypress?

Yes, by using the newer API command {% url "cy.intercept()" intercept %} as described in the {% url "Smart GraphQL Stubbing in Cypress" https://glebbahmutov.com/blog/smart-graphql-stubbing/ %} post or by utilizing the {% url "cypress-graphql-mock" https://github.com/tgriesser/cypress-graphql-mock %} plugin.

## {% fa fa-angle-right %} Can Cypress be used for model-based testing?

Yes, for example see {% url "this webinar" https://www.youtube.com/watch?v=U30BKedA2CY %} hosted by Curiosity Software. In addition, since our {% url "Real World App (RWA)" https://github.com/cypress-io/cypress-realworld-app %} is implemented using XState model state library, we are looking for ways to make model-based testing simpler and more powerful. Read {% url "Access XState from Cypress Test" https://glebbahmutov.com/blog/cypress-and-xstate/ %} for our start.

## {% fa fa-angle-right %} Can Cypress be used for performance testing?

Cypress is not built for performance testing. Because Cypress instruments the page under test, proxies the network requests, and tightly controls the test steps, the Test Runner adds its own overhead. Thus, the performance numbers you get from Cypress tests are slower than "normal" use. Still, you can access the native `window.performance` object and grab the page time measurements, see the [Evaluate performance metrics](https://github.com/cypress-io/cypress-example-recipes#testing-the-dom) recipe. You can also [run Lighthouse audit straight from Cypress](https://www.mariedrake.com/post/web-performance-testing-with-google-lighthouse) via [cypress-audit](https://www.npmjs.com/package/cypress-audit) community plugin.

## {% fa fa-angle-right %} Can Cypress test WASM code?

Yes, read the blog post {% url "Cypress WASM Example" https://glebbahmutov.com/blog/cypress-wasm-example/ %}. We welcome more user feedback to make WASM testing simpler.

## {% fa fa-angle-right %} Can I use Cypress to document my application?

End-to-end tests are an excellent way to keep your application's documentation accurate and up-to-date. Read {% url "Cypress Book" https://glebbahmutov.com/blog/cypress-book/ %} blog post, and take a look at {% url "cypress-movie" https://github.com/bahmutov/cypress-movie %} project.

## {% fa fa-angle-right %} Can I use Jest snapshots?

While there is no built-in `snapshot` command in Cypress, you can make your own snapshot assertion command. Read how to do so in our blog post {% url "End-to-End Snapshot Testing" https://www.cypress.io/blog/2018/01/16/end-to-end-snapshot-testing/ %}. We recommend using the 3rd-party module {% url "cypress-plugin-snapshots" https://github.com/meinaart/cypress-plugin-snapshots %}. For other snapshot plugins, search the {% url Plugins %} page.

## {% fa fa-angle-right %} Can I use Testing Library?

Absolutely! Feel free to add the {% url '@testing-library/cypress' https://testing-library.com/docs/cypress-testing-library/intro/ %} to your setup and use its methods like `findByRole`, `findByLabelText`, `findByText`, and others to find the DOM elements.

The following example comes from the Testing Library's documentation

```js
cy.findByRole('button', { name: /Jackie Chan/i }).click()
cy.findByRole('button', { name: /Button Text/i }).should('exist')
cy.findByRole('button', { name: /Non-existing Button Text/i }).should(
  'not.exist'
)

cy.findByLabelText(/Label text/i, { timeout: 7000 }).should('exist')

// findAllByText _inside_ a form element
cy.get('form')
  .findByText('button', { name: /Button Text/i })
  .should('exist')

cy.findByRole('dialog').within(() => {
  cy.findByRole('button', { name: /confirm/i })
})
```

We have had a webinar with {% url 'Roman Sandler' https://twitter.com/RomanSndlr %} where he has given practical advice on writing effective tests using the Testing Library. You can find the recording and the slides {% url here https://www.cypress.io/blog/2020/07/15/webcast-recording-build-invincible-integration-tests-using-cypress-and-cypress-testing-library/ %}.
