# EcoVolt Charge - Project Architecture Guide

## Overview

**EcoVolt Charge (EVC)** is an on-demand EV charging service that connects users with mobile charging units. This Phase 1 implementation includes:

- **User Mobile App** (Flutter)
- **Backend API** (Serverpod/Dart)
- **Database** (PostgreSQL + PostGIS)

---

## Project Structure (Monorepo)

```
eco_volt_charge/
├── eco_volt_charge_server/     # Serverpod backend
│   ├── lib/
│   │   └── src/
│   │       ├── endpoints/       # API endpoints
│   │       ├── generated/       # Auto-generated protocol files
│   │       └── models/          # Data model definitions (.yaml)
│   ├── migrations/              # Database migrations
│   └── docker-compose.yaml      # PostgreSQL + PostGIS, Redis
│
├── eco_volt_charge_client/      # Generated Dart client library
│   └── lib/
│       └── src/
│           └── protocol/        # Generated protocol classes
│
└── eco_volt_charge_flutter/     # Flutter user app
    └── lib/
        ├── main.dart
        └── src/
            ├── constants/       # App constants
            ├── providers/       # Riverpod providers
            ├── repositories/    # Data repositories
            └── screens/         # UI screens
```

---

## Data Models

### ChargingUnit (`charging_unit.yaml`)

Represents a mobile EV charging vehicle/unit.

| Field | Type | Description |
|-------|------|-------------|
| `id` | int | Primary key (auto) |
| `name` | String | Display name (e.g., "EVC Unit #001") |
| `status` | String | `available`, `busy`, `offline` |
| `lat` | double | GPS latitude |
| `lng` | double | GPS longitude |
| `lastActive` | DateTime | Last position update time |

### ChargingRequest (`charging_request.yaml`)

Represents a user's request for charging service.

| Field | Type | Description |
|-------|------|-------------|
| `id` | int | Primary key (auto) |
| `customerId` | int | User ID |
| `unitId` | int? | Assigned unit (nullable until assigned) |
| `status` | String | `pending`, `accepted`, `en_route`, `charging`, `completed`, `cancelled` |
| `pickupLat` | double | Service location latitude |
| `pickupLng` | double | Service location longitude |
| `pickupAddress` | String? | Optional address description |
| `createdAt` | DateTime | Request creation time |
| `updatedAt` | DateTime? | Last status update |
| `completedAt` | DateTime? | Completion timestamp |

---

## Backend API (ChargeEndpoint)

### Core Endpoints

| Method | Description |
|--------|-------------|
| `requestCharge(lat, lng, pickupAddress?)` | Request charge at location, auto-assigns nearest unit |
| `findNearestAvailableUnit(lat, lng, maxDistanceKm)` | PostGIS query to find nearest unit |
| `getAllUnits()` | Get all charging units (for map display) |
| `getAvailableUnits()` | Get only available units |
| `getUnitsNearLocation(lat, lng, radiusKm)` | Get units within radius |
| `getRequest(requestId)` | Get specific request details |
| `getCustomerRequests(customerId)` | Get all requests for a customer |
| `updateRequestStatus(requestId, newStatus)` | Update request status |
| `cancelRequest(requestId)` | Cancel an active request |

### Development Endpoints

| Method | Description |
|--------|-------------|
| `seedDummyUnits(centerLat, centerLng, radiusKm, count)` | Create test units |
| `resetAllUnits()` | Reset all units to available |
| `getUnitStats()` | Get unit count statistics |

---

## PostGIS Geospatial Queries

The backend uses **PostGIS** for accurate geospatial calculations:

```sql
-- Find nearest available unit within 10km
SELECT 
    cu.*,
    ST_Distance(
        ST_SetSRID(ST_MakePoint(cu.lng, cu.lat), 4326)::geography,
        ST_SetSRID(ST_MakePoint($user_lng, $user_lat), 4326)::geography
    ) as distance_meters
FROM charging_units cu
WHERE cu.status = 'available'
    AND ST_DWithin(
        ST_SetSRID(ST_MakePoint(cu.lng, cu.lat), 4326)::geography,
        ST_SetSRID(ST_MakePoint($user_lng, $user_lat), 4326)::geography,
        $max_distance_meters
    )
ORDER BY distance_meters ASC
LIMIT 1;
```

**Key PostGIS Functions:**
- `ST_MakePoint(lng, lat)` - Creates a geometry point
- `ST_SetSRID(..., 4326)` - Sets WGS84 coordinate system
- `::geography` - Casts to geography type for meter-based calculations
- `ST_Distance()` - Returns distance in meters
- `ST_DWithin()` - Efficient radius filtering

---

## Getting Started

### 1. Start Infrastructure

```bash
cd eco_volt_charge_server
docker-compose up -d
```

### 2. Generate Serverpod Code

```bash
cd eco_volt_charge_server
serverpod generate
```

### 3. Run Database Migrations

```bash
cd eco_volt_charge_server
dart bin/main.dart --apply-migrations
```

### 4. Enable PostGIS

```bash
# Connect to PostgreSQL and run:
psql -h localhost -p 8090 -U postgres -d eco_volt_charge
\i migrations/postgis_setup.sql
```

### 5. Start the Server

```bash
cd eco_volt_charge_server
dart bin/main.dart
```

### 6. Configure Mapbox (Flutter)

See `MAPBOX_SETUP.md` for detailed instructions.

### 7. Run Flutter App

```bash
cd eco_volt_charge_flutter
flutter pub get
flutter run
```

### 8. Seed Test Data

After the app loads, use the menu (⋮) > "Seed Dummy Units" to create 5 test charging units around Warsaw.

---

## Architecture Decisions

### Why PostGIS?

- **Accuracy**: Uses geodesic calculations (accounts for Earth's curvature)
- **Performance**: Spatial indexes for fast radius queries
- **Standard**: Industry-standard for geospatial applications

### Why Serverpod?

- **Type Safety**: Dart-to-Dart with generated client
- **Productivity**: Auto-generated models and database access
- **Scalability**: Built-in support for clustering and caching

### Why Mapbox?

- **Customization**: Full control over map styling
- **Performance**: Optimized for mobile devices
- **Features**: Built-in annotations, gestures, offline support

---

## Phase 2 Considerations

Future enhancements:
- Real-time unit tracking (WebSocket streams)
- Driver companion app
- Push notifications
- Payment integration
- Route optimization
- Battery level tracking
- Estimated arrival times

---

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `SERVER_URL` | Serverpod server URL | `http://localhost:8080/` |
| `MAPBOX_ACCESS_TOKEN` | Mapbox public token | (required) |

---

## Troubleshooting

### "No available charging units found"
1. Ensure database is running: `docker-compose ps`
2. Run migrations: `dart bin/main.dart --apply-migrations`
3. Seed test data via app menu

### PostGIS functions not found
1. Verify PostGIS image: `postgis/postgis:14-3.3`
2. Run: `CREATE EXTENSION IF NOT EXISTS postgis;`

### Flutter build errors
1. Run: `flutter pub get`
2. Generate code: `cd ../eco_volt_charge_server && serverpod generate`
