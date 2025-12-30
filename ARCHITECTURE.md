# Smart Home Application - Architecture Documentation

## Table of Contents
1. [Overview](#overview)
2. [Architecture & Design Patterns](#architecture--design-patterns)
3. [Application Flow](#application-flow)
4. [Core Components](#core-components)
5. [Data Management](#data-management)
6. [Authentication System](#authentication-system)
7. [UI/UX Architecture](#uiux-architecture)
8. [State Management](#state-management)
9. [Navigation](#navigation)
10. [Theme System](#theme-system)
11. [Code Structure](#code-structure)

---

## Overview

The Smart Home application is a **Flutter-based mobile application** designed to control and monitor smart home devices. It provides a comprehensive interface for managing rooms, devices, energy consumption, and user settings.

### Technology Stack
- **Framework**: Flutter (Dart 3.4.3+)
- **State Management**: StatefulWidget with ChangeNotifier pattern
- **HTTP Client**: Dio for API communication
- **Local Storage**: SharedPreferences for persistent data
- **UI Framework**: Material Design 3 (Material 3)

### Key Features
- User authentication (login/signup)
- Dashboard with device overview
- Room management with custom icons
- Device control with type-specific interfaces
- Energy monitoring
- Light/Dark theme support
- Session persistence
- Offline device state management

---

## Architecture & Design Patterns

### 1. **Layered Architecture**

The application follows a clean layered architecture:

```
┌─────────────────────────────────────┐
│         Presentation Layer          │
│  (Screens, Widgets, UI Components)  │
├─────────────────────────────────────┤
│          Business Logic             │
│    (Services, Controllers, Utils)   │
├─────────────────────────────────────┤
│           Data Layer                │
│  (Models, Storage, API Clients)     │
└─────────────────────────────────────┘
```

**Presentation Layer**: All UI components including screens and reusable widgets
**Business Logic**: Service classes, session management, authentication logic
**Data Layer**: Data models, storage management, API communication

### 2. **Design Patterns Used**

#### Singleton Pattern
Used extensively for service classes to ensure single instances:
```dart
// Example: AuthService
class AuthService implements AuthClient {
  AuthService._internal();  // Private constructor
  static final AuthService instance = AuthService._internal();
  // ...
}
```

**Why Singleton?**
- Maintains consistent state across the app
- Prevents multiple API client instances
- Efficient resource management
- Centralized access point

#### Repository Pattern
`RoomStorage` class acts as a repository for room and device data:
- Abstracts data source (SharedPreferences)
- Provides clean CRUD operations
- Handles data serialization/deserialization

#### Observer Pattern (ChangeNotifier)
`ThemeController` uses Flutter's `ChangeNotifier` for reactive theme updates:
```dart
class ThemeController extends ChangeNotifier {
  ThemeMode _mode = ThemeMode.light;
  
  Future<void> setTheme(String value) async {
    // Update theme
    notifyListeners(); // Notify all observers
  }
}
```

#### Factory Pattern
`RoomStorage.makeRoom()` and `RoomStorage.makeDevice()` use factory methods to create consistent data structures.

---

## Application Flow

### 1. **Application Startup Sequence**

```
main() 
  ↓
Initialize Flutter Bindings
  ↓
Lock Device Orientation (Portrait Only)
  ↓
Load Theme Settings (ThemeController.load())
  ↓
Configure System UI (Status Bar, Navigation Bar)
  ↓
Run Application (SmartHomeApp)
  ↓
Check Session (_checkExistingSession())
  ↓
Navigate to Dashboard OR Login
```

**Step-by-step breakdown:**

1. **`main()` function** (`lib/main.dart:17-40`):
   - Ensures Flutter bindings are initialized
   - Locks screen orientation to portrait mode
   - Loads saved theme preferences before app starts
   - Configures system UI appearance

2. **Session Check** (`lib/main.dart:58-77`):
   ```dart
   Future<bool> _checkExistingSession() async {
     final token = await SessionManager.instance.getToken();
     if (token == null || token.isEmpty) return false;
     
     // Validate token by fetching user data
     try {
       final user = await AuthService.instance.fetchCurrentUser();
       // Extract username from various possible response formats
       final name = (user['name'] ?? user['data']?['name'] ?? user['user']?['name'] ?? '').toString().trim();
       await SessionManager.instance.saveUserName(name.isEmpty ? 'User' : name);
       return true;
     } catch (_) {
       await SessionManager.instance.clearToken();
       return false;
     }
   }
   ```
   - Retrieves stored authentication token
   - Validates token by making API call
   - Handles multiple API response formats
   - Falls back gracefully on errors

3. **Initial Route Decision**:
   - Valid session → `DashboardScreen`
   - Invalid/No session → `LoginScreen`

### 2. **User Authentication Flow**

```
LoginScreen
  ↓
Enter Credentials
  ↓
AuthService.login()
  ↓
API Call (/login)
  ↓
Extract & Save Token
  ↓
Fetch User Profile (/me)
  ↓
Save User Data
  ↓
Navigate to Dashboard
```

**Authentication Process Details:**

1. **Login Request** (`lib/services/auth_service.dart:70-87`):
   ```dart
   Future<String> login({required String email, required String password}) async {
     final response = await _dio.post(
       '/login',
       data: {'email': email, 'password': password},
     );
     
     final token = _extractToken(response.data);
     await _sessionManager.saveToken(token);
     return token;
   }
   ```

2. **Token Extraction** (`lib/services/auth_service.dart:165-188`):
   - Handles multiple response formats
   - Checks various field names: `token`, `access_token`, `plainTextToken`, `plain_text_token`
   - Supports nested data structures

3. **Session Persistence**:
   - Token stored in SharedPreferences
   - Auto-attached to subsequent API requests via Dio interceptor

### 3. **Dashboard Flow**

```
DashboardScreen (Home)
  ↓
Load User Name from Session
  ↓
Display Greeting (Time-based)
  ↓
Show Status (Online Devices, Active Count)
  ↓
Overview Cards (Rooms, Devices, Energy)
  ↓
Favorite Devices List
```

**Dashboard Components:**

1. **Greeting Section** (`lib/screens/dashboard_screen.dart:107-112`):
   ```dart
   String _getGreeting() {
     final hour = DateTime.now().hour;
     if (hour < 12) return 'Good Morning';
     if (hour < 17) return 'Good Afternoon';
     return 'Good Evening';
   }
   ```
   - Dynamic greeting based on time of day
   - Personalized with user name

2. **Status Chips**:
   - Online indicator (green dot + text)
   - Active device count

3. **Overview Grid** (4 cards):
   - Total Rooms
   - Total Devices
   - Online Devices
   - Energy Today (kWh)

4. **Favorite Devices**:
   - Quick access toggles
   - Shows device state (on/off)
   - Device-specific information

---

## Core Components

### 1. **Services Layer**

#### AuthService (`lib/services/auth_service.dart`)

**Purpose**: Manages all authentication-related operations

**Key Features:**
- Singleton instance pattern
- Dio HTTP client with interceptors
- Automatic token attachment
- 401 error handling (auto-logout)
- Flexible API response parsing

**API Configuration:**
```dart
static const String _baseUrl = String.fromEnvironment(
  'API_BASE_URL',
  defaultValue: 'https://landlordbd-1.onrender.com/api',
);
```

**Request Interceptor** (`lib/services/auth_service.dart:28-40`):
```dart
onRequest: (options, handler) async {
  final token = await _sessionManager.getToken();
  options.headers['Accept'] = 'application/json';
  options.headers['Content-Type'] = 'application/json';
  
  if (token != null && token.isNotEmpty) {
    options.headers['Authorization'] = 'Bearer $token';
  }
  
  handler.next(options);
}
```
- Adds authentication headers to every request
- Sets content type headers
- Retrieves token from session manager

**Error Interceptor** (`lib/services/auth_service.dart:41-48`):
```dart
onError: (error, handler) async {
  if (error.response?.statusCode == 401) {
    await _sessionManager.clearToken();
    authNavigatorKey.currentState
        ?.pushNamedAndRemoveUntil('/login', (route) => false);
  }
  handler.next(error);
}
```
- Handles unauthorized responses
- Clears invalid tokens
- Redirects to login screen

**Error Formatting** (`lib/services/auth_service.dart:143-163`):
- Extracts user-friendly messages
- Handles various error response formats
- Provides fallback messages

#### SessionManager (`lib/services/session_manager.dart`)

**Purpose**: Manages user session data persistence

**Stored Data:**
- `auth_token`: JWT or bearer token
- `user_name`: Displayed in dashboard
- `user_email`: User's email address

**Key Methods:**
```dart
Future<void> saveToken(String token)
Future<String?> getToken()
Future<void> clearToken()
Future<void> saveUserName(String name)
Future<String?> getUserName()
Future<void> clearSession()  // Clears all session data
```

**Implementation:**
- Uses SharedPreferences for persistence
- Survives app restarts
- Simple key-value storage

#### RoomStorage (`lib/services/room_storage.dart`)

**Purpose**: Manages rooms and devices data locally

**Why Local Storage?**
- Provides offline functionality
- Fast data access
- No dependency on backend API
- User-specific data isolation

**Data Structure:**
```dart
Room {
  id: String
  name: String
  iconCodePoint: int
  isOnline: bool
  devices: List<Device>
}

Device {
  id: String
  name: String
  type: String (light, fan, thermostat, air_purifier, socket)
  status: String (online, offline, waiting, error)
  value: String
  iconCodePoint: int
  isOn: bool
  brightness: double   // For lights (0-100)
  speed: int          // For fans (1-5)
  temperature: double // For thermostats (16-30)
  mode: String        // For air purifiers (auto, sleep, turbo)
}
```

**Key Functions:**

1. **Factory Methods:**
   ```dart
   static Map<String, dynamic> makeRoom({...})
   static Map<String, dynamic> makeDevice({...})
   ```
   - Create properly structured data
   - Ensure required fields present
   - Set default values

2. **CRUD Operations:**
   ```dart
   static Future<List<Map<String, dynamic>>> loadRooms()
   static Future<void> saveRooms(List<Map<String, dynamic>> rooms)
   static Future<void> updateRoom({required int roomIndex, required Map<String, dynamic> patch})
   static Future<void> renameRoom({required int roomIndex, required String newName})
   static Future<void> deleteRoom({required int roomIndex})
   static Future<void> addRoom({required Map<String, dynamic> room})
   static Future<void> updateDeviceInRoom({...})
   ```

3. **Key Resolution** (`lib/services/room_storage.dart:11-16`):
   ```dart
   static Future<String?> _resolveKey() async {
     final token = await SessionManager.instance.getToken();
     final username = await SessionManager.instance.getUserName();
     if (token == null || username == null) return null;
     return '$_roomsKeyPrefix$username';
   }
   ```
   - Creates user-specific storage key
   - Isolates data between users
   - Requires valid session

4. **Icon Handling:**
   ```dart
   static int iconCodePoint(IconData icon) => icon.codePoint;
   static IconData iconFromCodePoint(int codePoint) {
     return IconData(codePoint, fontFamily: 'MaterialIcons');
   }
   ```
   - Stores icons as integers (codePoints)
   - JSON-serializable format
   - Reconstructs IconData on load

#### ThemeController (`lib/services/theme_controller.dart`)

**Purpose**: Manages application theme (Light/Dark/System)

**Pattern**: Observer (ChangeNotifier)

```dart
class ThemeController extends ChangeNotifier {
  ThemeMode _mode = ThemeMode.light;
  
  ThemeMode get mode => _mode;
  
  Future<void> load() async {
    final prefs = await SharedPreferences.getInstance();
    final v = (prefs.getString(_kThemeKey) ?? 'light').toLowerCase();
    _mode = _fromString(v);
    notifyListeners();
  }
  
  Future<void> setTheme(String value) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(_kThemeKey, value.toLowerCase());
    _mode = _fromString(value.toLowerCase());
    notifyListeners();  // Triggers UI rebuild
  }
}
```

**Usage in MaterialApp** (`lib/main.dart:113-123`):
```dart
return AnimatedBuilder(
  animation: ThemeController.instance,
  builder: (_, __) {
    return MaterialApp(
      themeMode: ThemeController.instance.mode,
      theme: _lightTheme(),
      darkTheme: _darkTheme(),
      // ...
    );
  },
);
```

**How it works:**
1. `AnimatedBuilder` listens to ThemeController
2. When `notifyListeners()` called, builder rebuilds
3. MaterialApp switches theme based on `themeMode`
4. Entire app re-renders with new theme

### 2. **Models**

#### DeviceItem (`lib/models/device_item.dart`)

**Purpose**: Data model for devices

```dart
class DeviceItem {
  final String id;
  String name;
  final String type;
  DeviceStatus status;
  String value;
  IconData icon;
  bool isOn;
  
  // Type-specific properties
  double brightness;     // lights
  int speed;             // fans
  double temperature;    // thermostat
  String mode;           // air purifier
}

enum DeviceStatus { online, offline, waiting, error }
```

**Helper Methods:**
- `statusFromString()`: Parse status from storage
- `statusToString()`: Serialize status for storage

**Why separate properties?**
- Not all devices need all properties
- Type-safe device control
- Easy to extend with new device types

### 3. **Utilities**

#### FormValidators (`lib/utils/form_validators.dart`)

Provides validation for user inputs (email, password, names, etc.)

#### IdGen (`lib/utils/id_gen.dart`)

Generates unique IDs for rooms and devices:
```dart
class IdGen {
  static String room() => 'room_${DateTime.now().millisecondsSinceEpoch}';
  static String device() => 'device_${DateTime.now().millisecondsSinceEpoch}';
}
```

---

## Data Management

### 1. **Storage Strategy**

**Two-tier storage:**

1. **Remote (API)**:
   - User authentication data
   - User profiles
   - Potentially synced device states (future)

2. **Local (SharedPreferences)**:
   - Auth tokens
   - User session data
   - Rooms configuration
   - Device states
   - Theme preferences

**Why local storage for devices?**
- Offline functionality
- Instant response (no network latency)
- Privacy (user data stays on device)
- Reduced server load
- Simplified MVP architecture

### 2. **Data Serialization**

**Room Storage Format:**
```json
{
  "id": "room_1735558400000",
  "name": "Living Room",
  "iconCodePoint": 58745,
  "isOnline": true,
  "devices": [
    {
      "id": "device_1735558401000",
      "name": "Main Light",
      "type": "light",
      "status": "online",
      "value": "On",
      "iconCodePoint": 58740,
      "isOn": true,
      "brightness": 75.0,
      "speed": 2,
      "temperature": 22.0,
      "mode": "auto"
    }
  ]
}
```

**Serialization Process:**
```dart
// Save
final encoded = rooms.map((r) => jsonEncode(r)).toList();
await prefs.setStringList(key, encoded);

// Load
final list = prefs.getStringList(key);
return list.map((s) => Map<String, dynamic>.from(jsonDecode(s) as Map)).toList();
```

### 3. **State Persistence**

**Device State Updates** (`lib/screens/device_control_screen.dart:73-94`):
```dart
Future<void> _savePatch(Map<String, dynamic> patch) async {
  await RoomStorage.updateDeviceInRoom(
    roomIndex: widget.roomIndex,
    deviceId: widget.deviceId,
    patch: patch,
  );
}

Future<void> _saveAll() async {
  await _savePatch({
    'isOn': _isOn,
    'value': _isOn ? 'On' : 'Off',
    'brightness': _brightness,
    'speed': _speed,
    'temperature': _temperature,
    'mode': _mode,
  });
}
```

**When state is saved:**
- Toggle switch changed
- Slider moved (brightness, temperature)
- Speed adjusted
- Mode changed

**Persistence flow:**
1. User interacts with control
2. Local state updated (`setState`)
3. UI updates immediately (optimistic update)
4. State saved to SharedPreferences
5. Persists across app restarts

---

## Authentication System

### 1. **Authentication Flow Details**

**Login Process:**

```
User enters credentials
         ↓
LoginScreen validates input
         ↓
AuthService.login(email, password)
         ↓
POST /api/login
         ↓
Response: {token: "xxx", user: {...}}
         ↓
Extract token (_extractToken)
         ↓
SessionManager.saveToken(token)
         ↓
AuthService.fetchCurrentUser()
         ↓
GET /api/me (with Bearer token)
         ↓
Response: {name: "John", email: "..."}
         ↓
SessionManager.saveUserName(name)
         ↓
Navigate to Dashboard
```

**Registration Process:**

```
User fills signup form
         ↓
SignupScreen validates input
         ↓
AuthService.register(name, email, password)
         ↓
POST /api/register
         ↓
{name, email, password, password_confirmation}
         ↓
Response: {token: "xxx"}
         ↓
Token saved
         ↓
User profile fetched
         ↓
Navigate to Dashboard
```

### 2. **Token Management**

**Automatic Token Attachment:**
Every API request automatically includes the token via Dio interceptor:

```dart
options.headers['Authorization'] = 'Bearer $token';
```

**Token Validation:**
- Checked on app startup
- Validated by calling `/api/me`
- Invalid tokens cleared automatically

**Token Expiration Handling:**
- 401 responses trigger automatic logout
- User redirected to login screen
- Token cleared from storage

### 3. **Security Considerations**

**Current Implementation:**
- Tokens stored in SharedPreferences (plain text)
- HTTPS communication with API
- Bearer token authentication

**Production Recommendations:**
- Use Flutter Secure Storage for tokens
- Implement token refresh mechanism
- Add biometric authentication option
- Implement certificate pinning
- Add request rate limiting

---

## UI/UX Architecture

### 1. **Screen Structure**

**Common Screen Pattern:**
```dart
class ExampleScreen extends StatefulWidget {
  @override
  State<ExampleScreen> createState() => _ExampleScreenState();
}

class _ExampleScreenState extends State<ExampleScreen>
    with SingleTickerProviderStateMixin {
  
  // Animation controllers
  late AnimationController _animationController;
  late Animation<double> _fadeAnimation;
  late Animation<Offset> _slideAnimation;
  
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
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: FadeTransition(
        opacity: _fadeAnimation,
        child: SlideTransition(
          position: _slideAnimation,
          child: _buildContent(),
        ),
      ),
    );
  }
}
```

**Why this pattern?**
- Consistent animations across app
- Smooth transitions
- Professional feel
- Memory management (dispose)

### 2. **Animation System**

**Entry Animations:**

Every major screen uses fade + slide animations:

```dart
_animationController = AnimationController(
  duration: const Duration(milliseconds: 600-800),
  vsync: this,
);

_fadeAnimation = CurvedAnimation(
  parent: _animationController,
  curve: Curves.easeOut,
);

_slideAnimation = Tween<Offset>(
  begin: const Offset(0, 0.05),  // Slight upward offset
  end: Offset.zero,
).animate(_fadeAnimation);

_animationController.forward();  // Trigger animation
```

**Animation Timing:**
- Dashboard: 800ms
- Rooms Screen: 600ms
- Device Control: 600ms

**Curve Selection:**
- `Curves.easeOut`: Smooth deceleration
- Feels natural and responsive

### 3. **Widget Architecture**

**Reusable Widgets:**

1. **AppBottomNav** (`lib/widgets/app_bottom_nav.dart`)
   - 5 tabs: Dashboard, Devices, Rooms, Energy, Settings
   - Floating action button (FAB) integration
   - Active state highlighting

2. **DeviceTile** (`lib/widgets/device_tile.dart`)
   - Device icon with colored background
   - Title and subtitle
   - Toggle switch
   - Theme-aware colors

3. **StatCard** (`lib/widgets/stat_card.dart`)
   - Icon with background
   - Label and value
   - Used in dashboard overview

4. **StatusChip** (`lib/widgets/status_chip.dart`)
   - Online/offline indicators
   - Active device counts
   - Badge-style display

5. **PrimaryButton** (`lib/widgets/primary_button.dart`)
   - Consistent button styling
   - Loading states
   - Disabled states

6. **AppTextField** (`lib/widgets/app_text_field.dart`)
   - Custom text input
   - Validation support
   - Password visibility toggle

**Benefits of Widget Reusability:**
- Consistent UI
- Less code duplication
- Easier maintenance
- Centralized styling

---

## State Management

### 1. **State Management Approach**

**Primary: StatefulWidget**
- Most screens use local state
- Simple and straightforward
- No external dependencies
- Perfect for MVP

**Example:**
```dart
class _DashboardScreenState extends State<DashboardScreen> {
  String _userName = 'User';
  List<Map<String, dynamic>> _dashboardDevices = [...];
  int _selectedBottomTab = 0;
  
  void _toggleDevice(int index, bool value) {
    setState(() => _dashboardDevices[index]['isOn'] = value);
  }
}
```

### 2. **Secondary: ChangeNotifier (Observer Pattern)**

**ThemeController:**
- Global state (theme affects entire app)
- Multiple listeners needed
- State survives screen navigation

```dart
// In main.dart
AnimatedBuilder(
  animation: ThemeController.instance,
  builder: (_, __) {
    return MaterialApp(
      themeMode: ThemeController.instance.mode,
      // ...
    );
  },
)

// In settings screen
ElevatedButton(
  onPressed: () {
    ThemeController.instance.setTheme('Dark');
    // All AnimatedBuilders rebuild automatically
  },
)
```

### 3. **State Persistence**

**SessionManager Pattern:**
```dart
// Save state
await SessionManager.instance.saveUserName(name);

// Retrieve state
final name = await SessionManager.instance.getUserName();

// Clear state
await SessionManager.instance.clearSession();
```

**RoomStorage Pattern:**
```dart
// Load state
final rooms = await RoomStorage.loadRooms();
setState(() => _rooms = rooms);

// Update state
setState(() => _rooms[index]['name'] = newName);
await RoomStorage.saveRooms(_rooms);
```

**Pattern Benefits:**
- Synchronous UI updates (setState)
- Asynchronous persistence (async/await)
- Optimistic updates (UI first, save later)

---

## Navigation

### 1. **Navigation Strategy**

**Named Routes** (`lib/main.dart:126-134`):
```dart
routes: {
  '/login': (context) => const LoginScreen(),
  '/signup': (context) => const SignupScreen(),
  '/dashboard': (context) => const DashboardScreen(),
  '/rooms': (context) => const RoomsScreen(),
  '/devices': (context) => const AllDevicesScreen(),
  '/settings': (context) => const SettingsScreen(),
  '/energy': (context) => const EnergyScreen(),
}
```

**Benefits:**
- Centralized route definitions
- Easy deep linking (future)
- Type-safe navigation
- Clear app structure

### 2. **Navigation Patterns**

**Push Navigation:**
```dart
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (_) => ProfileScreen(),
  ),
);
```
- Adds to navigation stack
- Back button returns to previous screen

**Named Navigation:**
```dart
Navigator.pushNamed(context, '/dashboard');
```
- Uses route name
- Cleaner code

**Replacement Navigation:**
```dart
Navigator.pushReplacementNamed(context, '/dashboard');
```
- Replaces current screen
- No back navigation
- Used for bottom nav tabs

**Clear and Navigate:**
```dart
Navigator.pushNamedAndRemoveUntil(context, '/login', (_) => false);
```
- Clears entire stack
- Used for logout
- No back button to authenticated screens

### 3. **Bottom Navigation**

**Implementation** (`lib/screens/dashboard_screen.dart:160-185`):
```dart
AppBottomNav(
  selectedIndex: _selectedBottomTab,
  onTabSelected: (index) {
    if (index == _selectedBottomTab) return;
    setState(() => _selectedBottomTab = index);
    
    switch (index) {
      case 0: Navigator.pushReplacementNamed(context, '/dashboard'); break;
      case 1: Navigator.pushReplacementNamed(context, '/devices'); break;
      case 2: Navigator.pushReplacementNamed(context, '/rooms'); break;
      case 3: Navigator.pushReplacementNamed(context, '/energy'); break;
      case 4: Navigator.pushReplacementNamed(context, '/settings'); break;
    }
  },
  onFabPressed: () { },
)
```

**Key Points:**
- Uses `pushReplacementNamed` (no stack buildup)
- Prevents redundant navigation to same tab
- Updates selected tab state
- FAB action configurable per screen

---

## Theme System

### 1. **Theme Architecture**

**Two Complete Themes:**
- Light Theme (default)
- Dark Theme

**Theme Definition** (`lib/main.dart:79-109`):
```dart
ThemeData _lightTheme() {
  return ThemeData(
    useMaterial3: true,
    brightness: Brightness.light,
    fontFamily: 'Inter',
    primaryColor: AppColors.primary,
    scaffoldBackgroundColor: AppColors.bg,
    colorScheme: ColorScheme.fromSeed(
      seedColor: AppColors.primary,
      brightness: Brightness.light,
    ),
    inputDecorationTheme: InputDecorationTheme(
      border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)),
    ),
  );
}

ThemeData _darkTheme() {
  return ThemeData(
    useMaterial3: true,
    brightness: Brightness.dark,
    fontFamily: 'Inter',
    colorScheme: ColorScheme.fromSeed(
      seedColor: AppColors.primary,
      brightness: Brightness.dark,
    ),
    // ...
  );
}
```

### 2. **Color System**

**AppColors Strategy** (`lib/theme/app_colors.dart`):

1. **Static Colors** (consistent across themes):
```dart
static const Color primary = Color(0xFF4DA3FF);
static const Color onlineGreen = Color(0xFF16A34A);
static const Color warning = Color(0xFFF59E0B);
```

2. **Context-Aware Colors** (adapt to theme):
```dart
static Color bgOf(BuildContext context) {
  final isDark = Theme.of(context).brightness == Brightness.dark;
  return isDark ? const Color(0xFF0B1220) : Color(0xFFFFFFFF);
}

static Color cardOf(BuildContext context) {
  final isDark = Theme.of(context).brightness == Brightness.dark;
  return isDark ? const Color(0xFF111B2E) : Color(0xFFFFFFFF);
}

static Color headingOf(BuildContext context) {
  final isDark = Theme.of(context).brightness == Brightness.dark;
  return isDark ? const Color(0xFFE5E7EB) : Color(0xFF1F2A44);
}
```

**Usage:**
```dart
Container(
  color: AppColors.cardOf(context),  // Adapts to current theme
  child: Text(
    'Title',
    style: TextStyle(color: AppColors.headingOf(context)),
  ),
)
```

### 3. **Theme Switching**

**User Flow:**
```
Settings Screen
      ↓
Theme Dropdown
      ↓
Select (Light/Dark/System)
      ↓
ThemeController.setTheme(value)
      ↓
Save to SharedPreferences
      ↓
notifyListeners()
      ↓
AnimatedBuilder rebuilds
      ↓
MaterialApp applies new theme
      ↓
Entire app re-renders
```

**Implementation:**
```dart
// In SettingsScreen
DropdownButton<String>(
  value: ThemeController.instance.modeLabel,
  items: ['Light', 'Dark', 'System']
      .map((e) => DropdownMenuItem(value: e, child: Text(e)))
      .toList(),
  onChanged: (value) {
    if (value != null) {
      ThemeController.instance.setTheme(value);
    }
  },
)
```

**System Theme Mode:**
- Follows device theme
- Automatic switching
- Battery friendly (OLED dark mode)

---

## Code Structure

### Project Organization

```
lib/
├── main.dart                    # App entry point
├── models/                      # Data models
│   ├── device_item.dart
│   └── scene_item.dart
├── screens/                     # Full-page screens
│   ├── login_screen.dart
│   ├── signup_screen.dart
│   ├── dashboard_screen.dart
│   ├── rooms_screen.dart
│   ├── room_devices_screen.dart
│   ├── all_devices_screen.dart
│   ├── device_control_screen.dart
│   ├── energy_screen.dart
│   ├── settings_screen.dart
│   ├── profile_screen.dart
│   └── edit_profile_screen.dart
├── services/                    # Business logic
│   ├── auth_service.dart
│   ├── session_manager.dart
│   ├── room_storage.dart
│   └── theme_controller.dart
├── widgets/                     # Reusable components
│   ├── app_bottom_nav.dart
│   ├── device_tile.dart
│   ├── device_card.dart
│   ├── stat_card.dart
│   ├── status_chip.dart
│   ├── status_badge.dart
│   ├── scene_card.dart
│   ├── info_card.dart
│   ├── quick_action_tile.dart
│   ├── primary_button.dart
│   └── app_text_field.dart
├── theme/                       # Styling
│   ├── app_colors.dart
│   └── app_text_styles.dart
└── utils/                       # Helpers
    ├── form_validators.dart
    └── id_gen.dart
```

### Naming Conventions

**Files:**
- `snake_case.dart`
- Descriptive names
- Suffix indicates purpose: `_screen`, `_service`, `_widget`

**Classes:**
- `PascalCase`
- Clear, descriptive names
- Widgets end with specific type: `DeviceTile`, `StatCard`

**Variables:**
- `camelCase`
- Private: `_leadingUnderscore`
- Boolean: `isLoading`, `hasError`, `canSubmit`

**Constants:**
- `camelCase` for private: `_kTokenKey`
- `UPPER_SNAKE_CASE` for public constants

**Functions:**
- `camelCase`
- Private: `_leadingUnderscore`
- Verb-based: `_loadUserName()`, `_handleLogout()`, `_toggleDevice()`

### Code Quality Practices

1. **Type Safety:**
   - Explicit types where helpful
   - Use `dynamic` sparingly
   - Leverage Dart's type inference

2. **Null Safety:**
   - All code null-safe
   - Use `?` for nullable types
   - Use `??` operator for defaults

3. **Async/Await:**
   - Consistent async handling
   - Proper error handling with try-catch
   - Loading states during async operations

4. **Code Organization:**
   - Methods grouped by functionality
   - Private helpers at bottom
   - Clear separation of concerns

5. **Comments:**
   - Docstrings for public APIs
   - Inline comments for complex logic
   - Section dividers for readability

---

## Key Technical Decisions

### 1. **Local Storage for Device State**

**Decision**: Store device and room data locally (not on backend)

**Reasoning:**
- Faster user experience (no network latency)
- Offline functionality
- Simpler MVP implementation
- Privacy (data stays on device)

**Trade-offs:**
- No cross-device sync
- No backup/restore
- Each device has independent state

**Future Enhancement:**
- Add optional cloud sync
- Implement device-to-device sync
- Add backup/export functionality

### 2. **StatefulWidget vs State Management Library**

**Decision**: Use StatefulWidget with local state

**Reasoning:**
- Simple and straightforward
- No external dependencies
- Sufficient for current scope
- Easy to understand and maintain

**When to consider alternatives:**
- Complex state sharing between screens
- Need for state persistence across navigation
- Performance issues with rebuilds

### 3. **Dio over http package**

**Decision**: Use Dio for HTTP requests

**Reasoning:**
- Built-in interceptors (auth, logging)
- Better error handling
- Request/response transformation
- Timeout configuration
- More features out-of-box

### 4. **Material Design 3**

**Decision**: Use Material 3 design system

**Reasoning:**
- Modern, fresh look
- Better theming support
- Improved accessibility
- Consistent with Android design
- Rich component library

---

## Performance Considerations

### 1. **Lazy Loading**

**Lists:**
```dart
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ItemWidget(items[index]);
  },
)
```
- Only builds visible items
- Efficient memory usage
- Smooth scrolling

### 2. **Animation Optimization**

**const Constructors:**
```dart
const Icon(Icons.home)
const SizedBox(height: 16)
```
- Widgets created once
- Reused across rebuilds
- Reduces garbage collection

**SingleTickerProviderStateMixin:**
- Optimized animation controller
- Pauses when screen not visible
- Better battery life

### 3. **Image Loading** (if added)

Best practices for future:
- Use `CachedNetworkImage`
- Implement placeholders
- Handle loading/error states
- Optimize image sizes

### 4. **State Updates**

**Targeted Rebuilds:**
```dart
// Bad: Rebuilds entire screen
setState(() => _userName = name);

// Good: Only rebuilds necessary widget
ValueListenableBuilder(...)
```

**Future optimization:**
- Consider Provider for targeted rebuilds
- Use const widgets where possible
- Profile with Flutter DevTools

---

## Testing Strategy (Future)

### Recommended Test Structure:

```
test/
├── unit/
│   ├── services/
│   │   ├── auth_service_test.dart
│   │   ├── session_manager_test.dart
│   │   └── room_storage_test.dart
│   ├── models/
│   │   └── device_item_test.dart
│   └── utils/
│       └── form_validators_test.dart
├── widget/
│   ├── widgets/
│   │   ├── device_tile_test.dart
│   │   └── stat_card_test.dart
│   └── screens/
│       └── dashboard_screen_test.dart
└── integration/
    └── login_flow_test.dart
```

### Test Types:

1. **Unit Tests**: Services, utilities, models
2. **Widget Tests**: Individual widgets, screens
3. **Integration Tests**: Complete user flows

---

## Security Best Practices

### Current Implementation:

✅ HTTPS communication
✅ Bearer token authentication
✅ Session management
✅ Auto-logout on 401

### Recommended Enhancements:

🔒 **Secure Storage:**
```dart
// Instead of SharedPreferences
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

final storage = FlutterSecureStorage();
await storage.write(key: 'token', value: token);
```

🔒 **Certificate Pinning:**
```dart
// Dio with certificate pinning
final dio = Dio();
dio.httpClientAdapter = IOHttpClientAdapter()
  ..onHttpClientCreate = (client) {
    client.badCertificateCallback = (cert, host, port) => false;
    return client;
  };
```

🔒 **Biometric Authentication:**
```dart
import 'package:local_auth/local_auth.dart';

final auth = LocalAuthentication();
final canAuth = await auth.canCheckBiometrics;
if (canAuth) {
  final authenticated = await auth.authenticate(
    localizedReason: 'Unlock Smart Home',
  );
}
```

🔒 **Input Sanitization:**
- Validate all user inputs
- Escape special characters
- Limit input lengths
- Use prepared statements (if SQL)

---

## Deployment Guide

### Build Configurations:

**Android:**
```bash
flutter build apk --release
flutter build appbundle --release
```

**iOS:**
```bash
flutter build ios --release
```

### Environment Variables:

```dart
const apiUrl = String.fromEnvironment(
  'API_BASE_URL',
  defaultValue: 'https://landlordbd-1.onrender.com/api',
);
```

Build with custom API:
```bash
flutter build apk --dart-define=API_BASE_URL=https://your-api.com
```

### Version Management:

In `pubspec.yaml`:
```yaml
version: 1.0.0+1
#        ^     ^
#        |     Build number
#        Version name
```

---

## Troubleshooting

### Common Issues:

**1. Login Not Working**
- Check API_BASE_URL
- Verify network connection
- Check token extraction logic
- Verify API response format

**2. Devices Not Saving**
- Check session validity
- Verify user is logged in
- Check SharedPreferences permissions
- Ensure username is saved

**3. Theme Not Persisting**
- Check SharedPreferences initialization
- Verify ThemeController.load() called
- Check for async timing issues

**4. Bottom Nav Not Working**
- Verify route names match
- Check selectedIndex state
- Ensure proper replacement navigation

---

## Future Enhancements

### Planned Features:

1. **Cloud Sync:**
   - Backend API for device states
   - Real-time sync across devices
   - Conflict resolution

2. **Automation/Scenes:**
   - Create custom scenes
   - Schedule-based automation
   - Trigger-based rules

3. **Voice Control:**
   - Integration with assistants
   - Voice commands for devices
   - Feedback through audio

4. **Analytics:**
   - Energy usage charts
   - Device usage patterns
   - Cost calculations

5. **Notifications:**
   - Device status changes
   - Alerts for offline devices
   - Scheduled reminders

6. **Multi-user Support:**
   - Family accounts
   - Permission levels
   - Activity logs

7. **Widget Support:**
   - Home screen widgets
   - Quick device controls
   - Status at-a-glance

---

## Conclusion

This Smart Home application demonstrates a well-structured Flutter application with:

- **Clean Architecture**: Clear separation of concerns
- **Scalable Design**: Easy to add new features
- **Modern UI**: Material Design 3 with theme support
- **Offline-First**: Local data persistence
- **Security-Conscious**: Token-based auth with auto-refresh
- **User-Friendly**: Smooth animations and intuitive navigation

The codebase is production-ready for MVP with clear paths for enhancement and scaling.

---

## Additional Resources

### Flutter Documentation:
- [Flutter Docs](https://docs.flutter.dev)
- [Dart Language Tour](https://dart.dev/guides/language/language-tour)
- [Material Design 3](https://m3.material.io)

### Packages Used:
- [Dio](https://pub.dev/packages/dio) - HTTP client
- [SharedPreferences](https://pub.dev/packages/shared_preferences) - Local storage
- [Flutter](https://flutter.dev) - UI framework

### Development Tools:
- [Flutter DevTools](https://docs.flutter.dev/development/tools/devtools/overview)
- [Android Studio](https://developer.android.com/studio)
- [VS Code](https://code.visualstudio.com)

---

**Document Version**: 1.0  
**Last Updated**: 2025  
**Author**: Smart Home Development Team
