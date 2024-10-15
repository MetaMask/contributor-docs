# Mocking Endpoints with E2E Testing

## Mocking

Mocking is an important technique we use in our testing strategy to simulate various scenarios and conditions that our application may encounter. By creating mock responses for our API endpoints, we can effectively test how the app behaves under specific circumstances, such as network failures or receiving unexpected data.

We believe that mocking should complement our end-to-end (E2E) testing rather than replace it. While E2E tests provide vital confidence in the overall functionality of the MetaMask app, mocking allows us to conduct more focused component tests. This helps us identify issues early in the development process and ensures our app can handle different states gracefully.

That said, we still believe E2E tests should be carried out wherever possible to ensure that the application behaves as intended for its end users. 

As we identify and add more mocked tests, the implementation of our mocking strategy will evolve. This flexibility ensures that our testing remains relevant and effective as the application grows.

## File Structure

To maintain a clear and organised structure, we have separated our E2E tests from our mocked tests. This distinction helps us manage our testing efforts more effectively.

```plaintext
root/
├── e2e/
│   ├── spec/
│   │   ├── Accounts/
│   │   │   └── spec.js
│   │   └── Transactions/
│   │       └── spec.js
└── mocks/
    ├── Accounts/
    │   └── mockSpec.js
    └── Transactions/
        └── mockSpec.js
This structure allows us to keep our E2E tests focused on overall app functionality while utilising mocks to simulate various conditions.

By adopting this approach, we ensure our tests remain robust and relevant as we continue to enhance the MetaMask application.