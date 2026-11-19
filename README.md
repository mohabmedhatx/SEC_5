# Section 5 Assignment — Hive Local NoSQL Storage (Offline Posts & Favorites App)

| Field | Details |
| :--- | :--- |
| **Student Name** | Mohab Medhat Ali |
| **Student ID** | 23010016 |
| **Faculty** | Faculty of Computers and Data Science, Alexandria University |
| **Term** | Fall 2026 — Section 5 (Hive Local NoSQL Storage) |

---

## Overview

This Flutter application fetches posts from [`https://jsonplaceholder.typicode.com/posts`](https://jsonplaceholder.typicode.com/posts), caches them locally in a **Hive** box (`postsBox`) for full offline availability, and allows users to mark posts as favorites inside a dedicated persistent Hive box (`favoritesBox`) that survives pull-to-refresh, app restarts, and Airplane Mode.

### Architecture & Key Implementation Details

1. **Dedicated Hive Boxes (`lib/main.dart`)**:
   - Initializes Hive (`Hive.initFlutter()`), registers `PostAdapter`, and pre-opens both `postsBox` and `favoritesBox`.
   - Keeping favorites in a separate box prevents `postsBox.clear()` during pull-to-refresh from wiping saved favorites.
2. **Offline-First Posts Service (`lib/services/post_service.dart`)**:
   - Attempts to fetch live posts from the API, clears `postsBox`, and caches the refreshed list.
   - On connectivity loss (e.g., Airplane Mode), gracefully falls back to `postsBox.values.toList()` or throws `"No Internet and No Cached Data"` if the cache is empty.
3. **Favorites Service (`lib/services/favorites_service.dart`)**:
   - `toggleFavorite(Post post)`: Uses `post.id` as the key (`box.put(post.id, copy)` / `box.delete(post.id)`). Saves a fresh `Post` copy because a `HiveObject` instance can only live in one Hive box at a time.
   - `isFavorite(int id)`: Checks `box.containsKey(id)` synchronously.
   - `getFavorites()`: Reads `box.values.toList()` synchronously from the already opened box.
4. **Reactive Posts Screen (`lib/screens/posts_screen.dart`)**:
   - Wraps the post list with `ValueListenableBuilder` listening to `favoritesBox.listenable()` so tapping a star switches immediately between a filled amber star (`Icons.star`) and an outlined star (`Icons.star_border`).
   - Adds a Favorites action icon in the `AppBar` to navigate to `FavoritesScreen`.
5. **Favorites Screen (`lib/screens/favorites_screen.dart`)**:
   - Displays all favorited posts using the same `Card` + `ListTile` design as `PostsScreen`.
   - Allows removing favorites directly with instant UI synchronization back to `PostsScreen`.
   - Displays a friendly **"No favorites yet"** empty state when `favoritesBox` is empty.

---

## Project Structure

```text
.
├── pubspec.yaml
├── README.md
├── screenshots/
│   ├── 01_posts_screen.png
│   ├── 02_favorites_screen.png
│   ├── 03_favorites_empty.png
│   ├── 04_offline_posts_airplane.png
│   ├── 05_offline_favorites_airplane.png
│   └── 06_offline_console_output.png
└── lib/
    ├── main.dart
    ├── models/
    │   ├── post_model.dart
    │   └── post_model.g.dart
    ├── services/
    │   ├── post_service.dart
    │   └── favorites_service.dart
    └── screens/
        ├── posts_screen.dart
        └── favorites_screen.dart
```

---

## Feature & Action Screenshots

### 1. Online Posts List, Star Toggle & Favorites Screen

| Posts Screen (Online & Favoriting) | Favorites Screen (Saved Posts) | Favorites Screen (Empty State) |
| :---: | :---: | :---: |
| ![Posts Screen](screenshots/01_posts_screen.png) | ![Favorites Screen](screenshots/02_favorites_screen.png) | ![Favorites Empty State](screenshots/03_favorites_empty.png) |
| Posts fetched from API and cached in `postsBox`; posts **1, 3, 5** toggled as favorites (`Icons.star`), others show `Icons.star_border`. | Opened via the `AppBar` star button; lists saved posts from `favoritesBox` using the same card layout. | Displayed cleanly when all favorites are removed or no posts have been starred yet. |

---

### 2. Offline / Airplane Mode Verification

After favoriting posts **1, 3, and 5**, closing the app completely, enabling **Airplane Mode**, and reopening the app:

| Posts Screen in Airplane Mode | Favorites Screen in Airplane Mode |
| :---: | :---: |
| ![Offline Posts](screenshots/04_offline_posts_airplane.png) | ![Offline Favorites](screenshots/05_offline_favorites_airplane.png) |
| Cached posts load immediately from `postsBox` with Airplane Mode active and starred states preserved. | All saved favorites remain intact offline in `favoritesBox` across app restarts. |

### 3. Debug Console Log (Online Cache & Offline Fallback)

![Offline Console Log](screenshots/06_offline_console_output.png)

---

## How to Run Locally

```bash
flutter pub get
dart run build_runner build --delete-conflicting-outputs
flutter run
```
