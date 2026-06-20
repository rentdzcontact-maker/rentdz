# RentDz - Flutter Architecture & Technical Specification

Since you've requested to act as an expert Flutter developer to design the architecture for "RentDz" (a real estate platform for the Algerian market), here is a complete, production-ready architecture blueprint.

## 1. Clean Architecture Folder Structure

We use a feature-first **Clean Architecture** combined with **BLoC** for state management. This ensures scalability, separation of concerns, and ease of testing.

```text
lib/
│
├── core/                       # App-wide configurations and utilities
│   ├── constants/              # App colors, styles, and dimensions
│   ├── error/                  # Failure and Exception classes
│   ├── network/                # HTTP client (Dio interceptors, etc.)
│   ├── router/                 # GoRouter configuration
│   └── localization/           # l10n translations (AR, FR, EN)
│
├── features/                   # Feature-based divisions
│   ├── auth/                   # Authentication (Landlord/Tenant)
│   │   ├── domain/             # Entities, Repositories (interfaces), UseCases
│   │   ├── data/               # Models, Repositories (impl), DataSources
│   │   └── presentation/       # BLoCs, Pages, Widgets
│   │
│   ├── property_listings/      # Exploring and filtering properties
│   │   ├── domain/
│   │   ├── data/
│   │   └── presentation/       # Property Catalog, Filters, Cards
│   │
│   └── property_details/       # Single property view & booking request
│       ├── domain/
│       ├── data/
│       └── presentation/       # Detail Page, Carousel, Booking Form
│
└── main.dart                   # Entry point and Dependency Injection (get_it setup)
```

## 2. Dependencies (`pubspec.yaml`)

```yaml
name: rentdz
description: A modern, high-performance real estate rental platform for Algeria.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: '>=3.2.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
    
  # State Management & Dependency Injection
  flutter_bloc: ^8.1.3
  get_it: ^7.6.4
  equatable: ^2.0.5

  # Routing (Deep linking support for Web)
  go_router: ^12.1.1

  # Networking & Data
  dio: ^5.3.3
  shared_preferences: ^2.2.2
  cached_network_image: ^3.3.0

  # UI & Styling
  responsive_builder: ^0.7.0    # For Web/Mobile breakpoint management
  google_fonts: ^6.1.0          # Inter & Playfair Display fonts
  carousel_slider: ^4.2.1       # For property image carousels
  flutter_svg: ^2.0.9           # For crisp vector icons

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.6
  json_serializable: ^6.7.1
  mocktail: ^1.0.0
```

## 3. Clean Code Examples (Responsive)

### A. Responsive Property Card Widget

This widget automatically scales appropriately whether rendered on a Mobile app or a Desktop browser.

```dart
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';

class PropertyCard extends StatelessWidget {
  final String title;
  final String location;
  final String propertyType;
  final String priceDzd;
  final String imageUrl;

  const PropertyCard({
    Key? key,
    required this.title,
    required this.location,
    required this.propertyType,
    required this.priceDzd,
    required this.imageUrl,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    // Determine card width constraint based on platform/screen bounds
    return Container(
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(16),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.04),
            blurRadius: 20,
            offset: const Offset(0, 10),
          )
        ],
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Image Section
          Stack(
            children: [
              ClipRRect(
                borderRadius: BorderRadius.circular(16),
                child: AspectRatio(
                  aspectRatio: 4 / 3,
                  child: Image.network(
                    imageUrl,
                    fit: BoxFit.cover,
                    errorBuilder: (context, error, stackTrace) => Container(
                      color: const Color(0xFFE5E2DC),
                    ),
                  ),
                ),
              ),
              Positioned(
                top: 16,
                left: 16,
                child: Container(
                  padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
                  decoration: BoxDecoration(
                    color: Colors.white.withOpacity(0.9),
                    borderRadius: BorderRadius.circular(20),
                  ),
                  child: Text(
                    propertyType.toUpperCase(),
                    style: const TextStyle(
                      fontSize: 10,
                      fontWeight: FontWeight.bold,
                      letterSpacing: 1.2,
                    ),
                  ),
                ),
              ),
            ],
          ),
          const SizedBox(height: 12),
          // Info Section
          Padding(
             padding: const EdgeInsets.symmetric(horizontal: 4.0),
             child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        title,
                        style: GoogleFonts.inter(
                          fontSize: 16,
                          fontWeight: FontWeight.w600,
                        ),
                        maxLines: 1,
                        overflow: TextOverflow.ellipsis,
                      ),
                      const SizedBox(height: 4),
                      Text(
                        location,
                        style: GoogleFonts.inter(
                          fontSize: 12,
                          color: Colors.black54,
                        ),
                      ),
                    ],
                  ),
                ),
                Column(
                  crossAxisAlignment: CrossAxisAlignment.end,
                  children: [
                    Text(
                      priceDzd,
                      style: GoogleFonts.playfairDisplay(
                        fontSize: 18,
                        fontWeight: FontWeight.w600,
                      ),
                    ),
                    Text(
                      'DZD / MONTH',
                      style: GoogleFonts.inter(
                        fontSize: 10,
                        fontWeight: FontWeight.bold,
                        color: Colors.black45,
                      ),
                    ),
                  ],
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

### B. Responsive Home Screen Layout

We utilize `responsive_builder` or simple `LayoutBuilder` breakpoints to adjust the grid axis count.

```dart
import 'package:flutter/material.dart';
import 'package:responsive_builder/responsive_builder.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF9F8F6),
      // App Bar code omitted for brevity
      body: SingleChildScrollView(
        child: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 24.0, vertical: 32.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
               // Hero Title
               Text(
                'Featured Rentals in Algiers',
                style: GoogleFonts.playfairDisplay(
                  fontSize: 28,
                  fontWeight: FontWeight.w500,
                ),
               ),
               const SizedBox(height: 24),
               
               // Responsive Grid
               ResponsiveBuilder(
                 builder: (context, sizingInformation) {
                   int crossAxisCount = 1; // Mobile
                   
                   if (sizingInformation.deviceScreenType == DeviceScreenType.tablet) {
                     crossAxisCount = 2;
                   } else if (sizingInformation.deviceScreenType == DeviceScreenType.desktop) {
                     crossAxisCount = 3;
                   }

                   return GridView.builder(
                     shrinkWrap: true,
                     physics: const NeverScrollableScrollPhysics(),
                     gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                       crossAxisCount: crossAxisCount,
                       crossAxisSpacing: 24,
                       mainAxisSpacing: 24,
                       childAspectRatio: 0.85, // Adjust based on card height
                     ),
                     itemCount: 6,
                     itemBuilder: (context, index) {
                       return const PropertyCard(
                         title: 'The Residence Elite',
                         location: 'Sidi Yahia, Hydra',
                         propertyType: 'Apartment',
                         priceDzd: '140,000',
                         imageUrl: 'https://placeholder.com/600x400',
                       );
                     },
                   );
                 },
               ),
            ],
          ),
        ),
      ),
    );
  }
}
```
