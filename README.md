# Cypress cheat sheet

When writing cypress tests, follow this flow:

1. _User interaction_ (click, type in a field, etc)
2. _Assertions_ (elements exist, url has changed, text matches some pattern)

This flow will repeat over and over in your tests. One or more assertions should follow each interaction.

## Things you should avoid

- Waits that use an arbitrary amount of time
- Testing display states of components that are part of UI libraries like material
- Using brittle selectors such as `cy.get('button:contains("Submit")')`

## Things you should do

- Test the UI, not the backend by stubbing responses with fixtures
- Alias selectors for reuse
- Alias network calls for waits, e.g. `cy.wait('@loginCall')`

## Intercepting requests

Reference: [Network requests guide](https://docs.cypress.io/guides/guides/network-requests)

Stub a response for a call:
```js
cy.intercept('POST', '/v1/user/login', {
  statusCode: 200,
  body: {
    success: true,
    token: testToken
  }
}).as('loginCall');
```

Use a fixture for a call:
```js
cy.intercept('GET', '/config', { fixture: 'config.json' });
```

Alias a call:
```js
cy.intercept('GET', '/config').as('configCall');
```

## Selectors

Reference: [Selector best practices](https://docs.cypress.io/guides/references/best-practices#How-It-Works)

Select an element:
```js
cy.get('[data-cy=submit]')
```

Select an element within another element:
```js
cy.get('mat-card').find('ion-icon[name=checkmark-circle]')
```

Select elements in a shadow DOM:
```js
cy.get('pwa-action-sheet').shadow().find('.action-sheet-button:eq(0)')
```

Select the first in a set of elements:
```js
cy.get('app-status-indicator').first()
cy.get('app-status-indicator:eq(0)')
```

Select a specific index in a set of elements:
```js
cy.get('app-status-indicator:eq(2)')
```

Select an element within a specific DOM structure:
```js
cy.get('#main-list mat-tree-node:last-child button.context-menu-btn')
```

Select an element containing specific text:
```js
cy.get('button.mat-menu-item:contains("Logout")')
```

## Interactions

Click an element:
```js
cy.get('[data-cy=submit]').click()
```

Click a hidden element:
```js
cy.get('[data-cy=submit]').click({ force: true })
```

Click multiple elements:
```js
cy.get('.close-button').click({ multiple: true })
```

Clear, type and blur a field
```js
cy.get('input[name=email]').clear().type('foo').blur()
```

Type in a field with options, then press enter
```js
cy.get('app-search-ahead-chips').find('input').type('foo{enter}', { timeout: 2000, delay: 40 });
```

## Assertions

Reference: [Cypress supported assertions](https://docs.cypress.io/guides/references/assertions

Assert an element exists:
```js
cy.get('[data-cy=submitted]').should('exist');
```

Assert an element does not exist:
```js
cy.get('[data-cy=submitted]').should('not.exist');
```

Assert a certain number of elements exist:
```js
cy.get('app-store-item').its('length').should('be.gte', 3); // greater than or equal
cy.get('app-store-item').its('length').should('be.gt', 0); // greater than
cy.get('app-store-item').its('length').should('equal', 3);
```

Assert an element has a class:
```js
cy.get('app-store-item').should('have.class', 'error');
```

Assert an element has a specific CSS style:
```js
cy.get('mat-drawer app-menu-item').should('have.css', 'visibility', 'visible');
```

Assert a url matches a regex:
```js
cy.url().should('match', /orders\/[\d]{4}$/);
```

Assert an element's text matches a regex:
```js
cy.get('p.page-number').should((elem) => {
  expect(elem.text()).to.match(/1 of 2/); 
});
```

Assert an input has a specific value:
```js
cy.get('@eInput').should('have.value', startVal);
```

## Commands

Stub a login call with a valid JWT token:
```js
import * as jwt from 'jsonwebtoken';

export function mockLogin() {
  
  const issued = Date.now() / 1000;
  const expires = issued + (60 * 60); // hour later

  const tokenPayload = {
    id: '60000',
    unique_name: 'Test Account',
    email: 'test@example.com',
    nbf: issued,
    exp: expires,
    iat: issued,
    iss: 'example.com',
    aud: 'https://api.example.com'
  }
  const token = jwt.sign(tokenPayload, 'secret');

  cy.intercept('POST', 'v1/users/login', {
    statusCode: 200,
    body: {
      success: true,
      token
    }
  });

};
Cypress.Commands.add('mockLogin', mockLogin);
```

Upload a file:
```js
import 'cypress-file-upload';

export function uploadImage(card) {
  
  cy.wrap(card).find('[data-cy=empty-slot]').click();
  cy.get('pwa-action-sheet').shadow().find('.action-sheet-button:eq(0)').click();
  // attach actual image file from fixtures folder
  cy.get('#_capacitor-camera-input').attachFile({ filePath: 'test-image.jpg'});

  cy.wrap(card).find('app-uploaded-image').find('ion-icon[name=checkmark-circle]').should('exist');
};
Cypress.Commands.add('uploadImage', uploadImage);
```

Declare namespace to avoid typescript errors:
```ts
declare global {
  namespace Cypress {
    interface Chainable<Subject = any> {
      mockLogin(): Chainable<typeof mockLogin>;
      uploadImage(param: any): Chainable<typeof uploadImage>;
    }
  }
}
```
