# Flutter Architecture Guide

Guía de arquitectura para aplicaciones Flutter basada en Clean Architecture y mejores prácticas.

## Índice

- [Estructura de proyecto](#estructura-de-proyecto)
- [Clean Architecture por features](#clean-architecture-por-features)
- [Capas y responsabilidades](#capas-y-responsabilidades)
- [Gestión de estado con Riverpod](#gestión-de-estado-con-riverpod)
- [Navegación con GoRouter](#navegación-con-gorouter)
- [Integración con Supabase](#integración-con-supabase)
- [Manejo de variables de entorno](#manejo-de-variables-de-entorno)
- [Internacionalización](#internacionalización)
- [Dependencias entre capas](#dependencias-entre-capas)
- [Convenciones de código](#convenciones-de-código)

---

## Estructura de proyecto

Las aplicaciones Flutter siguen esta estructura recomendada:

```
my_app/
├── lib/
│   ├── main.dart
│   ├── app.dart
│   ├── core/
│   │   ├── config/
│   │   │   ├── env.dart              # Variables de entorno
│   │   │   └── router.dart           # Configuración de navegación
│   │   ├── constants/                # Constantes de la aplicación
│   │   ├── errors/
│   │   │   └── failures.dart         # Clases de error del dominio
│   │   ├── network/
│   │   │   └── api_client.dart       # Cliente HTTP (Dio, http, etc)
│   │   └── utils/                    # Utilidades generales
│   ├── features/
│   │   ├── authentication/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   ├── feature_a/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   └── feature_b/
│   │       ├── data/
│   │       ├── domain/
│   │       └── presentation/
│   └── shared/
│       ├── widgets/              # Widgets reutilizables de la app
│       └── providers/            # Providers globales
├── .env                          # NO COMMITEAR (en .gitignore)
├── .env.example                  # Plantilla con claves sin valores
└── pubspec.yaml
```

**Nota:** Cada `feature` representa una funcionalidad completa del dominio de tu aplicación.

---

## Clean Architecture por features

Cada **feature** es una funcionalidad autocontenida del dominio (ej: `authentication`, `products`, `profile`, `shopping_cart`).

### Estructura de un feature

```
features/example_feature/
├── data/
│   ├── datasources/
│   │   └── example_remote_datasource.dart
│   ├── models/
│   │   └── example_model.dart
│   └── repositories/
│       └── example_repository_impl.dart
├── domain/
│   ├── entities/
│   │   └── example_entity.dart
│   ├── repositories/
│   │   └── example_repository.dart
│   └── usecases/
│       ├── get_example_usecase.dart
│       └── create_example_usecase.dart
└── presentation/
    ├── providers/
    │   └── example_provider.dart
    ├── widgets/
    │   └── example_widget.dart
    └── pages/
        └── example_page.dart
```

---

## Capas y responsabilidades

### 1. Domain (Núcleo de negocio)

**NO depende de ninguna capa externa.** Define las reglas de negocio puras.

#### **Entities** (`domain/entities/`)

Objetos de negocio puros. Sin anotaciones JSON, sin lógica de persistencia.

```dart
// features/products/domain/entities/product.dart
class Product {
  final String id;
  final String name;
  final double price;
  final String? imageUrl;
  final String description;

  const Product({
    required this.id,
    required this.name,
    required this.price,
    this.imageUrl,
    required this.description,
  });

  bool get isExpensive => price > 100.0;
}
```

#### **Repository Interfaces** (`domain/repositories/`)

Contratos abstractos. Las implementaciones están en `data/`.

```dart
// features/products/domain/repositories/product_repository.dart
import 'package:dartz/dartz.dart';
import '../entities/product.dart';
import '../../../../core/errors/failures.dart';

abstract class ProductRepository {
  Future<Either<Failure, List<Product>>> getProducts();
  Future<Either<Failure, Product>> getProductById(String productId);
  Future<Either<Failure, void>> createProduct(Product product);
}
```

**Nota:** Usamos `Either` de `dartz` para manejo de errores funcional (ver sección [Manejo de errores](#manejo-de-errores)).

#### **Use Cases** (`domain/usecases/`)

Casos de uso de la aplicación. Un caso de uso = una acción del usuario.

```dart
// features/products/domain/usecases/get_products.dart
import 'package:dartz/dartz.dart';
import '../entities/product.dart';
import '../repositories/product_repository.dart';
import '../../../../core/errors/failures.dart';

class GetProducts {
  final ProductRepository repository;

  GetProducts(this.repository);

  Future<Either<Failure, List<Product>>> call() {
    return repository.getProducts();
  }
}
```

---

### 2. Data (Implementación de infraestructura)

Implementa las interfaces definidas en `domain/`.

#### **Models / DTOs** (`data/models/`)

Objetos de transferencia de datos con serialización JSON. Extienden o mapean a entities.

```dart
// features/products/data/models/product_model.dart
import '../../domain/entities/product.dart';

class ProductModel {
  final String id;
  final String name;
  final double price;
  final String? imageUrl;
  final String description;

  ProductModel({
    required this.id,
    required this.name,
    required this.price,
    this.imageUrl,
    required this.description,
  });

  factory ProductModel.fromJson(Map<String, dynamic> json) {
    return ProductModel(
      id: json['id'] as String,
      name: json['name'] as String,
      price: (json['price'] as num).toDouble(),
      imageUrl: json['image_url'] as String?,
      description: json['description'] as String,
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'price': price,
      'image_url': imageUrl,
      'description': description,
    };
  }

  Product toEntity() {
    return Product(
      id: id,
      name: name,
      price: price,
      imageUrl: imageUrl,
      description: description,
    );
  }
}
```

**Importante:** Los DTOs NUNCA se exponen fuera de la capa `data`. Siempre se convierten a entities.

#### **Datasources** (`data/datasources/`)

Encapsulan el acceso a datos externos (API, base de datos local, etc.).

```dart
// features/products/data/datasources/product_remote_datasource.dart
import 'package:dio/dio.dart';
import '../models/product_model.dart';

abstract class ProductRemoteDataSource {
  Future<List<ProductModel>> getProducts();
  Future<ProductModel> getProductById(String productId);
}

class ProductRemoteDataSourceImpl implements ProductRemoteDataSource {
  final Dio dio;

  ProductRemoteDataSourceImpl(this.dio);

  @override
  Future<List<ProductModel>> getProducts() async {
    try {
      final response = await dio.get('/api/products');

      final List<dynamic> data = response.data as List;
      return data.map((json) => ProductModel.fromJson(json)).toList();
    } on DioException catch (e) {
      throw _handleDioError(e);
    }
  }

  @override
  Future<ProductModel> getProductById(String productId) async {
    try {
      final response = await dio.get('/api/products/$productId');

      return ProductModel.fromJson(response.data);
    } on DioException catch (e) {
      throw _handleDioError(e);
    }
  }

  Exception _handleDioError(DioException e) {
    // Transformar DioException en excepciones del dominio
    if (e.response?.statusCode == 404) {
      return Exception('Resource not found');
    }
    if (e.response?.statusCode == 401) {
      return Exception('Unauthorized');
    }
    return Exception('Network error: ${e.message}');
  }
}
```

**Nota:** Este ejemplo usa Dio como cliente HTTP, pero puedes usar cualquier paquete HTTP (http, retrofit, etc.).

#### **Repository Implementations** (`data/repositories/`)

Implementan las interfaces de `domain/repositories/`. Coordinan datasources y convierten DTOs a entities.

```dart
// features/products/data/repositories/product_repository_impl.dart
import 'package:dartz/dartz.dart';
import '../../domain/entities/product.dart';
import '../../domain/repositories/product_repository.dart';
import '../datasources/product_remote_datasource.dart';
import '../../../../core/errors/failures.dart';

class ProductRepositoryImpl implements ProductRepository {
  final ProductRemoteDataSource remoteDataSource;

  ProductRepositoryImpl(this.remoteDataSource);

  @override
  Future<Either<Failure, List<Product>>> getProducts() async {
    try {
      final models = await remoteDataSource.getProducts();
      final entities = models.map((model) => model.toEntity()).toList();
      return Right(entities);
    } catch (e) {
      return Left(ServerFailure(message: e.toString()));
    }
  }

  @override
  Future<Either<Failure, Product>> getProductById(String productId) async {
    try {
      final model = await remoteDataSource.getProductById(productId);
      return Right(model.toEntity());
    } catch (e) {
      return Left(ServerFailure(message: e.toString()));
    }
  }

  @override
  Future<Either<Failure, void>> createProduct(Product product) async {
    try {
      // Implementación de creación de producto
      return const Right(null);
    } catch (e) {
      return Left(ServerFailure(message: e.toString()));
    }
  }
}
```

---

### 3. Presentation (UI + Estado)

Contiene la lógica de presentación y los widgets.

#### **Providers** (`presentation/providers/`)

Estado gestionado con Riverpod. Ver [Gestión de estado](#gestión-de-estado-con-riverpod).

```dart
// features/products/presentation/providers/products_provider.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../domain/entities/product.dart';
import '../../domain/usecases/get_products.dart';
import '../../data/repositories/product_repository_impl.dart';
import '../../data/datasources/product_remote_datasource.dart';
import '../../../../core/network/api_client.dart';

// Provider del datasource
final productRemoteDataSourceProvider = Provider<ProductRemoteDataSource>((ref) {
  final dio = ref.watch(apiClientProvider);
  return ProductRemoteDataSourceImpl(dio);
});

// Provider del repositorio
final productRepositoryProvider = Provider((ref) {
  final dataSource = ref.watch(productRemoteDataSourceProvider);
  return ProductRepositoryImpl(dataSource);
});

// Provider del caso de uso
final getProductsUseCaseProvider = Provider((ref) {
  final repository = ref.watch(productRepositoryProvider);
  return GetProducts(repository);
});

// Provider de estado (FutureProvider para lista de productos)
final productsListProvider = FutureProvider<List<Product>>((ref) async {
  final useCase = ref.watch(getProductsUseCaseProvider);
  final result = await useCase();

  return result.fold(
    (failure) => throw Exception(failure.message),
    (products) => products,
  );
});
```

#### **Widgets** (`presentation/widgets/`)

Componentes reutilizables dentro del feature.

```dart
// features/products/presentation/widgets/product_card.dart
import 'package:flutter/material.dart';
import '../../domain/entities/product.dart';

class ProductCard extends StatelessWidget {
  final Product product;
  final VoidCallback onTap;

  const ProductCard({
    Key? key,
    required this.product,
    required this.onTap,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Card(
      clipBehavior: Clip.antiAlias,
      child: InkWell(
        onTap: onTap,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            if (product.imageUrl != null)
              Image.network(
                product.imageUrl!,
                height: 150,
                width: double.infinity,
                fit: BoxFit.cover,
              ),
            Padding(
              padding: const EdgeInsets.all(12.0),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    product.name,
                    style: Theme.of(context).textTheme.titleMedium,
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const SizedBox(height: 4),
                  Text(
                    '\$${product.price.toStringAsFixed(2)}',
                    style: Theme.of(context).textTheme.bodyLarge?.copyWith(
                      fontWeight: FontWeight.bold,
                      color: Theme.of(context).colorScheme.primary,
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### **Pages** (`presentation/pages/`)

Páginas completas que se registran en el router.

```dart
// features/products/presentation/pages/products_list_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/products_provider.dart';
import '../widgets/product_card.dart';
import 'package:go_router/go_router.dart';

class ProductsListPage extends ConsumerWidget {
  const ProductsListPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsListProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Products'),
      ),
      body: productsAsync.when(
        data: (products) {
          if (products.isEmpty) {
            return const Center(
              child: Text('No products available'),
            );
          }

          return GridView.builder(
            padding: const EdgeInsets.all(16.0),
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2,
              childAspectRatio: 0.75,
              crossAxisSpacing: 16.0,
              mainAxisSpacing: 16.0,
            ),
            itemCount: products.length,
            itemBuilder: (context, index) {
              final product = products[index];
              return ProductCard(
                product: product,
                onTap: () {
                  context.push('/products/${product.id}');
                },
              );
            },
          );
        },
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (error, stack) => Center(
          child: Text('Error: $error'),
        ),
      ),
    );
  }
}
```

---

## Gestión de estado con Riverpod

**Riverpod** es el sistema de gestión de estado e inyección de dependencias.

### Tipos de providers

#### **Provider** - Valores inmutables o instancias

```dart
final apiClientProvider = Provider<Dio>((ref) {
  final dio = Dio(BaseOptions(
    baseUrl: Env.apiBaseUrl,
    headers: {
      'Content-Type': 'application/json',
    },
    connectTimeout: const Duration(seconds: 10),
    receiveTimeout: const Duration(seconds: 10),
  ));
  return dio;
});
```

#### **FutureProvider** - Valores asíncronos de solo lectura

```dart
final userProfileProvider = FutureProvider.autoDispose<UserProfile>((ref) async {
  final repository = ref.watch(authRepositoryProvider);
  final result = await repository.getCurrentUser();

  return result.fold(
    (failure) => throw Exception(failure.message),
    (user) => user,
  );
});
```

#### **StateNotifierProvider** - Estado mutable complejo

```dart
// State class
class AuthState {
  final bool isAuthenticated;
  final String? userId;
  final bool isLoading;

  const AuthState({
    this.isAuthenticated = false,
    this.userId,
    this.isLoading = false,
  });

  AuthState copyWith({
    bool? isAuthenticated,
    String? userId,
    bool? isLoading,
  }) {
    return AuthState(
      isAuthenticated: isAuthenticated ?? this.isAuthenticated,
      userId: userId ?? this.userId,
      isLoading: isLoading ?? this.isLoading,
    );
  }
}

// StateNotifier
class AuthNotifier extends StateNotifier<AuthState> {
  final LoginUseCase loginUseCase;
  final LogoutUseCase logoutUseCase;

  AuthNotifier({
    required this.loginUseCase,
    required this.logoutUseCase,
  }) : super(const AuthState());

  Future<void> login(String email, String password) async {
    state = state.copyWith(isLoading: true);

    final result = await loginUseCase(email, password);

    result.fold(
      (failure) {
        state = state.copyWith(isLoading: false);
        // Handle error (mostrar SnackBar, etc.)
      },
      (user) {
        state = state.copyWith(
          isAuthenticated: true,
          userId: user.id,
          isLoading: false,
        );
      },
    );
  }

  Future<void> logout() async {
    await logoutUseCase();
    state = const AuthState();
  }
}

// Provider
final authProvider = StateNotifierProvider<AuthNotifier, AuthState>((ref) {
  final loginUseCase = ref.watch(loginUseCaseProvider);
  final logoutUseCase = ref.watch(logoutUseCaseProvider);

  return AuthNotifier(
    loginUseCase: loginUseCase,
    logoutUseCase: logoutUseCase,
  );
});
```

### Consumir providers en widgets

**ConsumerWidget** para widgets sin estado:

```dart
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authState = ref.watch(authProvider);

    return Text(authState.isAuthenticated ? 'Autenticado' : 'No autenticado');
  }
}
```

**ConsumerStatefulWidget** para widgets con estado:

```dart
class MyStatefulWidget extends ConsumerStatefulWidget {
  @override
  ConsumerState<MyStatefulWidget> createState() => _MyStatefulWidgetState();
}

class _MyStatefulWidgetState extends ConsumerState<MyStatefulWidget> {
  @override
  Widget build(BuildContext context) {
    final authState = ref.watch(authProvider);

    return ElevatedButton(
      onPressed: () {
        ref.read(authProvider.notifier).logout();
      },
      child: const Text('Cerrar sesión'),
    );
  }
}
```

### Modifiers útiles

- `.autoDispose` - Limpia el provider cuando no hay listeners
- `.family` - Parametriza el provider (ej: `childrenListProvider(parentId)`)

---

## Navegación con GoRouter

**GoRouter** gestiona todas las rutas de la aplicación.

### Configuración central

```dart
// lib/core/config/router.dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../features/authentication/presentation/pages/login_page.dart';
import '../../features/products/presentation/pages/products_list_page.dart';
import '../../features/products/presentation/pages/product_detail_page.dart';
import '../../features/authentication/presentation/providers/auth_provider.dart';

final routerProvider = Provider<GoRouter>((ref) {
  final authState = ref.watch(authProvider);

  return GoRouter(
    initialLocation: '/login',
    redirect: (context, state) {
      final isAuthenticated = authState.isAuthenticated;
      final isLoginRoute = state.matchedLocation == '/login';

      // Redirigir a login si no está autenticado
      if (!isAuthenticated && !isLoginRoute) {
        return '/login';
      }

      // Redirigir a home si está autenticado e intenta ir a login
      if (isAuthenticated && isLoginRoute) {
        return '/';
      }

      return null; // No redirigir
    },
    routes: [
      GoRoute(
        path: '/login',
        builder: (context, state) => const LoginPage(),
      ),
      GoRoute(
        path: '/',
        builder: (context, state) => const ProductsListPage(),
        routes: [
          GoRoute(
            path: 'products/:productId',
            builder: (context, state) {
              final productId = state.pathParameters['productId']!;
              return ProductDetailPage(productId: productId);
            },
          ),
        ],
      ),
    ],
  );
});
```

### Navegación en widgets

```dart
// Navegar a una ruta
context.push('/products/123');

// Navegar reemplazando la ruta actual
context.go('/');

// Volver atrás
context.pop();

// Navegar con parámetros de query
context.push('/search?q=laptop');
```

### Integración con MaterialApp

```dart
// lib/app.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'core/config/router.dart';

class App extends ConsumerWidget {
  const App({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(routerProvider);

    return MaterialApp.router(
      title: 'My App',
      routerConfig: router,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      darkTheme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.blue,
          brightness: Brightness.dark,
        ),
        useMaterial3: true,
      ),
    );
  }
}
```

---

## Integración con Backend

Esta arquitectura es agnóstica al backend. Puedes usar cualquier API REST, GraphQL, Firebase, Supabase, etc.

### Configuración de cliente HTTP con Dio

```dart
// lib/core/network/api_client.dart
import 'package:dio/dio.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../config/env.dart';

final apiClientProvider = Provider<Dio>((ref) {
  final dio = Dio(
    BaseOptions(
      baseUrl: Env.apiBaseUrl,
      headers: {
        'Content-Type': 'application/json',
      },
      connectTimeout: const Duration(seconds: 10),
      receiveTimeout: const Duration(seconds: 10),
    ),
  );

  // Interceptor para añadir token de autenticación
  dio.interceptors.add(InterceptorsWrapper(
    onRequest: (options, handler) async {
      // Si hay un token de sesión, añadirlo
      final token = await _getSessionToken();
      if (token != null) {
        options.headers['Authorization'] = 'Bearer $token';
      }
      handler.next(options);
    },
    onError: (error, handler) {
      // Manejo global de errores
      if (error.response?.statusCode == 401) {
        // Token expirado, cerrar sesión
        // ref.read(authProvider.notifier).logout();
      }
      handler.next(error);
    },
  ));

  // Logging en desarrollo
  if (Env.isDevelopment) {
    dio.interceptors.add(LogInterceptor(
      requestBody: true,
      responseBody: true,
    ));
  }

  return dio;
});

Future<String?> _getSessionToken() async {
  // Obtener token desde tu sistema de autenticación
  // (SharedPreferences, SecureStorage, etc.)
  return null;
}
```

### Ejemplo: Datasource de autenticación

```dart
// features/authentication/data/datasources/auth_remote_datasource.dart
import 'package:dio/dio.dart';
import '../models/user_model.dart';

abstract class AuthRemoteDataSource {
  Future<UserModel> login(String email, String password);
  Future<void> logout();
  Future<UserModel?> getCurrentUser();
}

class AuthRemoteDataSourceImpl implements AuthRemoteDataSource {
  final Dio dio;

  AuthRemoteDataSourceImpl(this.dio);

  @override
  Future<UserModel> login(String email, String password) async {
    final response = await dio.post(
      '/api/auth/login',
      data: {
        'email': email,
        'password': password,
      },
    );

    if (response.statusCode == 200) {
      return UserModel.fromJson(response.data);
    }

    throw Exception('Login failed');
  }

  @override
  Future<void> logout() async {
    await dio.post('/api/auth/logout');
  }

  @override
  Future<UserModel?> getCurrentUser() async {
    final response = await dio.get('/api/auth/me');

    if (response.statusCode == 200) {
      return UserModel.fromJson(response.data);
    }

    return null;
  }
}
```

### Otras opciones de backend

#### Firebase

```yaml
# pubspec.yaml
dependencies:
  firebase_core: ^latest
  firebase_auth: ^latest
  cloud_firestore: ^latest
```

#### GraphQL

```yaml
# pubspec.yaml
dependencies:
  graphql_flutter: ^latest
```

#### REST con http package

```yaml
# pubspec.yaml
dependencies:
  http: ^latest
```

---

## Manejo de variables de entorno

**NUNCA commitear credenciales.** Usar archivos `.env` con `.gitignore`.

### Estructura de archivos

```
my_app/
├── .env                  # NO COMMITEAR (gitignored)
├── .env.example          # SÍ commitear (plantilla sin valores)
└── lib/
    └── core/
        └── config/
            └── env.dart  # Clase que lee las variables
```

### .env.example (commitear)

```bash
# .env.example
API_BASE_URL=https://api.example.com
API_KEY=your-api-key-here
```

### .env (NO commitear)

```bash
# .env
API_BASE_URL=https://api.myapp.com
API_KEY=sk_live_51ABC123...
```

### Configuración en pubspec.yaml

```yaml
dependencies:
  flutter_dotenv: ^5.1.0
```

### Cargar variables al iniciar la app

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'app.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Cargar variables de entorno
  await dotenv.load(fileName: '.env');

  // Inicializar otros servicios si es necesario
  // await initializeServices();

  runApp(
    const ProviderScope(
      child: App(),
    ),
  );
}
```

### Clase Env para acceder a variables

```dart
// lib/core/config/env.dart
import 'package:flutter_dotenv/flutter_dotenv.dart';

class Env {
  static String get apiBaseUrl => dotenv.env['API_BASE_URL']!;
  static String get apiKey => dotenv.env['API_KEY']!;

  static bool get isDevelopment => const bool.fromEnvironment('dart.vm.product') == false;
}
```

### .gitignore

Asegurarse de que `.env` esté en `.gitignore`:

```gitignore
# Secrets
.env
*.env
!.env.example
```

---

## Internacionalización

### Configuración básica

```dart
// lib/app.dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';

MaterialApp.router(
  locale: const Locale('en'),
  localizationsDelegates: const [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  supportedLocales: const [
    Locale('en'),
    Locale('es'),
  ],
  // ...
)
```

### Opciones de internacionalización

#### Opción 1: Strings directamente en código (proyectos pequeños)

```dart
Text('Hello World')
```

#### Opción 2: Usar `flutter_gen` o `intl` (proyectos medianos/grandes)

```yaml
# pubspec.yaml
dependencies:
  flutter_localizations:
    sdk: flutter
  intl: ^latest

dev_dependencies:
  flutter_gen: ^latest
```

**Recomendación:** Para aplicaciones con múltiples idiomas, usa el paquete oficial `flutter_localizations` con archivos ARB.

---

## Manejo de errores

### Clases de Failure en el dominio

```dart
// lib/core/errors/failures.dart
abstract class Failure {
  final String message;
  const Failure({required this.message});
}

class ServerFailure extends Failure {
  const ServerFailure({required super.message});
}

class CacheFailure extends Failure {
  const CacheFailure({required super.message});
}

class ValidationFailure extends Failure {
  const ValidationFailure({required super.message});
}
```

### Usar Either (dartz)

Todas las operaciones que pueden fallar devuelven `Either<Failure, T>`:

```dart
Future<Either<Failure, User>> login(String email, String password);
```

**Left** = Error, **Right** = Éxito.

```dart
final result = await loginUseCase(email, password);

result.fold(
  (failure) => print('Error: ${failure.message}'),
  (user) => print('Éxito: ${user.name}'),
);
```

---

## Dependencias entre capas

### Reglas de importación

```
presentation
    ↓ (puede importar)
  domain
    ↓ (puede importar)
  data
```

**Prohibido:**
- `domain` importando de `data` o `presentation`
- `data` importando de `presentation`

### Inyección de dependencias con Riverpod

Las dependencias se resuelven en los providers:

```dart
// Provider de datasource
final productDataSourceProvider = Provider<ProductRemoteDataSource>((ref) {
  final dio = ref.watch(apiClientProvider);
  return ProductRemoteDataSourceImpl(dio);
});

// Provider de repository (inyecta datasource)
final productRepositoryProvider = Provider<ProductRepository>((ref) {
  final dataSource = ref.watch(productDataSourceProvider);
  return ProductRepositoryImpl(dataSource);
});

// Provider de use case (inyecta repository)
final getProductsUseCaseProvider = Provider((ref) {
  final repository = ref.watch(productRepositoryProvider);
  return GetProducts(repository);
});
```

---

## Convenciones de código

### Nombres de archivos

- **Dart:** `snake_case.dart`
- **Clases:** `PascalCase`
- **Variables y funciones:** `camelCase`
- **Constantes:** `lowerCamelCase` (no SCREAMING_CASE)

### Prefijos para widgets compartidos (opcional)

Si tu proyecto usa widgets compartidos en un paquete separado, considera usar un prefijo:

```dart
class MyButton extends StatelessWidget { }
class MyCard extends StatelessWidget { }
```

Widgets dentro de la aplicación principal NO usan prefijo.

### Organización de imports

```dart
// 1. Dart core
import 'dart:async';

// 2. Flutter
import 'package:flutter/material.dart';

// 3. Packages externos
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:dio/dio.dart';

// 4. Packages propios (si aplica)
import 'package:my_shared_package/my_shared_package.dart';

// 5. Imports relativos (mismo feature)
import '../domain/entities/product.dart';
import '../data/models/product_model.dart';
```

### Comentarios

- **Clases públicas:** Documentar con `///`
- **Métodos públicos:** Documentar parámetros y retorno
- **Lógica compleja:** Comentarios inline `//`

```dart
/// Repositorio para gestionar datos de productos.
///
/// Implementa [ProductRepository] usando [ProductRemoteDataSource].
class ProductRepositoryImpl implements ProductRepository {
  final ProductRemoteDataSource remoteDataSource;

  ProductRepositoryImpl(this.remoteDataSource);

  /// Obtiene la lista de productos disponibles.
  ///
  /// Retorna [Right] con la lista de [Product] si tiene éxito,
  /// o [Left] con [Failure] si hay error.
  @override
  Future<Either<Failure, List<Product>>> getProducts() async {
    // Implementación...
  }
}
```

### Formateo

Usar `dart format`:

```bash
dart format .
# o
flutter format .
```

### Análisis estático

Configuración en `analysis_options.yaml` (raíz del proyecto):

```yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  strong-mode:
    implicit-casts: false
    implicit-dynamic: false
  errors:
    missing_required_param: error
    missing_return: error

linter:
  rules:
    - always_declare_return_types
    - prefer_const_constructors
    - prefer_final_locals
    - avoid_print
```

Ejecutar análisis:

```bash
dart analyze
# o
flutter analyze
```

---

## Testing

### Estructura de tests

```
my_app/
└── test/
    ├── features/
    │   └── products/
    │       ├── data/
    │       │   ├── datasources/
    │       │   │   └── product_remote_datasource_test.dart
    │       │   ├── models/
    │       │   │   └── product_model_test.dart
    │       │   └── repositories/
    │       │       └── product_repository_impl_test.dart
    │       ├── domain/
    │       │   └── usecases/
    │       │       └── get_products_test.dart
    │       └── presentation/
    │           ├── providers/
    │           │   └── products_provider_test.dart
    │           └── widgets/
    │               └── product_card_test.dart
    └── helpers/
        └── test_helpers.dart
```

### Dependencias de testing

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mockito: ^5.4.0
  build_runner: ^2.4.0
```

### Ejemplo: Test de repositorio

```dart
// test/features/products/data/repositories/product_repository_impl_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:mockito/annotations.dart';
import 'package:dartz/dartz.dart';

@GenerateMocks([ProductRemoteDataSource])
import 'product_repository_impl_test.mocks.dart';

void main() {
  late ProductRepositoryImpl repository;
  late MockProductRemoteDataSource mockDataSource;

  setUp(() {
    mockDataSource = MockProductRemoteDataSource();
    repository = ProductRepositoryImpl(mockDataSource);
  });

  group('getProducts', () {
    test('should return list of products when call is successful', () async {
      // Arrange
      final models = [
        ProductModel(
          id: '1',
          name: 'Laptop',
          price: 999.99,
          description: 'A great laptop',
        ),
      ];
      when(mockDataSource.getProducts())
          .thenAnswer((_) async => models);

      // Act
      final result = await repository.getProducts();

      // Assert
      expect(result.isRight(), true);
      result.fold(
        (failure) => fail('Should not fail'),
        (products) {
          expect(products.length, 1);
          expect(products[0].name, 'Laptop');
        },
      );
    });

    test('should return ServerFailure when there is an exception', () async {
      // Arrange
      when(mockDataSource.getProducts())
          .thenThrow(Exception('Network error'));

      // Act
      final result = await repository.getProducts();

      // Assert
      expect(result.isLeft(), true);
      result.fold(
        (failure) => expect(failure, isA<ServerFailure>()),
        (products) => fail('Should not succeed'),
      );
    });
  });
}
```

### Ejecutar tests

```bash
# Todos los tests
flutter test

# Tests con coverage
flutter test --coverage

# Tests de un archivo específico
flutter test test/features/products/data/repositories/product_repository_impl_test.dart
```

---

## Checklist de nueva feature

Cuando agregas una nueva feature, sigue estos pasos:

- [ ] Crear estructura de carpetas: `data/`, `domain/`, `presentation/`
- [ ] Definir entidades en `domain/entities/`
- [ ] Definir interfaces de repositorios en `domain/repositories/`
- [ ] Crear casos de uso en `domain/usecases/`
- [ ] Implementar DTOs en `data/models/`
- [ ] Implementar datasources en `data/datasources/`
- [ ] Implementar repositorios en `data/repositories/`
- [ ] Crear providers en `presentation/providers/`
- [ ] Crear widgets en `presentation/widgets/`
- [ ] Crear páginas en `presentation/pages/`
- [ ] Registrar rutas en `core/config/router.dart`
- [ ] Escribir tests unitarios
- [ ] Documentar con comentarios `///`

---

## Dependencias recomendadas

### Essentials

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.4.0  # State management
  go_router: ^12.0.0        # Navigation
  dio: ^5.4.0               # HTTP client
  dartz: ^0.10.1            # Functional programming (Either)
  flutter_dotenv: ^5.1.0    # Environment variables

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0     # Linting rules
  mockito: ^5.4.0           # Testing
  build_runner: ^2.4.0      # Code generation
```

---

## Referencias

- [Clean Architecture - Uncle Bob](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Riverpod Documentation](https://riverpod.dev/)
- [GoRouter Documentation](https://pub.dev/packages/go_router)
- [Dio Documentation](https://pub.dev/packages/dio)
- [Flutter Documentation](https://docs.flutter.dev/)
- [Effective Dart](https://dart.dev/guides/language/effective-dart)

---

**Última actualización:** 2025-01-13
