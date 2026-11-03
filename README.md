# EcoVolt Charge (EVC)

**A Flutter-powered EV charging platform built with Dart and Serverpod.**

EcoVolt Charge simplifies access to electric vehicle charging services through vehicle management, location selection, charging preferences, and backend-powered workflows.

## 🚀 Live Application

- **Google Play:** [Download EcoVolt Charge](https://play.google.com/store/apps/details?id=com.ecovoltcharge.evuser)
- **GitHub:** [View Source Code](https://github.com/Tee-Kay404/ECOVOLT-CHARGE-)

## ✨ Key Features

- User authentication and account management.
- EV vehicle selection and management.
- Charging configuration (full, half, and custom).
- Location selection and saved locations.
- Dashboard, activity tracking, and profile management.
- Backend API integration and charging workflows.

## 🛠️ Technology Stack

- **Frontend:** Flutter, Dart
- **Backend:** Serverpod, Dart
- **State Management:** Provider, ChangeNotifier
- **Database:** PostgreSQL
- **Infrastructure:** Docker, Redis, PostGIS
- **Monorepo Management:** Melos
- **CI/CD:** GitHub Actions

## 🏗️ Architecture

The project follows a modular monorepo architecture:

```text
ECOVOLT-CHARGE/
├── apps/
│   ├── evc_user/
│   └── evc_driver/
├── evc_client/
├── evc_server/
├── evc_shared/
├── .github/workflows/
├── ARCHITECTURE.md
├── melos.yaml
└── pubspec.yaml
```

The architecture separates frontend applications, backend services, shared models, and generated API clients for maintainability and code reusability.

## ⚙️ Engineering Decisions

- **Monorepo:** Centralized package and application management.
- **Provider:** Reactive state management using ChangeNotifier.
- **Serverpod:** Structured backend endpoints and type-safe client-server communication.
- **Melos:** Workspace and dependency management.
- **Modular Architecture:** Separation of responsibilities for maintainability and scalability.

## 🚀 Getting Started

**Prerequisites:** Flutter, Dart, Docker, Melos, and Serverpod CLI.

```bash
git clone https://github.com/Tee-Kay404/ECOVOLT-CHARGE-.git
cd ECOVOLT-CHARGE-
dart pub global activate melos
melos bootstrap
melos run generate
```

Configure the required environment variables and backend services, then run the application:

```bash
cd apps/evc_user
flutter run
```

## 🧪 CI/CD

GitHub Actions provides automated workflows for code formatting, static analysis, and testing.

## 👨‍💻 Contributions & Responsibilities

**Role: Lead Mobile Developer / Full-Stack Engineer**

- Contributed to technical direction and architectural decisions.
- Developed Flutter application features and reusable components.
- Implemented state management using Provider.
- Worked on authentication, vehicle management, location services, and charging workflows.
- Integrated frontend features with backend APIs and Serverpod.
- Contributed to the Google Play release and deployment process.

## 🎯 Project Objective

To improve the digital EV charging experience through accessible charging services, intuitive mobile experiences, and maintainable backend architecture.
