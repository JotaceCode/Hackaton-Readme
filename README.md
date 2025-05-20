# DOCUMENTACIÓN API WEB - FOOD PLANNER

**Autor:** José Carlos Vélez Seisdedos
**Github:** [JotaceCode](https://github.com/JotaceCode)

>[!IMPORTANT]
>La aplicación no está finalizada.
---

## 1. Descripción del Proyecto

Esta **API RESTful** está diseñada para la planificación de comidas, ofreciendo un sistema robusto para gestionar ingredientes y menús. Cuenta con un sistema de autenticación de usuarios implementado con **Clerk**, garantizando la seguridad y personalización de la experiencia.

**Tecnologías Utilizadas:**

* **Node.js**: Entorno de ejecución JavaScript.
* **Express**: Framework web para Node.js.
* **Sequelize**: ORM para bases de datos relacionales.
* **Clerk**: Solución de autenticación para desarrolladores.

---

## 2. Estructura del Proyecto

El proyecto está organizado de manera modular para facilitar la mantenibilidad y escalabilidad:

* `src/config`: Contiene la configuración de la base de datos.
* `src/models`: Define los modelos de datos de Sequelize, que mapean las tablas de la base de datos.
* `src/controllers`: Implementa la lógica de negocio para cada endpoint de la API.
* `src/routes`: Define las rutas de la API y las asocia con sus respectivos controladores.
* `src/middleware`: Incluye middlewares, como el de autenticación con Clerk, para procesar las solicitudes antes de que lleguen a los controladores.

---

## 3. Autenticación con Clerk

La API utiliza **Clerk** para la autenticación y autorización de usuarios. El middleware `clerkAuth` protege las rutas privadas, asegurando que solo los usuarios autenticados puedan acceder a ellas.

* El middleware `clerkAuth` verifica la autenticación del usuario en cada solicitud protegida.
* El **ID del usuario autenticado** está disponible en `req.auth.userId` dentro de los controladores, lo que permite asociar los datos con el usuario correcto.

---

## 4. Endpoints Principales

A continuación, se describen los endpoints principales de la API, organizados por recurso:

### Ingredientes

Gestiona los ingredientes disponibles para el usuario autenticado.

* **`GET /ingredients`**: Obtiene una lista de todos los ingredientes asociados al usuario autenticado.
* **`POST /ingredients`**: Crea un nuevo ingrediente.
    * **Body (JSON):** `{ "name": "...", "amount": "...", "unit": "...", "category": "..." }`
* **`PUT /ingredients/:id`**: Actualiza un ingrediente existente por su ID.
    * **Body (JSON):** `{ "name": "...", "amount": "...", "unit": "...", "category": "..." }`
* **`DELETE /ingredients/:id`**: Elimina un ingrediente por su ID.

### Comidas (Meals)

Gestiona la planificación de comidas.

* **`GET /meals`**: Obtiene una lista de todas las comidas planificadas para el usuario autenticado.
* **`GET /meals/:date`**: Obtiene una lista de comidas programadas para una fecha específica.
* **`GET /meals/id/:id`**: Obtiene una comida concreta buscada por su ID primario.
* **`POST /meals`**: Crea una nueva comida, incluyendo la asociación con ingredientes.
    * **Body (JSON):** `{ "date": "YYYY-MM-DD", "mealType": "Desayuno|Almuerzo|Cena", "status": "Planificado|Cocinado|Cancelado", "ingredients": [ { "ingredientId": "...", "quantity": "...", "unit": "..." } ] }`

### Menú Semanal

Genera un menú semanal utilizando la IA de OpenAI.

* **`POST /menu/generate`**: Llama a la API de OpenAI para generar un menú semanal personalizado, basándose en los ingredientes del usuario.

---

## 5. Modelos de Datos

Estos son los modelos de datos principales utilizados en la API, que representan la estructura de las tablas en la base de datos:

* **`Ingredient`**
    * `id` (Integer, Primary Key)
    * `name` (String, Nombre del ingrediente)
    * `amount` (Float, Cantidad del ingrediente)
    * `unit` (String, Unidad de medida, ej. "gramos", "ml")
    * `category` (String, Categoría del ingrediente, ej. "Frutas", "Verduras")
    * `idUser` (Integer, Foreign Key que referencia al usuario de Clerk)


* **`User`**
    *`id` (Integer, Primary key)
    *`name` (String, guarda el username del usuario en Clerk)
    

* **`Meal`**
    * `id` (Integer, Primary Key)
    * `date` (Date, Fecha de la comida)
    * `mealType` (String, Tipo de comida: "Desayuno", "Almuerzo", "Cena")
    * `status` (String, Estado de la comida: "Planificado", "Cocinado", "Cancelado")
    * `idUser` (Integer, Foreign Key que referencia al usuario de Clerk)
    * `idMenu` (String, Foreign Key que referencia al menú generado por la IA)

* **`Menu`**
    * `id` (String, Primary Key, generado por la IA o UUID)
    * `idUser` (Integer, Foreign Key que referencia al usuario de Clerk)

* **`MealIngredients`** (Tabla intermedia para la relación muchos a muchos entre `Meal` e `Ingredient`)
    * `mealId` (Integer, Foreign Key)
    * `ingredientId` (Integer, Foreign Key)
    * `quantity` (Float, Cantidad utilizada del ingrediente en la comida)
    * `unit` (String, Unidad de medida para la cantidad utilizada)

*(Nota: La base de datos está preparada para futuras funcionalidades, como estadísticas sobre alimentos y comidas, que no han sido implementadas en la versión actual.)*

---

## 6. Testing

Para probar la API, puedes utilizar herramientas como **Postman**, **Insomnia**, o **cURL**. Es crucial que incluyas el **token de autenticación de Clerk** en los headers de las solicitudes (ej. `Authorization: Bearer <your_clerk_token>`) para acceder a rutas protegidas.

Considera las siguientes pruebas:

* **Rutas públicas:** Verifica que sean accesibles sin autenticación (si las hay).
* **Rutas privadas:** Asegúrate de que requieran autenticación y devuelvan un error `401 Unauthorized` si el token es inválido o no se proporciona.
* **Todos los endpoints:** Prueba casos de éxito (creación, lectura, actualización, eliminación de datos) y casos de error (datos inválidos, recursos no encontrados).
* **Persistencia de datos:** Confirma que los datos se guardan y recuperan correctamente de la base de datos.

---

## 7. Despliegue

Antes de desplegar la API en un entorno de producción, asegúrate de cumplir con los siguientes puntos:

* Todas las **variables de entorno** necesarias están configuradas en el entorno de producción (ej. claves de base de datos, credenciales de Clerk, claves de OpenAI).
* Las **claves de Clerk** (públicas y secretas) están configuradas correctamente en la plataforma de hosting.
* La **base de datos** está configurada, migrada y accesible desde el entorno de producción.
* La aplicación está configurada para ejecutarse en el **puerto correcto** y escuchar las solicitudes entrantes.

---

## 8. Autenticación con Clerk en el Frontend

Esta sección describe cómo la aplicación frontend (si aplica) utiliza Clerk para gestionar la autenticación de usuarios de forma sencilla y segura.

### ¿Qué hace Clerk en esta app?

* Permite a los usuarios **registrar, iniciar sesión y cerrar sesión** de manera eficiente.
* Proporciona un **`UserButton`** pre-construido para la gestión de la cuenta del usuario.
* **Protege rutas privadas** en el frontend, redirigiendo a los usuarios no autenticados.
* Expone **datos del usuario actual** (nombre, email, etc.) a través de hooks React.

### Uso Básico de Clerk

1.  **Proteger rutas con `SignedIn` y `SignedOut`**:
    Utiliza estos componentes para renderizar contenido condicionalmente basado en el estado de autenticación del usuario.

    ```javascript
    import { SignedIn, SignedOut, SignInButton, UserButton } from '@clerk/clerk-react';

    <SignedIn>
      <UserButton />
      <p>¡Hola usuario!</p>
    </SignedIn>

    <SignedOut>
      <SignInButton />
    </SignedOut>
    ```

2.  **Obtener datos del usuario**:
    El hook `useUser` proporciona acceso a la información del usuario autenticado.

    ```javascript
    import { useUser } from '@clerk/clerk-react';

    const { isSignedIn, user } = useUser();

    if (isSignedIn) {
      console.log('Email:', user?.primaryEmailAddress?.emailAddress);
      console.log('Nombre:', user?.firstName);
    }
    ```

3.  **Proteger rutas con `RedirectToSignIn`**:
    Para proteger componentes o páginas enteras, puedes usar `RedirectToSignIn` que redirige automáticamente si el usuario no está autenticado.

    ```javascript
    import { RedirectToSignIn, useUser } from '@clerk/clerk-react';

    function PrivatePage() {
      const { isLoaded, isSignedIn } = useUser();

      if (!isLoaded) return null; // O un spinner de carga
      if (!isSignedIn) return <RedirectToSignIn />;

      return <div>Página protegida ✅</div>;
    }
    ```

### Ejemplo Real de Uso en esta App

Un ejemplo común de cómo se integra Clerk en la interfaz de usuario es en un componente de cabecera:

```javascript
import { useUser, UserButton, SignInButton } from '@clerk/clerk-react';

function Header() {
  const { isSignedIn } = useUser();

  return (
    <header>
      {isSignedIn ? <UserButton /> : <SignInButton />}
    </header>
  );
}
```

![Captura de pantalla 2025-05-20 195405](https://github.com/user-attachments/assets/e8cd2bbc-251d-4156-95b4-5d402a3764f0)

![Captura de pantalla 2025-05-20 195011](https://github.com/user-attachments/assets/effae97c-39ce-4509-bbe3-2c0633a5249b)

![Captura de pantalla 2025-05-20 195333](https://github.com/user-attachments/assets/a8eec2d1-6d80-407e-a45b-57e301437c19)

