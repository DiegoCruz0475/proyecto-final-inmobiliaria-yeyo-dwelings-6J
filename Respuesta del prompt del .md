Entendido. Vamos a estructurar la habilidad de agente global **.agents** y el proyecto **YeyoDwelings** siguiendo una arquitectura profesional.

---

## 1. Estructura de la Habilidad Agente Global: `.agents`

Primero, creamos el ecosistema del agente que gestionará tus habilidades de diseño, código y scraping.

### Archivo `SKILL.md`
Este archivo define la identidad y capacidades del agente.

```markdown
# Agente Global: .agents - Ecosistema YeyoDwelings
## Descripción
Agente especializado en el desarrollo full-stack con Flutter y Firebase, con capacidades de automatización mediante sub-skills.

## Habilidades (Skills)
* **Skill de Diseño:** Implementación de UI/UX siguiendo Material Design 3.
* **Skill de Código:** Lógica de negocio en Dart y arquitectura limpia.
* **Skill de Scraping:** Automatización para poblar catálogos de dinosaurios.

## Estructura de Directorios
- /scripts: Scripts de automatización y despliegue.
- /ejemplos: Plantillas de UI y lógica CRUD.
- /resources: Assets, iconos y archivos JSON de configuración.
```

---

## 2. Configuración del Entorno y Prerrequisitos

### Verificación de Herramientas
Ejecuta estos comandos en tu terminal (VS Code o Antigravity):

1.  **Flutter:** `flutter --version` (Si no está, descárgalo en [flutter.dev](https://flutter.dev)).
2.  **Firebase CLI:** `npm install -g firebase-tools`.
3.  **Login:** `firebase login`.
4.  **Flutterfire:** `dart pub global activate flutterfire_cli`.

### Preparación del Proyecto
Crea la carpeta principal y el proyecto:
```bash
mkdir proyectoYeyoDwelings
cd proyectoYeyoDwelings
flutter create .
flutterfire configure
```
*Selecciona tu proyecto de Firebase Console y la plataforma Android.*

---

## 3. Configuración de Dependencias (`pubspec.yaml`)

Asegúrate de que tu archivo incluya las librerías necesarias para Firestore y navegación:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0
  cloud_firestore: ^5.0.0
  cupertino_icons: ^1.0.6
```
Ejecuta: `flutter pub get`

---

## 4. Implementación del Proyecto: YeyoDwelings

Estructuraremos el código para el CRUD de **Casas** (Tienda de Dinosaurios/Dwellings).

### A. Modelo de Datos (`lib/models/casa_model.dart`)
```dart
class Casa {
  String id;
  String nombre;
  double precio;
  String descripcion;

  Casa({required this.id, required this.nombre, required this.precio, required this.descripcion});

  Map<String, dynamic> toMap() => {
    'nombre': nombre,
    'precio': precio,
    'descripcion': descripcion,
  };

  factory Casa.fromFirestore(Map<String, dynamic> data, String id) {
    return Casa(
      id: id,
      nombre: data['nombre'] ?? '',
      precio: (data['precio'] ?? 0).toDouble(),
      descripcion: data['descripcion'] ?? '',
    );
  }
}
```

### B. Servicio Firestore (`lib/services/firebase_service.dart`)
```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/casa_model.dart';

class FirebaseService {
  final CollectionReference casasRef = FirebaseFirestore.instance.collection('casas');

  // CREATE
  Future<void> addCasa(Casa casa) => casasRef.add(casa.toMap());

  // READ
  Stream<List<Casa>> getCasas() {
    return casasRef.snapshots().map((snapshot) =>
        snapshot.docs.map((doc) => Casa.fromFirestore(doc.data() as Map<String, dynamic>, doc.id)).toList());
  }

  // UPDATE
  Future<void> updateCasa(Casa casa) => casasRef.doc(casa.id).update(casa.toMap());

  // DELETE
  Future<void> deleteCasa(String id) => casasRef.doc(id).delete();
}
```

### C. UI: Pantalla Listado y Formulario (`lib/screens/home_screen.dart`)
```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';
import '../models/casa_model.dart';

class HomeScreen extends StatelessWidget {
  final FirebaseService _service = FirebaseService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('YeyoDwelings - Tienda')),
      body: StreamBuilder<List<Casa>>(
        stream: _service.getCasas(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          final casas = snapshot.data!;
          return ListView.builder(
            itemCount: casas.size,
            itemBuilder: (context, index) {
              return ListTile(
                title: Text(casas[index].nombre),
                subtitle: Text('\$${casas[index].precio}'),
                trailing: IconButton(
                  icon: Icon(Icons.delete, color: Colors.red),
                  onPressed: () => _service.deleteCasa(casas[index].id),
                ),
                onTap: () => _mostrarDialogo(context, casa: casas[index]),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => _mostrarDialogo(context),
      ),
    );
  }

  void _mostrarDialogo(BuildContext context, {Casa? casa}) {
    final nombreCtrl = TextEditingController(text: casa?.nombre);
    final precioCtrl = TextEditingController(text: casa?.precio.toString());

    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text(casa == null ? 'Nueva Casa' : 'Editar Casa'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: nombreCtrl, decoration: InputDecoration(labelText: 'Nombre')),
            TextField(controller: precioCtrl, decoration: InputDecoration(labelText: 'Precio'), keyboardType: TextInputType.number),
          ],
        ),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context), child: Text('Cancelar')),
          ElevatedButton(
            onPressed: () {
              final nuevaCasa = Casa(
                id: casa?.id ?? '',
                nombre: nombreCtrl.text,
                precio: double.parse(precioCtrl.text),
                descripcion: 'Hogar para dinosaurios',
              );
              casa == null ? _service.addCasa(nuevaCasa) : _service.updateCasa(nuevaCasa);
              Navigator.pop(context);
            },
            child: Text('Guardar'),
          ),
        ],
      ),
    );
  }
}
```

---

## 5. Punto de Entrada (`lib/main.dart`)

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(YeyoDwelingsApp());
}

class YeyoDwelingsApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'YeyoDwelings',
      theme: ThemeData(primarySwatch: Colors.green, useMaterial3: true),
      home: HomeScreen(),
    );
  }
}
```

---

### Verificación de Funcionalidad
1.  **Prueba de Conexión:** Revisa en Firebase Console que la base de datos Firestore esté en "Modo de prueba" o con reglas de escritura abiertas.
2.  **Ejecución:** En la terminal de VS Code, presiona `F5` o escribe `flutter run`.
3.  **CRUD:** Intenta agregar una "Casa de T-Rex", edítala y bórrala para confirmar la sincronización en tiempo real.
