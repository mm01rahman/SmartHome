# Quick Reference Guide

A quick reference for developers working on the Smart Home application.

## Essential Commands

### Setup & Installation
```bash
# Clone and setup
git clone https://github.com/mm01rahman/SmartHome.git
cd SmartHome
flutter pub get

# Run the app
flutter run

# Run on specific device
flutter run -d <device_id>

# List available devices
flutter devices
```

### Development
```bash
# Hot reload
Press 'r' in terminal (while app running)

# Hot restart
Press 'R' in terminal

# Format code
flutter format .

# Analyze code
flutter analyze

# Run tests
flutter test

# Run specific test
flutter test test/unit/auth_service_test.dart

# Clean build
flutter clean
flutter pub get
```

### Building
```bash
# Android APK
flutter build apk --release

# Android App Bundle
flutter build appbundle --release

# iOS
flutter build ios --release

# With custom API
flutter build apk --dart-define=API_BASE_URL=https://api.example.com
```

---

## Project Structure Quick Reference

```
lib/
├── main.dart                       # Entry point, MaterialApp setup
│
├── models/                         # Data models
│   ├── device_item.dart           # Device data structure
│   └── scene_item.dart            # Scene data structure
│
├── screens/                        # Full screens
│   ├── login_screen.dart          # User authentication
│   ├── signup_screen.dart         # New user registration
│   ├── dashboard_screen.dart      # Main home screen
│   ├── rooms_screen.dart          # Room management
│   ├── room_devices_screen.dart   # Devices in a room
│   ├── all_devices_screen.dart    # All devices list
│   ├── device_control_screen.dart # Device controls
│   ├── energy_screen.dart         # Energy monitoring
│   ├── settings_screen.dart       # App settings
│   ├── profile_screen.dart        # User profile
│   └── edit_profile_screen.dart   # Edit user info
│
├── services/                       # Business logic
│   ├── auth_service.dart          # Authentication API
│   ├── session_manager.dart       # Session persistence
│   ├── room_storage.dart          # Room/device storage
│   └── theme_controller.dart      # Theme management
│
├── widgets/                        # Reusable components
│   ├── app_bottom_nav.dart        # Bottom navigation
│   ├── device_tile.dart           # Device list item
│   ├── device_card.dart           # Device grid card
│   ├── stat_card.dart             # Statistics card
│   ├── status_chip.dart           # Status badge
│   ├── status_badge.dart          # Status indicator
│   ├── scene_card.dart            # Scene card
│   ├── info_card.dart             # Info display
│   ├── quick_action_tile.dart     # Quick action button
│   ├── primary_button.dart        # Main button
│   └── app_text_field.dart        # Text input
│
├── theme/                          # Styling
│   ├── app_colors.dart            # Color definitions
│   └── app_text_styles.dart       # Text styles
│
└── utils/                          # Helpers
    ├── form_validators.dart       # Input validation
    └── id_gen.dart                # ID generation
```

---

## Key Classes & Usage

### AuthService
```dart
// Login
final token = await AuthService.instance.login(
  email: 'user@example.com',
  password: 'password123',
);

// Register
final token = await AuthService.instance.register(
  name: 'John Doe',
  email: 'user@example.com',
  password: 'password123',
);

// Get current user
final user = await AuthService.instance.fetchCurrentUser();

// Logout
await AuthService.instance.logout();
```

### SessionManager
```dart
// Save token
await SessionManager.instance.saveToken(token);

// Get token
final token = await SessionManager.instance.getToken();

// Save username
await SessionManager.instance.saveUserName('John');

// Get username
final name = await SessionManager.instance.getUserName();

// Clear all session data
await SessionManager.instance.clearSession();
```

### RoomStorage
```dart
// Load rooms
final rooms = await RoomStorage.loadRooms();

// Save rooms
await RoomStorage.saveRooms(rooms);

// Create a room
final room = RoomStorage.makeRoom(
  id: IdGen.room(),
  name: 'Living Room',
  icon: Icons.living_rounded,
  isOnline: true,
  devices: [],
);

// Add room
await RoomStorage.addRoom(room: room);

// Rename room
await RoomStorage.renameRoom(
  roomIndex: 0,
  newName: 'New Name',
);

// Delete room
await RoomStorage.deleteRoom(roomIndex: 0);

// Create a device
final device = RoomStorage.makeDevice(
  id: IdGen.device(),
  name: 'Main Light',
  type: 'light',
  status: 'online',
  value: 'On',
  icon: Icons.lightbulb,
  isOn: true,
  brightness: 75.0,
);

// Update device
await RoomStorage.updateDeviceInRoom(
  roomIndex: 0,
  deviceId: 'device_123',
  patch: {'isOn': true, 'brightness': 80.0},
);
```

