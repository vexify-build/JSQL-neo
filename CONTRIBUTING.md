# 🤝 Contributing to JSQL-neo

Thank you for your interest in contributing to JSQL-neo! 🎉

## 📋 Table of Contents

- [Code of Conduct](#-code-of-conduct)
- [Getting Started](#-getting-started)
- [Development Setup](#-development-setup)
- [Making Changes](#-making-changes)
- [Pull Request Process](#-pull-request-process)
- [Reporting Bugs](#-reporting-bugs)
- [Suggesting Features](#-suggesting-features)

## 📖 Code of Conduct

Please be respectful and constructive in all interactions. We aim to foster an inclusive and welcoming community.

## 🚀 Getting Started

1. **Fork the repository** on GitHub
2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/YOUR_USERNAME/JSQL-neo.git
   cd JSQL-neo
   ```
3. **Add upstream remote**:
   ```bash
   git remote add upstream https://github.com/vexify-build/JSQL-neo.git
   ```

## 🛠️ Development Setup

```bash
# Install dependencies
npm install

# Run in development mode
npm run dev

# Run tests
npm test

# Build for production
npm run build
```

## 🔄 Making Changes

1. **Create a feature branch** from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   # or
   git checkout -b fix/your-bug-fix
   ```

2. **Write clean, well-documented code** following existing patterns

3. **Add tests** for new features or bug fixes

4. **Run tests** to ensure everything works:
   ```bash
   npm test
   ```

5. **Commit your changes** with clear messages:
   ```bash
   git commit -m "feat: add new feature X"
   git commit -m "fix: resolve issue with Y"
   ```

## 📤 Pull Request Process

1. **Keep your fork updated**:
   ```bash
   git fetch upstream
   git rebase upstream/main
   ```

2. **Push your branch**:
   ```bash
   git push origin feature/your-feature-name
   ```

3. **Open a Pull Request** on GitHub:
   - Fill in the PR template
   - Reference any related issues (`Fixes #123`)
   - Ensure CI passes

4. **Code review** will be conducted; address feedback promptly

## 🐛 Reporting Bugs

Please report bugs via [GitHub Issues](https://github.com/vexify-build/JSQL-neo/issues) with:

- Clear title and description
- Steps to reproduce
- Expected vs actual behavior
- Environment details (OS, Node version, etc.)
- Logs or screenshots if applicable

## 💡 Suggesting Features

We welcome feature suggestions! Open an issue with:

- Clear description of the feature
- Use case / motivation
- Potential implementation ideas (optional)

## 📝 License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Happy coding!** 🚀
