# Contributing to Smart Home Application

Thank you for your interest in contributing to the Smart Home Application! This document provides guidelines and instructions for contributing to the project.

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Getting Started](#getting-started)
3. [Development Setup](#development-setup)
4. [Project Structure](#project-structure)
5. [Coding Standards](#coding-standards)
6. [Making Changes](#making-changes)
7. [Testing Guidelines](#testing-guidelines)
8. [Submitting Changes](#submitting-changes)
9. [Review Process](#review-process)

---

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inclusive environment for all contributors. Please:

- Be respectful and considerate
- Accept constructive criticism gracefully
- Focus on what's best for the community
- Show empathy towards other contributors

### Unacceptable Behavior

- Harassment or discrimination of any kind
- Trolling, insulting comments, or personal attacks
- Publishing others' private information
- Any conduct that could reasonably be considered inappropriate

---

## Getting Started

### Prerequisites

Before contributing, ensure you have:

- Flutter SDK (3.x or higher)
- Dart SDK (3.4.3 or higher)
- Git
- A code editor (VS Code or Android Studio recommended)
- Basic knowledge of Flutter and Dart

### Reading the Documentation

Before making changes:

1. Read the [README.md](README.md) for project overview
2. Study the [ARCHITECTURE.md](ARCHITECTURE.md) for detailed technical documentation
3. Understand the existing code structure
4. Check existing issues and pull requests

---

## Development Setup

### 1. Fork and Clone

```bash
# Fork the repository on GitHub, then clone your fork
git clone https://github.com/YOUR_USERNAME/SmartHome.git
cd SmartHome

# Add upstream remote
git remote add upstream https://github.com/mm01rahman/SmartHome.git
```

### 2. Install Dependencies

```bash
flutter pub get
```

### 3. Run the Application

```bash
flutter run
```

### 4. Verify Setup

```bash
# Run tests
flutter test

# Check code formatting
flutter format --set-exit-if-changed .

# Run static analysis
flutter analyze
```

---

## Project Structure

```
lib/
├── main.dart              # Entry point
├── models/                # Data models
├── screens/               # UI screens
├── services/              # Business logic
├── widgets/               # Reusable components
├── theme/                 # Styling
└── utils/                 # Helper functions
```

### Key Files

- **Services**: Business logic and data management
  - `auth_service.dart`: Authentication
  - `session_manager.dart`: Session persistence
  - `room_storage.dart`: Local data storage
  - `theme_controller.dart`: Theme management

- **Screens**: Full-page UI components
  - Follow `_screen.dart` naming convention
  - Each screen has a StatefulWidget

- **Widgets**: Reusable UI components
  - Small, focused components
  - Highly reusable

---

## Coding Standards

### Dart Style Guide

Follow the [official Dart style guide](https://dart.dev/guides/language/effective-dart/style):

```dart
// Good
class UserProfile {
  final String userName;
  
  UserProfile({required this.userName});
}

// Bad
class userProfile {
  String user_name;
}
```

### Naming Conventions

**Files**:
```
snake_case.dart
dashboard_screen.dart
auth_service.dart
device_tile.dart
```

**Classes**:
```dart
class DashboardScreen { }
class DeviceItem { }
class AuthService { }
```

**Variables and Functions**:
```dart
// camelCase
final String userName;
void loadUserData() { }

// Private with underscore
String _privateField;
void _privateMethod() { }

// Constants
static const String kTokenKey = 'token';
const Color primaryColor = Color(0xFF4DA3FF);
```

**Boolean Variables**:
```dart
bool isLoading = true;
bool hasError = false;
bool canSubmit = false;
```

### Code Organization

**Class Member Order**:
```dart
class ExampleWidget extends StatefulWidget {
  // 1. Static constants
  static const String title = 'Example';
  
  // 2. Final fields
  final String id;
  
  // 3. Constructor
  const ExampleWidget({required this.id});
  
  // 4. Override methods
  @override
  State<ExampleWidget> createState() => _ExampleWidgetState();
}

class _ExampleWidgetState extends State<ExampleWidget> {
  // 1. Private fields
  late AnimationController _controller;
  bool _isLoading = false;
  
  // 2. Lifecycle methods
  @override
  void initState() {
    super.initState();
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
  
  // 3. Public methods
  void publicMethod() { }
  
  // 4. Build method
  @override
  Widget build(BuildContext context) {
    return Container();
  }
  
  // 5. Private helper methods
  void _privateHelper() { }
  
  // 6. Build methods for widgets
  Widget _buildHeader() {
    return Container();
  }
}
```

### Comments and Documentation

**Doc Comments** for public APIs:
```dart
/// Authenticates a user with email and password.
///
/// Returns a JWT token on success.
/// Throws [AuthException] if authentication fails.
Future<String> login({
  required String email,
  required String password,
}) async {
  // Implementation
}
```

**Inline Comments** for complex logic:
```dart
// Extract token from various possible response formats
final token = response.data['token'] ?? 
              response.data['access_token'] ??
              response.data['data']?['token'];
```

**Section Dividers** for organization:
```dart
// ============================================================
// AUTHENTICATION METHODS
// ============================================================

Future<void> login() { }
Future<void> register() { }
Future<void> logout() { }

// ============================================================
// USER PROFILE METHODS
// ============================================================

Future<User> getProfile() { }
```

### Code Formatting

**Always run before committing**:
```bash
flutter format .
```

**Key formatting rules**:
- 2-space indentation
- Max line length: 80 characters (flexible)
- Trailing commas for better diffs
- Space after control flow keywords

```dart
// Good
if (condition) {
  doSomething();
}

return Container(
  padding: const EdgeInsets.all(16),
  child: Text('Hello'),
);

// Bad
if(condition){
  doSomething();
}

return Container(padding: const EdgeInsets.all(16), child: Text('Hello'));
```

---

## Making Changes

### Branch Naming

Create descriptive branch names:

```bash
# Features
git checkout -b feature/add-voice-control
git checkout -b feature/cloud-sync

# Bug fixes
git checkout -b fix/login-validation
git checkout -b fix/theme-persistence

# Documentation
git checkout -b docs/update-readme
git checkout -b docs/api-documentation

# Refactoring
git checkout -b refactor/auth-service
git checkout -b refactor/widget-structure
```

### Commit Messages

Follow the [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting)
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

**Examples**:
```bash
git commit -m "feat(auth): add biometric authentication"
git commit -m "fix(theme): persist theme selection across restarts"
git commit -m "docs(architecture): add state management section"
git commit -m "refactor(storage): simplify room storage API"
```

### Making Code Changes

1. **Keep changes focused**: One feature/fix per PR
2. **Write clean code**: Follow style guide
3. **Add comments**: Explain complex logic
4. **Update documentation**: If behavior changes
5. **Test thoroughly**: Manual and automated tests

### Adding New Features

When adding a feature:

1. **Plan the implementation**
   - Review architecture
   - Design data flow
   - Consider edge cases

2. **Implement incrementally**
   - Start with data models
   - Add business logic
   - Create UI components
   - Integrate everything

3. **Follow existing patterns**
   - Use similar code structure
   - Match naming conventions
   - Maintain consistency

4. **Document the feature**
   - Add code comments
   - Update ARCHITECTURE.md if needed
   - Include usage examples

---

## Testing Guidelines

### Manual Testing

Before submitting:

1. **Run the app**: `flutter run`
2. **Test your changes**: Verify functionality
3. **Test edge cases**: Empty states, errors, etc.
4. **Test on both themes**: Light and dark mode
5. **Check different screen sizes**: Various devices

### Automated Testing

**Writing Tests**:

```dart
// Unit test example
void main() {
  group('AuthService', () {
    test('login returns token on success', () async {
      final service = AuthService.instance;
      final token = await service.login(
        email: 'test@example.com',
        password: 'password123',
      );
      expect(token, isNotEmpty);
    });
  });
}
```

**Widget test example**:
```dart
void main() {
  testWidgets('DeviceTile shows device name', (tester) async {
    await tester.pumpWidget(
      MaterialApp(
        home: Scaffold(
          body: DeviceTile(
            title: 'Living Room Light',
            subtitle: 'Brightness: 75%',
            icon: Icons.lightbulb,
            color: Colors.blue,
            isOn: true,
            onToggle: (_) {},
          ),
        ),
      ),
    );
    
    expect(find.text('Living Room Light'), findsOneWidget);
  });
}
```

**Run tests**:
```bash
flutter test
```

---

## Submitting Changes

### Pre-submission Checklist

- [ ] Code follows style guide
- [ ] Code is properly formatted (`flutter format .`)
- [ ] No analysis errors (`flutter analyze`)
- [ ] All tests pass (`flutter test`)
- [ ] Manual testing completed
- [ ] Documentation updated
- [ ] Commit messages are clear
- [ ] Branch is up to date with main

### Creating a Pull Request

1. **Push your branch**:
   ```bash
   git push origin feature/your-feature-name
   ```

2. **Create PR on GitHub**:
   - Go to the repository
   - Click "Pull requests" → "New pull request"
   - Select your branch
   - Fill in the template

3. **PR Title**:
   ```
   feat: Add voice control for devices
   fix: Resolve theme persistence issue
   docs: Update architecture documentation
   ```

4. **PR Description**:
   ```markdown
   ## Description
   Brief description of changes
   
   ## Changes Made
   - Added voice control feature
   - Implemented speech recognition
   - Updated device control screen
   
   ## Testing
   - Tested on Android and iOS
   - Verified in both light and dark themes
   - Tested with different voice commands
   
   ## Screenshots
   (if applicable)
   
   ## Related Issues
   Closes #123
   ```

### Pull Request Template

```markdown
## Description
<!-- Describe your changes -->

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex code
- [ ] Documentation updated
- [ ] No new warnings
- [ ] Tests added/updated
- [ ] All tests pass
- [ ] Manual testing completed

## Testing
<!-- Describe testing performed -->

## Screenshots
<!-- Add screenshots if applicable -->
```

---

## Review Process

### What Reviewers Look For

1. **Code Quality**
   - Follows style guide
   - Clean and readable
   - Well-organized
   - Properly commented

2. **Functionality**
   - Works as intended
   - No bugs introduced
   - Edge cases handled
   - Good error handling

3. **Testing**
   - Adequate test coverage
   - Tests pass
   - Manual testing done

4. **Documentation**
   - Code is documented
   - README/ARCHITECTURE updated if needed
   - Clear commit messages

### Addressing Review Comments

1. **Respond promptly**: Address feedback quickly
2. **Be open to feedback**: Accept suggestions gracefully
3. **Ask questions**: If something is unclear
4. **Make requested changes**: Update your PR
5. **Mark resolved**: Check off resolved comments

### After Approval

Once approved and merged:

1. **Delete your branch**: Clean up
   ```bash
   git branch -d feature/your-feature
   git push origin --delete feature/your-feature
   ```

2. **Update your fork**:
   ```bash
   git checkout main
   git pull upstream main
   git push origin main
   ```

---

## Common Contribution Areas

### Good First Issues

Look for issues labeled:
- `good first issue`
- `help wanted`
- `documentation`

### Areas Needing Help

1. **Features**
   - Cloud sync
   - Automation/scenes
   - Voice control
   - Analytics

2. **Improvements**
   - Performance optimization
   - Better error handling
   - UI/UX enhancements
   - Accessibility

3. **Documentation**
   - Code examples
   - Tutorial videos
   - API documentation
   - Translations

4. **Testing**
   - Unit tests
   - Widget tests
   - Integration tests
   - E2E tests

---

## Getting Help

### Resources

- **Documentation**: [ARCHITECTURE.md](ARCHITECTURE.md)
- **Flutter Docs**: https://docs.flutter.dev
- **Dart Docs**: https://dart.dev/guides

### Contact

- **Issues**: Open an issue on GitHub
- **Discussions**: Use GitHub Discussions
- **Questions**: Tag maintainers in PR comments

---

## Recognition

Contributors are recognized in:
- Commit history
- Release notes
- Contributors list (future)

Thank you for contributing to the Smart Home Application! 🎉

---

**Last Updated**: 2025
**Maintainer**: mm01rahman