### ThemeController
```dart
// Get current theme
final mode = ThemeController.instance.mode;
final label = ThemeController.instance.modeLabel;

// Set theme
await ThemeController.instance.setTheme('Dark');
await ThemeController.instance.setTheme('Light');
await ThemeController.instance.setTheme('System');

// Load saved theme (call in main())
await ThemeController.instance.load();
```

---

## Common Patterns

### Screen Structure
```dart
class ExampleScreen extends StatefulWidget {
  const ExampleScreen({super.key});

  @override
  State<ExampleScreen> createState() => _ExampleScreenState();
}

class _ExampleScreenState extends State<ExampleScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _animationController;
  late Animation<double> _fadeAnimation;
  
  @override
  void initState() {
    super.initState();
    _setupAnimations();
    _loadData();
  }
  
  @override
  void dispose() {
    _animationController.dispose();
    super.dispose();
  }
  
  Future<void> _loadData() async {
    // Load data
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: FadeTransition(
        opacity: _fadeAnimation,
        child: _buildContent(),
      ),
    );
  }
  
  Widget _buildContent() {
    return Container();
  }
}
```

### Service Pattern (Singleton)
```dart
class MyService {
  MyService._internal();
  static final MyService instance = MyService._internal();
  
  // Methods
  Future<void> doSomething() async {
    // Implementation
  }
}

// Usage
await MyService.instance.doSomething();
```

### Storage Pattern
```dart
// Save
final prefs = await SharedPreferences.getInstance();
await prefs.setString('key', 'value');
await prefs.setInt('count', 42);
await prefs.setBool('flag', true);

// Load
final value = prefs.getString('key');
final count = prefs.getInt('count') ?? 0;
final flag = prefs.getBool('flag') ?? false;

// Remove
await prefs.remove('key');

// Clear all
await prefs.clear();
```

### Navigation
```dart
// Push
Navigator.push(
  context,
  MaterialPageRoute(builder: (_) => TargetScreen()),
);

// Named route
Navigator.pushNamed(context, '/dashboard');

// Replace
Navigator.pushReplacementNamed(context, '/login');

// Clear and navigate
Navigator.pushNamedAndRemoveUntil(
  context,
  '/dashboard',
  (route) => false,
);

// Pop
Navigator.pop(context);

// Pop with result
Navigator.pop(context, result);
```

### Error Handling
```dart
try {
  await riskyOperation();
} on AuthException catch (e) {
  // Handle auth error
  showSnackbar(e.message);
} on DioException catch (e) {
  // Handle network error
  showSnackbar('Network error');
} catch (e) {
  // Handle other errors
  showSnackbar('An error occurred');
}
```

### Loading States
```dart
class _MyScreenState extends State<MyScreen> {
  bool _isLoading = false;
  
  Future<void> _loadData() async {
    setState(() => _isLoading = true);
    
    try {
      // Load data
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }
  
  @override
  Widget build(BuildContext context) {
    if (_isLoading) {
      return Center(child: CircularProgressIndicator());
    }
    return _buildContent();
  }
}
```

---

## Theme-Aware Colors

```dart
// Using AppColors
Container(
  color: AppColors.bgOf(context),          // Background
  child: Card(
    color: AppColors.cardOf(context),      // Card background
    child: Text(
      'Title',
      style: TextStyle(
        color: AppColors.headingOf(context),  // Heading text
      ),
    ),
  ),
)

// Available colors
AppColors.bgOf(context)           // Screen background
AppColors.cardOf(context)         // Card background
AppColors.headingOf(context)      // Heading text
AppColors.textSecondaryOf(context) // Secondary text
AppColors.borderSoftOf(context)   // Border color
AppColors.shadowOf(context)       // Shadow color
AppColors.primarySoftOf(context)  // Primary soft background
AppColors.onlineBgOf(context)     // Online badge background
AppColors.onlineTextOf(context)   // Online badge text
AppColors.errorTextOf(context)    // Error text

// Static colors (same in all themes)
AppColors.primary                 // Primary blue
AppColors.primarySoft             // Light primary
AppColors.onlineGreen             // Success green
AppColors.warning                 // Warning amber
AppColors.teal                    // Teal accent
AppColors.purple                  // Purple accent
```

