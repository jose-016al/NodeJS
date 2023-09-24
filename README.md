# Backend con NodeJS

# Tabla de contenidos
- [Backend con NodeJS](#backend-con-nodejs)
  - [Instalacion de dependencias necesarias](#instalacion-de-dependencias-necesarias)
  - [Creacion de la base de datos](#creacion-de-la-base-de-datos)
  - [Conexion con la base de datos](#conexion-con-la-base-de-datos)

Vamos a aprender a usar NodeJS para crear un backend, con una API Rest, para ello,
empecemos por crear una carpeta dodne ira nuestro proyecto, e inicializarla como un proyecto Node
```jsx
npm init
```
Una vez hayamos rellenado los campos que nos indica, nos dara la salida del package.json, que seria algo asi
```json
{
  "name": "api-rest-node",
  "version": "1.0.0",
  "description": "API Rest con NodeJS",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Jose Almiron",
  "license": "MIT"
}
```
## Instalacion de dependencias necesarias
Instalamos Express, que sera el encargado de gestionar las peticiones http, con --save indicamos que se instale en el proyecto y no de forma global
```bash
npm install express --save
```
Con mongoose nos permitira trabajar de una fomra mas comoda con mongodb
```bash
npm install mongoose --save
```
Seria interesante tener una libreria que nos permita subir archivos, ya sea imagenes o cualquier otro tipo de fichero
```bash
npm install multer --save
```
Una libreria que nos permita validar datos
```bash
npm install validator --save
```
Para evitar errores con las CORS 
```bash
npm install cors --save
```
Nodemon es una dependencia que vamos a tener solo en el desarrollo y no cuando el proyecto este en produccion, lo que nos permite es monitorear todos los cambios que realicemos en le proyecto
```bash
npm install nodemon --save-dev
```
Cuando estemos en desarrollo seria interesante añadir un script al package.json, a la hora de ejecutar el proyecto, es decir para usar nodemon
```jsx

```

## Creacion de la base de datos
Para crear la base de datos usaremos [MongoDB](https://github.com/jose-016al/MongoDB), en este repositorio explico lo basico para usar mongodb, asi como su intalacion, aunque tambien podemos usar la documentacion oficial

## Conexion con la base de datos
Para la conexion con la base de datos, crearemos una carpeta llamada "database" y dentro un archivo javascript para la connexion "conection.js"
```javascript
const mongoose = require("mongoose");

const conection = async() => {

    try {
        await mongoose.connect("mongodb://localhost:27017/mi_blog");
        /* Parametros dentro de objeto, solo en caso de aviso */
        /* useNewUrlParser: true */
        /* useUnifiedTopology: true */
        /* useCreateIndex: true */
        console.log("Conexion establecida");
    } catch (error) {
        console.log(error);
        throw new Error("No se ha podido conectar a la base de datos");
    }

}

module.exports = {
    conection
}
```

# Estrucutra de directorios
Usaremos el modelo vista controlador, MVC aunque en este caso no tendremos vistas si no rutas
```jsx

```