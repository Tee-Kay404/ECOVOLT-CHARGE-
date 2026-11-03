# EcoVolt Charge (EVC) - Monorepo

A professional on-demand EV charging platform built with Flutter and Serverpod.

## 📁 Project Structure

```
eco_volt_charge/
├── evc_server/              # Serverpod Backend (Dart)
│   ├── lib/
│   │   └── src/
│   │       ├── endpoints/   # API Controllers
│   │       └── generated/   # Auto-generated protocol
│   ├── migrations/          # Database Migrations
│   ├── config/              # Environment configs
│   └── docker-compose.yaml  # PostgreSQL + PostGIS + Redis
│
├── evc_client/              # Generated Client (DO NOT MODIFY)
│   └── lib/
│       └── src/
│           └── protocol/    # Auto-generated from server models
│
├── apps/                    # Frontend Applications
│   ├── evc_user/           # Consumer Mobile App (iOS/Android/Web)
│   ├── evc_driver/         # Service Provider Mobile App
│   └── evc_admin/          # Admin Dashboard (Web)
│
└── melos.yaml              # Monorepo management
```

---

## 🛠 Prerequisites

Before setting up the project, ensure you have the following installed:

| Software | Version | Installation |
|----------|---------|--------------|
| Flutter | 3.22+ | [flutter.dev](https://flutter.dev/docs/get-started/install) |
| Dart | 3.6+ | Included with Flutter |
| Docker Desktop | Latest | [docker.com](https://www.docker.com/products/docker-desktop) |
| Melos | Latest | `dart pub global activate melos` |
| Git | Latest | [git-scm.com](https://git-scm.com/) |

### Verify Installation

```bash
flutter --version
dart --version
docker --version
melos --version
```

---

## 🚀 Quick Setup (Step-by-Step)

### Step 1: Clone the Repository

```bash
git clone https://github.com/your-org/eco_volt_charge.git
cd eco_volt_charge
```

### Step 2: Install Melos

```bash
dart pub global activate melos
```

### Step 3: Bootstrap the Monorepo

This installs all dependencies for all packages:

```bash
melos bootstrap
```

> ⚠️ **If `melos bootstrap` hangs for more than 5 minutes:**
> 1. Press `Ctrl+C` to cancel
> 2. Clear pub cache: `flutter pub cache clean` (select 'n' when prompted)
> 3. Install packages individually:
>    ```bash
>    cd evc_client && dart pub get && cd ..
>    cd evc_server && dart pub get && cd ..
>    cd apps/evc_user && flutter pub get && cd ../..
>    cd apps/evc_driver && flutter pub get && cd ../..
>    cd apps/evc_admin && flutter pub get && cd ../..
>    ```

### Step 4: Start Docker Containers

```bash
cd evc_server
docker compose up -d
```

This starts:
| Container | Port | Purpose |
|-----------|------|---------|
| evc_postgres | 8090 | PostgreSQL + PostGIS |
| evc_redis | 8091 | Redis cache |
| evc_postgres_test | 9090 | Test database |
| evc_redis_test | 9091 | Test Redis |

**Verify containers are running:**

```bash
docker ps
```

All 4 containers should show status "Up".

### Step 5: Apply Database Migrations

```bash
cd evc_server
dart bin/main.dart --apply-migrations
```

> ✅ Migrations should complete in **seconds**, not minutes. If it takes longer, check Docker container logs:
> ```bash
> docker compose logs postgres
> ```

### Step 6: Start the Server

```bash
cd evc_server
dart bin/main.dart
```

**Expected output:**
```
SERVERPOD version: 3.2.3, dart: 3.x.x
...
SERVERPOD initialized
```

### Step 7: Run a Flutter App

Open a **new terminal** and run:

```bash
# User App (Web)
cd apps/evc_user
flutter run -d chrome

# Driver App (Web)
cd apps/evc_driver
flutter run -d chrome

# Admin Dashboard (Web)
cd apps/evc_admin
flutter run -d chrome
```

---

## 📱 Running on Mobile Devices

### Android
```bash
cd apps/evc_user
flutter run -d android
```

### iOS (macOS only)
```bash
cd apps/evc_user
flutter run -d ios
```

### Windows Desktop
Requires Visual Studio with C++ workload:
```bash
cd apps/evc_user
flutter run -d windows
```

---

## 🔧 Configuration

### Server Passwords

Database credentials are in `evc_server/config/passwords.yaml`:

```yaml
development:
  database: 'evc_dev_password_2026'
  redis: 'evc_redis_dev_2026'
```

> ⚠️ **Important:** These must match the passwords in `docker-compose.yaml`

### API Endpoint

Flutter apps connect to `http://localhost:8080/` by default. To change, modify `main.dart`:

```dart
final client = Client(
  'http://your-server-url:8080/',
  authenticationKeyManager: FlutterAuthenticationKeyManager(),
);
```

---

## 📋 Common Commands

### Melos Commands

```bash
melos bootstrap      # Install all dependencies
melos clean          # Clean all packages
melos run test       # Run tests in all packages
```

### Docker Commands

```bash
docker compose up -d          # Start containers
docker compose down           # Stop containers
docker compose logs -f        # View logs
docker compose down -v        # Reset database (DELETES ALL DATA)
```

### Serverpod Commands

```bash
cd evc_server
serverpod generate                    # Generate code after protocol changes
dart bin/main.dart --apply-migrations # Apply database migrations
serverpod create-migration            # Create new migration
```

---

## 🐛 Troubleshooting

### ❌ `melos bootstrap` hangs indefinitely

**Cause:** Network issues or pub.dev connectivity problems

**Solution:**
1. Cancel with `Ctrl+C`
2. Run: `flutter pub cache clean` (select 'n' to skip full clean)
3. Install packages individually (see Step 3 above)

---

### ❌ Docker containers not starting

**Cause:** Port conflicts or Docker not running

**Solution:**
```bash
# Check if ports are in use
netstat -ano | findstr "8090"
netstat -ano | findstr "8091"

# Stop all Docker containers
docker stop $(docker ps -q)

# Start fresh
docker compose down -v
docker compose up -d
```

---

### ❌ Migration fails or takes too long

**Cause:** Database not accessible or stuck process

**Solution:**
1. Verify containers: `docker ps` (all 4 should be "Up")
2. Test database connection:
   ```bash
   docker exec -it evc_postgres psql -U postgres -d eco_volt_charge -c "\dt"
   ```
3. Check logs: `docker compose logs postgres`

---

### ❌ `FlutterAuthenticationKeyManager` not found

**Cause:** Missing dependency

**Solution:**
Add to `pubspec.yaml`:
```yaml
dependencies:
  serverpod_auth_shared_flutter: ^2.9.0
```
Then run: `flutter pub get`

---

### ❌ "setState() called during build" error

**Cause:** Calling `notifyListeners()` during widget build phase

**Solution:**
Wrap initialization in `addPostFrameCallback`:
```dart
@override
void initState() {
  super.initState();
  WidgetsBinding.instance.addPostFrameCallback((_) {
    _initialize();
  });
}

---

## 🏗️ Architecture

### Backend Stack
| Technology | Version | Purpose |
|------------|---------|---------|
| Serverpod | 3.2.3 | Backend framework |
| PostgreSQL | 14 | Primary database |
| PostGIS | 3.3 | Geospatial queries |
| Redis | 7.2 | Caching & sessions |

### Frontend Stack
| Technology | Purpose |
|------------|---------|
| Flutter | Cross-platform UI |
| Provider | State management |
| Mapbox Maps(or open street map) | Map integration|
| Serverpod Client | API communication |

### Key Features
- 🗺️ Real-time driver tracking with PostGIS geospatial queries
- 📱 Multi-app architecture with shared client library
- 🔐 Role-based access control (User, Driver, Admin)
- ⚡ Live location updates via WebSocket streams

---

## 📱 Applications

| App | Platform | Description |
|-----|----------|-------------|
| **evc_user** | iOS, Android, Web | Consumer app for requesting EV charging |
| **evc_driver** | iOS, Android | Driver app for accepting and fulfilling requests |
| **evc_admin** | Web | Admin dashboard for monitoring and management |

---

## 🗄️ Database Schema

### Core Tables
| Table | Description |
|-------|-------------|
| `charging_units` | Driver vehicles with geolocation (lat, lng, status) |
| `charging_requests` | Service requests with status tracking |

---

## 📦 Package Versions

```yaml
# Server
serverpod: 3.2.3

# Flutter Apps
flutter: 3.22+
provider: ^6.1.2
serverpod_flutter: ^2.9.0
serverpod_auth_shared_flutter: ^2.9.0
mapbox_maps_flutter: ^2.4.0
```

---

## 🤝 Contributing

1. Create a feature branch: `git checkout -b feature/my-feature`
2. Make changes and test locally
3. Run `serverpod generate` if protocol files changed
4. Run all apps to verify no errors
5. Commit with descriptive message
6. Push and create a Pull Request

---

## 📄 License

Copyright © 2026 EcoVolt Charge. All rights reserved.