---

## Common Widgets

### DeviceTile
```dart
DeviceTile(
  title: 'Living Room Light',
  subtitle: 'Brightness: 75%',
  icon: Icons.lightbulb,
  color: Colors.blue,
  isOn: true,
  onToggle: (value) {
    setState(() => isOn = value);
  },
)
```

### StatCard
```dart
StatCard(
  icon: Icons.home,
  iconColor: Colors.blue,
  iconBgColor: Colors.blue.withOpacity(0.1),
  label: 'Total Rooms',
  value: '6',
)
```

### StatusChip
```dart
StatusChip(
  label: 'ONLINE',
  textColor: Colors.green,
  leadingIcon: Icon(Icons.check_circle, size: 14),
)
```

### PrimaryButton
```dart
PrimaryButton(
  text: 'Login',
  onPressed: _handleLogin,
  isLoading: _isLoading,
)
```

### AppTextField
```dart
AppTextField(
  controller: _emailController,
  label: 'Email',
  hint: 'Enter your email',
  keyboardType: TextInputType.emailAddress,
  validator: FormValidators.email,
)
```

---

## Debugging Tips

### Print Debugging
```dart
print('Debug: value = $value');
debugPrint('Debug info');
```

### Flutter DevTools
```bash
# Open DevTools
flutter pub global activate devtools
flutter pub global run devtools
```

### Common Issues

**Hot reload not working**:
```bash
# Press 'R' for hot restart instead of 'r'
# Or stop and restart the app
```

**Build errors**:
```bash
flutter clean
flutter pub get
flutter run
```

**Package conflicts**:
```bash
flutter pub upgrade
flutter pub get
```

**Gradle issues** (Android):
```bash
cd android
./gradlew clean
cd ..
flutter clean
flutter run
```

---

## Testing

### Widget Test Template
```dart
void main() {
  testWidgets('Description', (tester) async {
    // Build widget
    await tester.pumpWidget(
      MaterialApp(home: MyWidget()),
    );
    
    // Find elements
    expect(find.text('Hello'), findsOneWidget);
    
    // Interact
    await tester.tap(find.byType(ElevatedButton));
    await tester.pump();
    
    // Verify
    expect(find.text('Result'), findsOneWidget);
  });
}
```

### Unit Test Template
```dart
void main() {
  group('ClassName', () {
    test('should do something', () {
      // Arrange
      final instance = MyClass();
      
      // Act
      final result = instance.method();
      
      // Assert
      expect(result, equals(expectedValue));
    });
  });
}
```

---

## Git Workflow

```bash
# Create feature branch
git checkout -b feature/my-feature

# Make changes and commit
git add .
git commit -m "feat: add new feature"

# Keep branch updated
git fetch upstream
git rebase upstream/main

# Push changes
git push origin feature/my-feature

# After PR merged
git checkout main
git pull upstream main
git branch -d feature/my-feature
```

---

## Environment Variables

### Setting API URL
```bash
# Run with custom API
flutter run --dart-define=API_BASE_URL=https://api.example.com

# Build with custom API
flutter build apk --dart-define=API_BASE_URL=https://api.example.com
```

### In Code
```dart
const apiUrl = String.fromEnvironment(
  'API_BASE_URL',
  defaultValue: 'https://default-api.com',
);
```

---

## Useful Links

- **Project Docs**: [ARCHITECTURE.md](ARCHITECTURE.md)
- **Contributing**: [CONTRIBUTING.md](CONTRIBUTING.md)
- **Flutter Docs**: https://docs.flutter.dev
- **Dart Docs**: https://dart.dev
- **Material Design**: https://m3.material.io
- **Pub.dev**: https://pub.dev

---

**Quick Tip**: Keep this file bookmarked for quick reference during development!
