# softAssertion
softAssertion In cypress
step 1) Copy below code and paste it in command.js
step 2)Then place it in e2e.js under support folder
step 3) Call this command whereever required in the workflow
*******************************************************************************************************************************************************************************************************************
let itBlockErrors = {};
let totalFailedAssertionsByDescribe = {};

Cypress.Commands.add('softAssert', (actualValue, expectedValue, message) => {
  return cy.wrap(null, { timeout: Cypress.config('defaultCommandTimeout') }).then(() => {
    try {
      expect(actualValue).to.equal(expectedValue, message);
    } catch (err) {
      const itBlockTitle = Cypress.currentTest.title;
      const describeBlockTitle = Cypress.currentTest.titlePath[0];

      totalFailedAssertionsByDescribe[describeBlockTitle] = totalFailedAssertionsByDescribe[describeBlockTitle] || 0;
      totalFailedAssertionsByDescribe[describeBlockTitle]++;

      if (!itBlockErrors[itBlockTitle]) {
        itBlockErrors[itBlockTitle] = [];
      }

      itBlockErrors[itBlockTitle].push({ message, error: err });
    }
  });
});

Cypress.Commands.add('softShould', { prevSubject: 'element' }, (subject, assertion, ...args) => {
  try {
    const itBlockTitle = Cypress.currentTest.title;
    const describeBlockTitle = Cypress.currentTest.titlePath[0];

    // Split the assertion string like "not.be.visible" => ['not', 'be', 'visible']
    const parts = assertion.split('.');

    let expectChain = expect(subject);
    // Apply each part dynamically
    parts.forEach(part => {
      expectChain = expectChain[part];
    });
    // Finally call the matcher with arguments
    expectChain(...args);
  } catch (err) {
    const itBlockTitle = Cypress.currentTest.title;
    const describeBlockTitle = Cypress.currentTest.titlePath[0];
    totalFailedAssertionsByDescribe[describeBlockTitle] = totalFailedAssertionsByDescribe[describeBlockTitle] || 0;
    totalFailedAssertionsByDescribe[describeBlockTitle]++;
    if (!itBlockErrors[itBlockTitle]) {
      itBlockErrors[itBlockTitle] = [];
    }
    const message = `Expected element to satisfy: "${assertion}" with args: ${args.join(', ')}`;
    itBlockErrors[itBlockTitle].push({ message, error: err });
  }
  return cy.wrap(subject); // to allow continued chaining
});

Cypress.Commands.add('assertAll', () => {
  const errors = itBlockErrors;
  itBlockErrors = {};
  if (Object.keys(errors).length > 0) {
    const errorMessages = Object.entries(errors).map(([title, entries], index) => {
      const errorText = entries.map(({ error }) => `=> ${error.message}`).join('\n\n');
      return `${index + 1}. Test Title: ${title}\n${errorText}`;
    });

    const summary = Object.entries(totalFailedAssertionsByDescribe).map(([describe, count]) => {
      return `Total assertion failures in "${describe}": ${count}`;
    }).join('\n');

    throw new Error(`Soft assertion failed: Total it block failed (${Object.keys(errors).length})\n\n${errorMessages.join('\n')}\n\n${summary}`);
  }
});
*******************************************************************************************************************************************************************************************************************
What is Soft Assertion in Cypress?
Soft Assertion allows your Cypress test to continue executing even if some assertions fail.
Unlike hard assertions (default Cypress behavior), which stop the test execution on the first failure, soft assertions collect all the assertion errors and report them at the end of the test.

This is useful when you want to verify multiple things in a test without stopping early.

🔧 How Soft Assertion Works (Commands Used):
cy.softAssert(actual, expected, message)
➤ Compares two values just like expect(actual).to.equal(expected)
➤ If it fails, the error is stored instead of stopping the test.

cy.get(selector).softShould('be.visible')
➤ Works like .should('be.visible')
➤ But continues the test even if the condition fails.

cy.assertAll()
➤ At the end of the test, it checks if any softAssert or softShould failed
➤ If there are failures, it throws a summary error showing all soft assertion failures.

Example Usage:
js
Copy
Edit
cy.softAssert(2 + 2, 5, 'Math check failed');
cy.get('#login-button').softShould('be.visible');
cy.get('#username').softShould('have.value', 'kavinvel');

// Finally call this at the end of your test
cy.assertAll();
Benefits of Soft Assertion:
See all failures at once

Debug tests faster

Ideal for UI validations where multiple checks are required

Keeps tests less flaky due to early terminatio
