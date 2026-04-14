# Local Storage — Flutter

## Contents

- Storage options overview
- flutter_secure_storage (tokens)
- SharedPreferences (settings)
- Drift / SQLite (structured data)
- Hive (NoSQL alternative)
- Storage abstraction

## Storage Options Overview

| Storage | Use Case | Encryption | Performance |
|---------|----------|------------|-------------|
| `flutter_secure_storage` | Tokens, secrets, API keys | Yes (Keychain/Keystore) | Slow — encrypted I/O |
| `SharedPreferences` | User settings, flags, simple key-value | No | Fast |
| `Drift` (SQLite) | Structured data, relations, queries | No (optional) | Fast for queries |
| `Hive` | NoSQL collections, offline cache | Optional | Very fast |

**Rule:** Choose based on data sensitivity and structure needs. Most apps need `flutter_secure_storage` + `SharedPreferences` at minimum.

## flutter_secure_storage (Tokens) [LOW]

### Secure Storage Wrapper

```dart
// core/storage/secure_storage.dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:injectable/injectable.dart';

@lazySingleton
class SecureStorage {
  final FlutterSecureStorage _storage;

  SecureStorage()
    : _storage = const FlutterSecureStorage(
        aOptions: AndroidOptions(encryptedSharedPreferences: true),
        iOptions: IOSOptions(
          accessibility: KeychainAccessibility.first_unlock_this_device,
        ),
      );

  // ─── Keys ──────────────────────────────────────────
  static const _accessTokenKey = 'access_token';
  static const _refreshTokenKey = 'refresh_token';

  // ─── Token Operations ──────────────────────────────
  Future<void> saveTokens({
    required String accessToken,
    required String refreshToken,
  }) async {
    await Future.wait([
      _storage.write(key: _accessTokenKey, value: accessToken),
      _storage.write(key: _refreshTokenKey, value: refreshToken),
    ]);
  }

  Future<String?> getAccessToken() =>
      _storage.read(key: _accessTokenKey);

  Future<String?> getRefreshToken() =>
      _storage.read(key: _refreshTokenKey);

  Future<void> clearTokens() async {
    await Future.wait([
      _storage.delete(key: _accessTokenKey),
      _storage.delete(key: _refreshTokenKey),
    ]);
  }

  Future<void> clearAll() => _storage.deleteAll();
}
```

### Platform Configuration

**Android** (`android/app/build.gradle`):
- Minimum SDK 18+ (required for encrypted shared preferences)

**iOS**: No extra config needed — uses Keychain automatically.

## SharedPreferences (Settings) [MEDIUM]

### Local Storage Wrapper

```dart
// core/storage/local_storage.dart
import 'package:shared_preferences/shared_preferences.dart';
import 'package:injectable/injectable.dart';

@lazySingleton
class LocalStorage {
  final SharedPreferences _prefs;

  LocalStorage(this._prefs);

  // ─── Keys ──────────────────────────────────────────
  static const _themeKey = 'theme_mode';
  static const _localeKey = 'locale';
  static const _onboardingKey = 'onboarding_complete';
  static const _lastSyncKey = 'last_sync_timestamp';

  // ─── Theme ─────────────────────────────────────────
  ThemeMode get themeMode {
    final value = _prefs.getString(_themeKey);
    return switch (value) {
      'light' => ThemeMode.light,
      'dark' => ThemeMode.dark,
      _ => ThemeMode.system,
    };
  }

  Future<void> setThemeMode(ThemeMode mode) =>
      _prefs.setString(_themeKey, mode.name);

  // ─── Locale ────────────────────────────────────────
  String? get locale => _prefs.getString(_localeKey);

  Future<void> setLocale(String locale) =>
      _prefs.setString(_localeKey, locale);

  // ─── Onboarding ────────────────────────────────────
  bool get isOnboardingComplete =>
      _prefs.getBool(_onboardingKey) ?? false;

  Future<void> setOnboardingComplete() =>
      _prefs.setBool(_onboardingKey, true);

  // ─── Sync Timestamp ────────────────────────────────
  DateTime? get lastSyncTime {
    final ms = _prefs.getInt(_lastSyncKey);
    return ms != null ? DateTime.fromMillisecondsSinceEpoch(ms) : null;
  }

  Future<void> setLastSyncTime(DateTime time) =>
      _prefs.setInt(_lastSyncKey, time.millisecondsSinceEpoch);

  // ─── Clear ─────────────────────────────────────────
  Future<void> clear() => _prefs.clear();
}
```

### DI Registration

```dart
// In injection module
@module
abstract class StorageModule {
  @preResolve
  @lazySingleton
  Future<SharedPreferences> get prefs => SharedPreferences.getInstance();
}
```

## Drift / SQLite (Structured Data) [MEDIUM]

Use Drift when you need:
- Relational data with foreign keys
- Complex queries (joins, aggregates)
- Large datasets with pagination
- Full-text search

### Database Setup

```dart
// core/storage/database/app_database.dart
import 'package:drift/drift.dart';

part 'app_database.g.dart';

class CachedItems extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get remoteId => text().unique()();
  TextColumn get title => text()();
  TextColumn get description => text().nullable()();
  TextColumn get jsonData => text()(); // Store full JSON for offline
  DateTimeColumn get cachedAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().withDefault(currentDateAndTime)();
}

@DriftDatabase(tables: [CachedItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase(super.e);

  @override
  int get schemaVersion => 1;

  @override
  MigrationStrategy get migration => MigrationStrategy(
    onCreate: (m) => m.createAll(),
    onUpgrade: (m, from, to) async {
      // Handle migrations between versions
    },
  );
}
```

## Hive (NoSQL Alternative) [MEDIUM]

Use Hive when you need:
- Simple key-value or document storage
- Very fast read/write (no SQL overhead)
- No relational queries needed

### Setup

```dart
// In bootstrap
await Hive.initFlutter();
Hive.registerAdapter(CachedItemAdapter());
await Hive.openBox<CachedItem>('cachedItems');
```

### Box Wrapper

```dart
class CachedItemStorage {
  final Box<CachedItem> _box;

  CachedItemStorage(this._box);

  List<CachedItem> getAll() => _box.values.toList();

  CachedItem? get(String id) => _box.get(id);

  Future<void> put(CachedItem item) => _box.put(item.id, item);

  Future<void> putAll(List<CachedItem> items) async {
    final map = {for (final item in items) item.id: item};
    await _box.putAll(map);
  }

  Future<void> delete(String id) => _box.delete(id);

  Future<void> clear() => _box.clear();
}
```
