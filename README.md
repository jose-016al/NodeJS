# NodeJS ![NodeJS](.img/logo.png)

# Tabla de contenidos
- [Backend con NodeJS](#backend-con-nodejs)
  - [Instalacion de dependencias necesarias](#instalacion-de-dependencias-necesarias)
  - [Creacion de la base de datos](#creacion-de-la-base-de-datos)
  - [Conexion con la base de datos](#conexion-con-la-base-de-datos)
  - [Estructura de directorios](#estrucutra-de-directorios)
    - [Controllers](#controllers)
    - [Database](#database)
    - [Helpers](#helpers)
    - [Images](#images)
    - [Index](#Index)
    - [Models](#models)
    - [Routes](#routes)

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
```json
"scripts": {
  /* --de esta fomra podremos ejecutar el proyecto con npm start-- */
  "start": "nodemon index.js"
},
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

## Estrucutra de directorios
Usaremos el modelo vista controlador, MVC aunque en este caso no tendremos vistas si no rutas
```bash
.
├── controllers
│   └── article.js
├── database
│   └── conection.js
├── helpers
│   └── validate.js
├── images
│   └── articles
│       └── article1695578491084PRUEBA.png
├── index.js
├── models
│   └── Article.js
├── package.json
├── package-lock.json
└── routes
    └── article.js
```

### Controllers
Aqui es donde iran las funcionalidades de la API Rest, como añadir, editar, eliminar un articulo
```javascript
const fs = require('fs');
const path = require('path');
const { validate } = require('../helpers/validate');
const Article = require('../models/Article');

const add = async (req, res) => {

    /* Recoger parametros por post a guardar */
    let parameters = req.body;

    /* Validar datos */
    try {
        validate(parameters);
    } catch (error) {
        return res.status(400).json({
            status: "error",
            mensaje: "Faltan datos por enviar"
        });
    }

    /* Crear el objeto a guardar */
    const article = new Article(parameters);

    /* Asignar valores a objeto basado en el modelo (manua o automaticol */
    // article.title = parameters.title(); // Esto seria de forma manual

    /* Guardar el articulo en la base de datos */
    const articleSave = await article.save();

    /* Devolver resultado */
    return res.status(200).json({
        mensaje: "success",
        article: articleSave
    });
}

const getArticles = async (req, res) => {
    let limit = req.params.limit;

    if (limit && isNaN(parseInt(limit))) {
        return res.status(400).json({
            status: "error",
            mensaje: "El valor de límite (limit) no es válido"
        });
    }

    let query = Article.find({}).sort({ date: -1 });

    if (limit) {
        query = query.limit(parseInt(limit));
    }

    const articles = await query.exec();

    if (!articles) {
        return res.status(404).json({
            status: "error",
            mensaje: "No se han encontrado artículos"
        });
    }

    return res.status(200).json({
        status: "success",
        articles: articles
    });
}

const show = async (req, res) => {
    /* Recoger un id por la URL */
    let id = req.params.id;

    /* Buscar el artículo de manera asincrónica */
    const article = await Article.findById(id).exec();

    /* Si no existe, devolver un error */
    if (!article) {
        return res.status(404).json({
            status: "error",
            mensaje: "No se ha encontrado el artículo"
        });
    }

    /* Devolver el resultado */
    return res.status(200).json({
        status: "success",
        article: article
    });

}

const remove = async (req, res) => {
    /* Recoger un id por la URL */
    let id = req.params.id;

    /* Buscar el artículo de manera asincrónica */
    const article = await Article.findOneAndDelete({ _id: id }).exec();

    /* Si no existe, devolver un error */
    if (!article) {
        return res.status(404).json({
            status: "error",
            mensaje: "No se ha encontrado el artículo"
        });
    }

    /* Devolver el resultado */
    return res.status(200).json({
        status: "success",
        mensaje: "Articulo eliminado",
        article: article
    });

}

const edit = async (req, res) => {
    /* Recoger un id por la URL */
    let id = req.params.id;

    /* Recoger datos del body */
    let parameters = req.body;

    /* Validar datos */
    try {
        validate(parameters);
    } catch (error) {
        return res.status(400).json({
            status: "error",
            mensaje: "Faltan datos por enviar"
        });
    }

    /* Buscar y actualizar articulo */
    const article = await Article.findOneAndUpdate({ _id: id }, parameters, { new: true }).exec();

    /* Si no existe, devolver un error */
    if (!article) {
        return res.status(404).json({
            status: "error",
            mensaje: "No se ha encontrado el artículo"
        });
    }

    /* Devolver el resultado */
    return res.status(200).json({
        status: "success",
        article: article
    });
}

const uploadImage = async(req, res) => {

    /* Configurar multer */

    /* Recoger el fichero de imagen subido */
    if (!req.file && !req.files) {
        return res.status(404).json({
            status: "error",
            mensaje: "Peticion invalida"
        });
    }

    /* Nombre del archivo */
    let file = req.file.originalname;

    /* Extension del archivo */
    let file_split = file.split("\.");
    let extension = file_split[1];

    /* Comprobar extension correcta */
    if (extension != "png" && extension != "jpg" && extension != "jpeg" && extension != "gif") {
        /* Borrar archivo y dar respuesta */
        fs.unlink(req.file.path, (error) => {
            return res.status(400).json({
                status: "error",
                mensaje: "Imagen invalida"
            });
        });
    } else {
        /* Recoger un id por la URL */
        let id = req.params.id;

        /* Buscar y actualizar articulo */
        const article = await Article.findOneAndUpdate({ _id: id }, {image: req.file.filename}, { new: true }).exec();

        /* Si no existe, devolver un error */
        if (!article) {
            return res.status(404).json({
                status: "error",
                mensaje: "No se ha encontrado el artículo"
            });
        }

        /* Si todo va bien, actualizar el articulo */

        // Devolver respuesta
        return res.status(200).json({
            status: "success",
            article: article
        });
    }
}

const image = (req, res) => {
    let file = req.params.file;
    let url = "./images/articles/" + file;
    fs.stat(url, (error, exist) => {
        if (exist) {
            return res.sendFile(path.resolve(url));
        } else {
            return res.status(404).json({
                status: "error",
                mensaje: "La imagen no existe",
                existe,
                file,
                url
            });
        }
    }) 
}

const search = async(req, res) => {
    /* Sacar el string de busqueda */
    let search = req.params.search;

    /* Find Or */
    const article = await Article.find({ "$or": [
        { "title": { "$regex": search, "$options": "i" } },
        { "content": { "$regex": search, "$options": "i" } },
    ]}).sort({ date: -1 }).exec();

    if (!article || article.length <= 0) {
        return res.status(404).json({
            status: "error",
            mensaje: "No se han encontrado articulos"
        });
    }

    return res.status(200).json({
        status: "success",
        article: article
    });
}

module.exports = {
    add,
    getArticles,
    show,
    remove,
    edit,
    uploadImage,
    image,
    search
}
```

### Database
Como ya hemos visto este directorio contendra el fichero de conexion a la base de datos

### Helpers
Aqui podremos tener helpers que nos ayudaran a evitar codigo, en este caso un helper de validacion de datos
```javascript
const validator = require('validator');

const validate = (parameters) => {
    let validar_title = !validator.isEmpty(parameters.title);
    let validar_content = !validator.isEmpty(parameters.content);

    if (!validar_title || !validar_content) {
        throw new Error("No se ha validado la informacion");
    }
}

module.exports = {
    validate
}
```

### Images
Directorio de almacenamiento para las imagenes, podemos ver su funcionamienot en el controlador y en las rutas

### Index
Este sera el fichero principal desde donde cargaremos todo
```javascript
const { conection } = require("./database/conection");
const express = require("express");
const cors = require("cors");

/* Inicializar app */
console.log("App de node arrancada");

/* Conectar a la base de datos */
conection();

/* Crear servidor Node */
const app = express();
const puerto = 3900;

/* Configurar cors */
app.use(cors());

/* Convertir body a objeto js */
app.use(express.json()); // recibir datos con content-type app/json
app.use(express.urlencoded({extended: true})); // form-urlencoded

/* Crear rutas */
const router_article = require("./routes/article");

/* Cargar rutas */
app.use("/api", router_article);

/* Crear servidor y escuchar peticiones http */
app.listen(puerto, () => {
    console.log("Servidor corriendo en el puerto: " + puerto);
});
```

### Models
Aqui iran todas las entidades que necesitemos, y podremos controlar los datos que se enviaran a la base de datos
```javascript
const { Schema, model } = require("mongoose");

const ArticleSchema = Schema({
    title: {
        type: String,
        required: true
    },
    content: {
        type: String,
        required: true
    },
    date: {
        type: Date,
        default: Date.now()
    },
    image: {
        type: String,
        required: false
    }
});

module.exports = model("Article", ArticleSchema);
                      // articles
```

### Routes
Aqui iran nuestras rutas para consultar la api
```javascript
const express = require('express');
const router = express.Router();
const multer = require('multer');

const storage = multer.diskStorage({
    destination: function(req, file, cb) {
        cb(null, "./images/articles");
    },
    filename: function(req, file, cb) {
        cb(null, "article" + Date.now() + file.originalname);
    }
});

const uploads = multer({storage: storage});

const ArticleController = require("../controllers/article");

/* Rutas de pruebas */
router.post("/add", ArticleController.add);
router.get("/articles/:limit?", ArticleController.getArticles);
router.get("/article/:id", ArticleController.show);
router.delete("/article/:id", ArticleController.remove);
router.put("/article/:id", ArticleController.edit);
router.post("/upload-image/:id", [uploads.single("file0")], ArticleController.uploadImage);
router.get("/image/:file", ArticleController.image);
router.get("/search/:search", ArticleController.search);

module.exports = router;
```
