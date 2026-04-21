¡Hola! Como desarrollador de software, me encanta este desafío. Vamos a construir un sistema robusto. Sin embargo, antes de empezar, es vital aclarar un punto técnico para que tu aprendizaje sea preciso:

En el ecosistema de desarrollo actual, **"Antigravity"** no es un framework estándar de IA o de Flutter (como lo serían LangChain o CrewAI). Para efectos de esta práctica guiada, interpretaremos **Antigravity** como un **Framework de Orquestación de Agentes** (similar a un sistema de agentes autónomos) que dictará cómo se divide el trabajo.

Aquí tienes la metodología paso a paso para transformar una idea en un CRUD funcional de panadería.

---

## 1. Fase de Preparación: Entorno y Firebase

### Creación del Proyecto
Abre tu terminal y ejecuta:
```bash
flutter create crudpanaderia
cd crudpanaderia
```

### Configuración en Firebase Console
1.  Ve a [Firebase Console](https://console.firebase.google.com/).
2.  Crea un proyecto llamado `crud-panaderia`.
3.  En el menú lateral, ve a **Firestore Database** y haz clic en **Crear base de datos**.
4.  Selecciona "Modo de prueba" (para facilitar el desarrollo inicial) y elige la ubicación del servidor.
5.  Registra tu app (Android/iOS/Web) siguiendo los pasos y descarga el archivo `google-services.json` (para Android) colocándolo en `android/app/`.

### Integración de Librerías (`pubspec.yaml`)
Agrega estas dependencias para que Flutter pueda hablar con Firebase:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0      # Núcleo de Firebase
  cloud_firestore: ^5.0.0    # Base de datos Firestore
```
*Ejecuta `flutter pub get` en la terminal.*

---

## 2. Metodología de Agentes (Antigravity Style)

Para este flujo de trabajo, definiremos 4 agentes que interactúan para construir el CRUD:

| Agente | Rol | Skill (Habilidad) |
| :--- | :--- | :--- |
| **Architect** | Estructura de carpetas | Definición de Clean Architecture |
| **Data Master** | Modelado de Datos | Conversión JSON/Map a Objetos Dart |
| **Firebase Handler** | Conectividad | Implementación de métodos CRUD en Firestore |
| **UI Designer** | Interfaz de Usuario | Creación de formularios y listas dinámicas |

### Estructura de Carpetas Sugerida
```text
lib/
├── agents/          # Lógica de orquestación (Antigravity)
├── models/          # producto_model.dart
├── services/        # firebase_service.dart
├── screens/         # home_screen.dart y add_product_screen.dart
└── main.dart
```

---

## 3. Implementación del Código Funcional

### A. El Modelo (Skill del Data Master)
Archivo: `lib/models/producto_model.dart`
```dart
class Producto {
  String? id;
  String nombre;
  String categoria;
  double precio;

  Producto({this.id, required this.nombre, required this.categoria, required this.precio});

  // Convertir de Firestore a Objeto
  factory Producto.fromMap(Map<String, dynamic> data, String id) {
    return Producto(
      id: id,
      nombre: data['nombre'] ?? '',
      categoria: data['categoria'] ?? '',
      precio: (data['precio'] ?? 0).toDouble(),
    );
  }

  // Convertir de Objeto a Firestore
  Map<String, dynamic> toMap() {
    return {
      "nombre": nombre,
      "categoria": categoria,
      "precio": precio,
    };
  }
}
```

### B. El Servicio CRUD (Skill del Firebase Handler)
Archivo: `lib/services/firebase_service.dart`
```dart
import 'cloud_firestore/cloud_firestore.dart';
import '../models/producto_model.dart';

FirebaseFirestore db = FirebaseFirestore.instance;

// LEER
Future<List<Producto>> getProductos() async {
  List<Producto> productos = [];
  QuerySnapshot querySnapshot = await db.collection('productos').get();
  for (var doc in querySnapshot.docs) {
    productos.add(Producto.fromMap(doc.data() as Map<String, dynamic>, doc.id));
  }
  return productos;
}

// CREAR
Future<void> addProducto(String nombre, String cat, double precio) async {
  await db.collection('productos').add({
    'nombre': nombre,
    'categoria': cat,
    'precio': precio,
  });
}

// ACTUALIZAR
Future<void> updateProducto(String id, String nombre, String cat, double precio) async {
  await db.collection('productos').doc(id).set({
    'nombre': nombre,
    'categoria': cat,
    'precio': precio,
  });
}

// BORRAR
Future<void> deleteProducto(String id) async {
  await db.collection('productos').doc(id).delete();
}
```

### C. La Interfaz (Skill del UI Designer)
Archivo: `lib/screens/home_screen.dart` (Resumen funcional)
En esta pantalla usarás un `FutureBuilder` para listar y botones para ejecutar `deleteProducto`.

```dart
// Fragmento para mostrar la lista
FutureBuilder(
  future: getProductos(),
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      return ListView.builder(
        itemCount: snapshot.data?.length,
        itemBuilder: (context, index) {
          return ListTile(
            title: Text(snapshot.data![index].nombre),
            subtitle: Text("${snapshot.data![index].categoria} - \$${snapshot.data![index].precio}"),
            trailing: IconButton(
              icon: Icon(Icons.delete),
              onPressed: () async {
                await deleteProducto(snapshot.data![index].id!);
                setState(() {}); // Refrescar
              },
            ),
          );
        },
      );
    } else {
      return Center(child: CircularProgressIndicator());
    }
  },
)
```

---

## 4. Inicialización Global (`main.dart`)

Es fundamental inicializar Firebase antes de lanzar la App:

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Inicialización clave
  runApp(const CrudPanaderia());
}

class CrudPanaderia extends StatelessWidget {
  const CrudPanaderia({super.key});
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Panadería CRUD',
      home: const HomeScreen(),
    );
  }
}
```

### Flujo de trabajo de los Agentes (Metodología):
1.  **Agente Arquitecto**: Crea la estructura de carpetas y el `pubspec.yaml`.
2.  **Agente Data**: Define el modelo `Producto` basado en los campos (Nombre, Categoría, Precio).
3.  **Agente Conector**: Implementa las funciones asíncronas de Firebase en `services/`.
4.  **Agente UI**: Consume los servicios y construye los formularios de entrada y la lista de visualización.

¿Te gustaría que desarrollemos el formulario detallado para la creación de productos con validación de campos?
