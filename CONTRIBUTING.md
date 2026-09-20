# Contributing to JSQL-neo

Thank you for your interest in contributing to JSQL-neo! This document provides guidelines and instructions for contributing.

## 🏗️ Project Overview

JSQL-neo is a JavaScript SQL parser and executor library. Contributions are welcome from developers of all skill levels.

## 🚀 Getting Started

### Prerequisites

- Node.js >= 16.x
- npm >= 8.x

### Setup

```bash
git clone https://github.com/vexify-build/JSQL-neo.git
cd JSQL-neo
npm install
npm test
```

## 📐 Code Standards

### Style Guide

- **Formatting:** Use 2 spaces for indentation (no tabs)
- **Semicolons:** Always end statements with semicolons
- **Quotes:** Use single quotes for strings
- **Line length:** Keep lines under 120 characters

### Best Practices

- Write self-documenting code with clear variable/function names
- Add JSDoc comments for all public APIs
- Keep functions small and focused (single responsibility)
- Avoid deeply nested callbacks — prefer async/await

## 📝 Commit Message Convention

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

Types: feat | fix | docs | style | refactor | test | chore
```

Examples:
```
feat(parser): add support for HAVING clause
fix(executor): handle NULL in JOIN conditions
docs(readme): update installation instructions
```

## 🧪 Testing

- All new features must include tests
- All tests must pass before merging: `npm test`
- Aim for meaningful test coverage, not just quantity

```bash
# Run tests
npm test

# Run with coverage
npm run test:coverage
```

## 🔧 Pull Request Process

1. Fork the repository and create a feature branch:
   ```bash
   git checkout -b feat/your-feature-name
   ```

2. Make your changes and commit using conventional commits

3. Push to your fork and open a Pull Request:
   ```bash
   git push origin feat/your-feature-name
   ```

4. Ensure all CI checks pass

5. Link related issues in your PR description

## 🐛 Reporting Issues

When reporting bugs, please include:
- Node.js and npm versions
- Minimal reproducible example
- Expected vs actual behavior
- Error messages and stack traces

## 📄 License

By contributing, you agree that your contributions will be licensed under the MIT License.
