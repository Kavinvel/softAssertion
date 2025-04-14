# softAssertion
softAssertion In cypress
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
