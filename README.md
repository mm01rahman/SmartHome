# Smart Home Application

A modern Flutter-based mobile application for controlling and monitoring smart home devices with a beautiful Material Design 3 interface.

## 📱 Features

- **User Authentication**: Secure login and signup with JWT token management
- **Dashboard**: Overview of rooms, devices, and energy consumption
- **Room Management**: Create, edit, and organize rooms with custom icons
- **Device Control**: Control various device types (lights, fans, thermostats, air purifiers, smart sockets)
- **Theme Support**: Light, Dark, and System theme modes
- **Offline Storage**: Device states persist locally for instant access
- **Energy Monitoring**: Track energy consumption across devices
- **User Profile**: Manage user settings and preferences
- **Smooth Animations**: Professional entry animations and transitions

## 🏗️ Architecture

This project follows a clean, layered architecture with:
- **Presentation Layer**: Screens and reusable widgets
- **Business Logic**: Services, controllers, and utilities
- **Data Layer**: Models, storage management, and API clients

### Tech Stack

- **Framework**: Flutter 3.x with Dart 3.4.3+
- **State Management**: StatefulWidget with ChangeNotifier pattern
- **HTTP Client**: Dio for API communication
- **Local Storage**: SharedPreferences for data persistence
- **UI Framework**: Material Design 3

## 📚 Documentation

For a comprehensive, in-depth explanation of how the application works, see:

**[ARCHITECTURE.md](ARCHITECTURE.md)** - Complete architectural documentation including:
- Detailed architecture and design patterns
- Complete application flow diagrams
- Service layer explanations
- Data management strategies
- Authentication system deep dive
- UI/UX architecture
- State management patterns
- Navigation system
- Theme system implementation
- Code structure and organization
- Security best practices
- Performance considerations
- Testing strategies
- Future enhancements

## 🚀 Getting Started

### Prerequisites

- Flutter SDK 3.x or higher
- Dart 3.4.3 or higher
- Android Studio / VS Code with Flutter extensions
- An Android/iOS device or emulator

### Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/mm01rahman/SmartHome.git
   cd SmartHome
   ```

2. **Install dependencies**:
   ```bash
   flutter pub get
   ```

3. **Run the application**:
   ```bash
   flutter run
   ```

### Configuration

The app connects to a backend API. To change the API endpoint:

```bash
flutter run --dart-define=API_BASE_URL=https://your-api-url.com/api
```

Or modify the default in `lib/services/auth_service.dart`:
```dart
static const String _baseUrl = String.fromEnvironment(
  'API_BASE_URL',
  defaultValue: 'https://your-api-url.com/api',
);
```

## 📂 Project Structure

```
lib/
├── main.dart                 # Application entry point
├── models/                   # Data models
├── screens/                  # Full-page screens
│   ├── login_screen.dart
│   ├── dashboard_screen.dart
│   ├── rooms_screen.dart
│   └── ...
├── services/                 # Business logic
│   ├── auth_service.dart
│   ├── session_manager.dart
│   ├── room_storage.dart
│   └── theme_controller.dart
├── widgets/                  # Reusable UI components
├── theme/                    # Styling and theming
└── utils/                    # Helper functions
```

## 🎨 Features Overview

### Authentication
- Secure JWT-based authentication
- Auto-logout on token expiration
- Session persistence across app restarts

### Dashboard
- Time-based greetings (Morning/Afternoon/Evening)
- Device status overview
- Quick device controls
- Energy consumption tracking

### Room Management
- Create custom rooms with icons
- Organize devices by room
- Rename and delete rooms
- Room-specific device lists

### Device Control
- **Lights**: Brightness control (0-100%)
- **Fans**: Speed control (1-5)
- **Thermostats**: Temperature control (16-30°C)
- **Air Purifiers**: Mode selection (Auto/Sleep/Turbo)
- **Smart Sockets**: On/Off toggle

### Theme System
- Light theme (default)
- Dark theme (OLED-friendly)
- System theme (follows device settings)
- Persistent theme selection

## 🔐 Security

- HTTPS communication with backend API
- Bearer token authentication
- Automatic token management
- 401 error handling with auto-logout
- Input validation on all forms

**Recommended for Production:**
- Implement Flutter Secure Storage for tokens
- Add certificate pinning
- Enable biometric authentication
- Implement token refresh mechanism

## 🧪 Testing

Run tests with:
```bash
flutter test
```

For widget tests:
```bash
flutter test test/widget
```

## 📱 Building for Production

### Android
```bash
# APK
flutter build apk --release

# App Bundle (recommended for Play Store)
flutter build appbundle --release
```

### iOS
```bash
flutter build ios --release
```

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 👥 Authors

- **mm01rahman** - Initial work

## 🙏 Acknowledgments

- Flutter team for the amazing framework
- Material Design team for the design system
- All contributors who have helped improve this project

## 📞 Support

If you have any questions or need help, please:
- Open an issue on GitHub
- Check the [ARCHITECTURE.md](ARCHITECTURE.md) for detailed documentation
- Review Flutter documentation at [docs.flutter.dev](https://docs.flutter.dev/)

## 🔮 Future Enhancements

- Cloud sync for device states
- Real-time device status updates
- Automation and scene creation
- Voice control integration
- Analytics and usage reports
- Multi-user support with permissions
- Home screen widgets
- Push notifications

---

**Made with ❤️ using Flutter**
