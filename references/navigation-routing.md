# Navigation & Routing — GoRouter

## Contents

- GoRouter setup
- Route definitions
- Auth-aware redirects
- Deep linking
- Navigation patterns
- Nested navigation (bottom nav)

## GoRouter Setup [LOW]

### Router Configuration

```dart
// core/router/app_router.dart
import 'package:go_router/go_router.dart';

class AppRouter {
  final AuthBloc authBloc; // or ref for Riverpod

  AppRouter({required this.authBloc});

  late final GoRouter router = GoRouter(
    initialLocation: RouteNames.splash,
    debugLogDiagnostics: EnvConfig.instance.enableLogging,
    refreshListenable: authBloc, // re-evaluate redirect on auth change
    redirect: _redirect,
    routes: _routes,
    errorBuilder: (context, state) => ErrorScreen(
      error: state.error?.message ?? 'Page not found',
    ),
  );

  String? _redirect(BuildContext context, GoRouterState state) {
    final isAuthenticated = authBloc.state is AuthAuthenticated;
    final isAuthRoute = state.matchedLocation == RouteNames.login ||
        state.matchedLocation == RouteNames.register;
    final isSplash = state.matchedLocation == RouteNames.splash;

    // Splash handles its own routing
    if (isSplash) return null;

    // Not authenticated → go to login
    if (!isAuthenticated && !isAuthRoute) return RouteNames.login;

    // Authenticated but on auth page → go to home
    if (isAuthenticated && isAuthRoute) return RouteNames.home;

    return null; // No redirect needed
  }

  List<RouteBase> get _routes => [
    GoRoute(
      path: RouteNames.splash,
      name: 'splash',
      builder: (context, state) => const SplashScreen(),
    ),
    GoRoute(
      path: RouteNames.login,
      name: 'login',
      builder: (context, state) => const LoginScreen(),
    ),
    GoRoute(
      path: RouteNames.register,
      name: 'register',
      builder: (context, state) => const RegisterScreen(),
    ),
    ShellRoute(
      builder: (context, state, child) => MainShell(child: child),
      routes: [
        GoRoute(
          path: RouteNames.home,
          name: 'home',
          builder: (context, state) => const HomeScreen(),
        ),
        GoRoute(
          path: RouteNames.profile,
          name: 'profile',
          builder: (context, state) => const ProfileScreen(),
        ),
        GoRoute(
          path: RouteNames.settings,
          name: 'settings',
          builder: (context, state) => const SettingsScreen(),
        ),
      ],
    ),
  ];
}
```

### Route Names

```dart
// core/router/route_names.dart
class RouteNames {
  RouteNames._();

  static const splash = '/splash';
  static const login = '/login';
  static const register = '/register';
  static const home = '/home';
  static const profile = '/profile';
  static const settings = '/settings';
}
```

## Auth-Aware Redirects [LOW]

The redirect function runs on EVERY navigation. Rules:

1. **Splash screen**: No redirect — splash checks auth state and navigates
2. **Unauthenticated + non-auth page**: Redirect to login
3. **Authenticated + auth page**: Redirect to home
4. **Otherwise**: No redirect (allow navigation)

### Refresh on Auth Change

```dart
// For BLoC: make AuthBloc extend ChangeNotifier or use GoRouterRefreshStream
class GoRouterRefreshStream extends ChangeNotifier {
  late final StreamSubscription<dynamic> _subscription;

  GoRouterRefreshStream(Stream<dynamic> stream) {
    _subscription = stream.asBroadcastStream().listen((_) {
      notifyListeners();
    });
  }

  @override
  void dispose() {
    _subscription.cancel();
    super.dispose();
  }
}

// Usage:
refreshListenable: GoRouterRefreshStream(authBloc.stream),
```

## Deep Linking [MEDIUM]

### Android: `AndroidManifest.xml`

```xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data
        android:scheme="https"
        android:host="example.com"
        android:pathPrefix="/app" />
</intent-filter>
```

### iOS: `Info.plist` + Associated Domains

```xml
<!-- Info.plist -->
<key>FlutterDeepLinkingEnabled</key>
<true/>
```

Add Associated Domains in Xcode: `applinks:example.com`

### GoRouter Handles Deep Links Automatically

Routes defined with proper paths are automatically matched to deep links. No extra configuration needed in GoRouter itself.

## Navigation Patterns

### Navigate To

```dart
// Push a new route
context.go('/home');           // Replace entire stack
context.push('/home/detail');  // Push onto stack
context.pop();                 // Go back

// With parameters
context.go('/user/${user.id}');

// With query parameters
context.go('/search?q=flutter');

// Named routes
context.goNamed('profile', pathParameters: {'id': user.id});
```

### Pass Data via Extra

```dart
// Route definition
GoRoute(
  path: '/detail/:id',
  builder: (context, state) {
    final item = state.extra as Item?;
    final id = state.pathParameters['id']!;
    return DetailScreen(id: id, item: item);
  },
),

// Navigation
context.push('/detail/${item.id}', extra: item);
```

## Nested Navigation (Bottom Navigation Bar)

### Using StatefulShellRoute

```dart
StatefulShellRoute.indexedStack(
  builder: (context, state, navigationShell) {
    return ScaffoldWithBottomNav(
      navigationShell: navigationShell,
    );
  },
  branches: [
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: '/home',
          builder: (context, state) => const HomeScreen(),
          routes: [
            GoRoute(
              path: 'detail/:id',
              builder: (context, state) => DetailScreen(
                id: state.pathParameters['id']!,
              ),
            ),
          ],
        ),
      ],
    ),
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: '/search',
          builder: (context, state) => const SearchScreen(),
        ),
      ],
    ),
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: '/profile',
          builder: (context, state) => const ProfileScreen(),
        ),
      ],
    ),
  ],
),
```

### Bottom Nav Shell

```dart
class ScaffoldWithBottomNav extends StatelessWidget {
  final StatefulNavigationShell navigationShell;

  const ScaffoldWithBottomNav({required this.navigationShell});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell,
      bottomNavigationBar: NavigationBar(
        selectedIndex: navigationShell.currentIndex,
        onDestinationSelected: (index) {
          navigationShell.goBranch(
            index,
            initialLocation: index == navigationShell.currentIndex,
          );
        },
        destinations: const [
          NavigationDestination(icon: Icon(Icons.home), label: 'Home'),
          NavigationDestination(icon: Icon(Icons.search), label: 'Search'),
          NavigationDestination(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }
}
```
