# Testing Guidelines

## Use Jest's mock functions instead of Sinon

Jest incorporates most of the features of Sinon with a slimmer API:

* `jest.fn()` can be used in place of `sinon.stub()`.
* `jest.spyOn(object, method) can be used in place of `sinon.spy(object, method)`.
* `jest.spyOn(object, method)` can be used in place of `sinon.stub(object, method)` (with the caveat that the method being spied upon will still be called by default).
* `jest.useFakeTimers()` can be used in place of `sinon.useFakeTimers()` (though note that Jest's "clock" object had fewer features than Sinon's prior to Jest v29.5).

## Don't test private code

If you encounter a function or method whose name starts with `\_` or which has been tagged with the `@private` JSDoc tag or `private` keyword in TypeScript, you do not need to test this code. These markers are used to indicate that the code in question should not be used outside the file. As a result, any tests written for this code would need to have knowledge that no other part of the system would have. This is not to say that private code should not be tested at all, but rather that it should be tested via public code that makes use of the private code.
