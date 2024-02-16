# React Guidelines

## Purpose

This document provides guidelines for contributing new components or updating existing components in our React codebases.

## Guidelines for React Components

The following guidelines are not strictly required, but are considered best practices that contribute to higher quality components. Not all guidelines will be applicable depending on the specific React repository's support. Please refer to the respective repository for additional guidelines and further details.

If certain requirements cannot be met due to time constraints, legacy code, or other factors, contributors should use their best judgement on which requirements are most important or feasible to implement.

### Component Structure

- Follow consistent file structure
- Use functional components with hooks instead of class components
- Separate container and presentational components when appropriate
- Avoid large, complex components - break into reusable pieces

### Component User Interface Features (If applicable)

- Accessibility (A11y)
- Dark-mode
- Dynamic text length
- Internationalization (i18n) translations including right-to-left languages
- Responsive UI/UX (support mobile, fullscreen, and pop-up views)
- Use of [design tokens](https://github.com/MetaMask/design-tokens) for color and typography to ensure brand consistency and uphold accessibility standards

### Code Quality

- Prefer TypeScript for static typing
- Follow linting rules and formatting guidelines
- Name props and state values descriptively
- Extract reusable logic into custom hooks
- Avoid unnecessary rerenders
  - Examples:
    - Blog: [5 Ways to Optimize Your Functional React Components
      ](https://javascript.plainenglish.io/5-ways-to-optimize-your-functional-react-components-cb3cf6c7bd68)
    - React Doc: https://react.dev/learn/render-and-commit#optimizing-performance

### Testing

- Strive for full test coverage
- Write unit tests
- Write e2e tests (If applicable)

### Documentation

- Support Storybook pages for interactive component docs (If applicable)
