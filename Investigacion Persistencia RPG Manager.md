# Investigación previa · Persistencia RPG


### INTEGRANTES: 
**Andres Villarraga Piratoba** - `20252578015`

**Sergio Pastrana** - `20252578086`

**Juan Daniel Garzon Solaque** - `20252578019`

---
### **¡Aclaración!: Se uso Gemini para los códigos aplicados a nuestro RPG Manager.**
---

## <p align="center">Tema 1: El Patron Repository</p>


**1. ¿Qué es el patrón Repository (o DAO) y qué problema resuelve?**

El patrón Repository y el patrón DAO son patrones de diseño de software que nos sirven para separar la lógica de negocio de la capa de acceso a datos.

Aunque ambos patrones estan destinados con el mismo proposito manejan diferentes niveles de abstracción, notese la diferenciación a continuación:

**Patrón DAO (Objeto de Acceso a Datos)**

Separa la lógica de negocio de la capa de acceso a datos.
Mejora la modularidad y facilita el cambio de la fuente de datos sin afectar la lógica de negocio (Enfoque en operaciones CRUD a nivel de tablas/registros).

**Patrón Repository**

Abstrae el acceso a datos proporcionando una interfaz de alto nivel para interactuar con varios tipos de almacenamiento de datos (base de datos, API, memoria, etc.)

### __***Ejemplo:***__

 Imaginemos que creamos una interfaz o clase abstracta para manejar las operaciones de nuestros personajes, separando el controlador HTTP de la forma en la que se guardan y consultan los datos

- ### En Java (Javalin)
    Primero definimos el contrato (la interfaz del repositorio):
    
   ```
    public interface PersonajeRepository {
    List<Personaje> obtenerTodos();
    Personaje buscarPorId(String id);
    void guardar(Personaje personaje);
    }
    ```
    Luego, implementamos una versión en memoria que simula el almacenamiento actual y más adelante podríamos crear otra implementación para una base de datos (Ej: MySQL) sin tocar la lógica del controlador.


    ```
    public class PersonajeRepositoryEnMemoria implements PersonajeRepository {
    private List<Personaje> almacenamiento = new ArrayList<>();

        @Override
        public List<Personaje> obtenerTodos() {
            return almacenamiento;
        }

        @Override
        public Personaje buscarPorId(String id) {
            return almacenamiento.stream()
                    .filter(p -> p.getId().equals(id))
                    .findFirst()
                    .orElse(null);
        }

        @Override
        public void guardar(Personaje personaje) {
            almacenamiento.add(personaje);
        }
    }
    ```
<br>

- ### En Python (Flask)
    En Python podemos usar clases y tipado para lograr el mismo nivel de abstracción:

    ```
    from abc import ABC, abstractmethod

    class PersonajeRepository(ABC):
        @abstractmethod
        def obtener_todos(self):
            pass

        @abstractmethod
        def guardar(self, personaje):
            pass

    class PersonajeRepositoryEnMemoria(PersonajeRepository):
        def __init__(self):
            self.almacenamiento = []

        def obtener_todos(self):
            return self.almacenamiento

        def guardar(self, personaje):
            self.almacenamiento.append(personaje)

    ```

<br>

*Este patrón nos servira a futuro cuando integremos una base de datos real (Ej: MySQL con JDBC o SQLAlchemy), ya que el controlador de Javalin o Flask no sufrirá cambios complejos, ya que solo interactua con métodos del `Repository` (`obtener_todos`,`guardar`), lo que aisla el motor de la base de datos del flujo web.* 

<br>

**2. ¿Qué diferencia hay entre la capa de negocio (Service) y la capa de datos
(Repository)? Den un ejemplo con una regla del RPG Manager.**

La diferencia principal radica en qué hace y dónde interactúan dentro de la arquitectura de la aplicación:

**Capa de Negocio (`Service`)**

Es la capa que contiene la lógica del negocio (reglas y flujos de trabajo), es decir, la capa que se encarga de orquestar las operaciones, validar datos, aplicar cálculos y tomar decisiones.

Llama a la capa de repositorio para pedir o guardar datos, pero no le importa cómo se guardan físicamente en la base de datos.

**Capa de datos (`Repository`)**

Es la capa que se encarga del acceso a datos (persistencia). Dicho de otra manera, es un intermediario entre la base de datos y la aplicación.

Se encarga exculsivamente de las operaciones CRUD y de las consultas en la base de datos (SQL, archivos, etc.).

### __***Ejemplo:***__

Imaginemos que tenemos la siguiente regla de negocio importante:

 > Cuando se crea un nuevo personaje, su nivel base debe ser 1, pero si clase del personaje es "Mago", automáticamente debe recibir un bono de +50 puntos de Maná base antes de guardarlo.

Ahora veamos cómo se divide esta responsabilidad entre el `Service` y el `Repository`.

- ### 1. Capa de Negocio (`PersonajeService`)

    Aquí aplicamos la regla, validamos y orquestamos.

    - ### En Java (Javalin):
     ```
        public class PersonajeService {
        private final PersonajeRepository repository;

        public PersonajeService(PersonajeRepository repository) {
            this.repository = repository;
        }

        public void registrarPersonaje(Personaje personaje) {
            // 1. Aplicamos la lógica y reglas de negocio
            personaje.setNivel(1);
            
            if ("Mago".equalsIgnoreCase(personaje.getClase())) {
                personaje.setMana(personaje.getMana() + 50); // Bono de regla de negocio
            }

            // 2. Delegamos la persistencia al Repositorio
            repository.guardar(personaje);
        }
    }
     ```
    <br>

    - ### En Python (Flask):
    ```
        class PersonajeService:
        def __init__(self, repository):
            self.repository = repository

        def registrar_personaje(self, personaje):
            # 1. Aplicamos la regla de negocio
            personaje.nivel = 1
            
            if personaje.clase.lower() == "mago":
                personaje.mana += 50  # Bono de regla de negocio

            # 2. Delegamos la persistencia al Repositorio
            self.repository.guardar(personaje)
    ```

<br>

- ### 2. Capa de Datos / Persistencia (`PersonajeService`)

    A este nivel no le importa si el personaje es Mago, Guerrero o si tiene bonos de maná; su única misión es recibir el objeto ya procesado y llevarlo al almacenamiento:

    - ### En Java:

    ```
    public class PersonajeRepositoryEnMemoria implements PersonajeRepository {
        private List<Personaje> database = new ArrayList<>();

        @Override
        public void guardar(Personaje personaje) {
            // Solo ejecuta la inserción o almacenamiento físico
            database.add(personaje);
        }
    }
    ```

    - ### En Python:
    
    ```
    class PersonajeRepositoryEnMemoria(PersonajeRepository):
        def __init__(self):
            self.database = []

        def guardar(self, personaje):
            # Solo guarda el objeto en la lista/base de datos
            self.database.append(personaje)
    ```
    
    *Esta separación es clave, ya que si en una situación hipotética, modificamos alguna regla del juego (por ejemplo, que los arqueros tengan más agilidad o que los magos obtengan un bono diferente), solo modificamos el `Service`*
    
    *El `Repository` se mantiene intacto porque a él solo le interesa guardar datos, no calcular atributos del juego.*

<br>

**3. ¿Por qué se dice que un Repository no debe validar reglas de negocio?.**

Un Repository no debe validar reglas de negocio porque su única responsabildiad es la persistencia y el acceso a los datos.

Al mezclar la lógica de almacenamiento con las reglas de negocio, se viola el **Principio de responsabilidad Única (SRP)** de SOLID y rompe la separación de capas.

Por ejemplo, si ponemos una regla de negocio dentro de un repositorio (Ej: `registrarPersonaje()`), esa regla queda atada a un mecanismo de persistencia específico. Si el día de mañana necesitamos registrar un usuario desde un comando de consola, un archivo de pruebas o migras de base de datos, tendriamos que duplicar la validación o arrastrar dependencias innecesarias.

Para **Testing (Pruebas)** las reglas de negocio deben ser fáciles y rápidas de probar, sin necesidad de levantar una base de datos o usar simuladores (mocks) complejos.

### Fuentes citadas:
https://platzi.com/cursos/java-sql/patron-dao-vs-repository-en-java/


---

## <p align="center">Tema 2: Contratos en Java y Python</p>

**1. ¿Qué es una interface en Java? ¿Qué pasa si una clase que la implementa olvida un
método?**

Una interfaz (`interface`) en Java es un contrato que define un conjunto de métodos pero sin implementar la lógica de como se ejecutan. Sirve para establecer qué deber hacer una clase, permitiendo que diferentes clases compartan un mismo comportamiento (poliformismo) aunque lo realicen de distinta manera.

Si una clase declara que implementa una interfaz, **está obligada a definir todos los métodos abstractos que contiene.** Si olvida implementar uno o más métodos habrá un error de compilación o una excepción (si se declara abstracta) y no podremos crear instancias (objetos) de dicha clase directamente. Tendriamos que esperar a que otra clase la extienda y complete el método faltante.

### __***Ejemplo:***__

Imaginemos que en nuestro RPG todos los personajes deben poder realizar una acción de ataque, pero cada clase lo ejecuta de forma diferente.

Creamos la interfaz para definir el contrato:

    ```
        public interface AccionCombate {
        void atacar(Personaje objetivo);
        int calcularDanioBase();
    }
    ```
- ### **CASO: IMPLEMENTACIÓN CORRECTA (Guerrero)**
    Aquí implementamos los métodos que exige la interfaz:

    ```
    public class Guerrero extends Personaje implements AccionCombate {

    @Override
        public void atacar(Personaje objetivo) {
            int danio = calcularDanioBase();
            System.out.println("El Guerrero ataca con su espada causando " + danio + " de daño.");
            // Lógica para restar vida al objetivo
        }

        @Override
        public int calcularDanioBase() {
            return 25; // Daño base del guerrero
        }
    }
    ```

- ### **CASO: ¿Qué pasa si "olvidamos un método? (Mago)**
    Si creamos la clase `Mago` y por decuido se nos olvida implementar el método `calcularDanioBase()`, Java dejará compilar el proyecto.

    ```
    // ESTO GENERA ERROR DE COMPILACIÓN
    public class Mago extends Personaje implements AccionCombate {

    
        @Override
        public void atacar(Personaje objetivo) {
            System.out.println("El Mago lanza un hechizo arcano.");
        }
        
        // ¡Omitimos calcularDanioBase()!
    }
    ```

<br>

**2. ¿Qué es una clase abstracta con ABC y @abstractmethod en Python? ¿Qué pasa si
intentan crear un objeto de esa clase?**
 
En Python, una clase abstracta es una clase que no se puede instanciar directamente. Es decir que funciona como una plantilla o contrato (con el módulo `abc`) para obligar a las clases hijas a implementar ciertos métodos.

Si intentamos instanciar directamente la clase abstracta, Python lanza un `TypeError`. Esto sucede porque el lenguaje protege el contrato a través de su metaclase (`ABCMeta`), impidiendo que existan métodos incompletos o que no cumplan con la interfaz definida.

**3. ¿En qué se parecen y en qué se diferencian las dos formas?**

La principal diferencia radica en que Java es de tipado estático, es decir, que necesita interfaces para simular la herencia múltiple, mientras que Python es un lenguaje dinámico que ya soporta herencia multiple de forma nativa y usa sus clases abstractas para validación y guía estructural.

Ambas formas se parecen en caracteristicas cómo:

- **Contratos de diseño:** Ambas se utilizan para definir un conjunto de métodos que las clases hijas estan obligadas a cumplir.

- **No instanciables:** No permiten crear objetos directamente de las interfaces o clases abstractas.

- **Métodos abstractos:** Permiten declarar la firma de un método (nombre y parámetros) sin proporcionarle un cuerpo o implementación (exceptuando los métodos por defectos).

Y se diferencian en:

- **Mecanismo general:** En Java la clase abstracta es una estructura completamente distinta a una clase, mientras que en Python es una clase normal que hereda de la base `ABC` 

- **Herencia Múltiple:** Una clase en Java implementar múltiples interface, pero solo heredar de una clase. Python permite herencia múltiple nativa, donde una clase puede heredar de varias clases abstractas o normales.

- **Momento de Validación:** El compilador en Java detecta si falta un método en el tiempo de compilación, mientras que en Python la validación ocurre en el tiempo de ejecución, al momento de intentar instanciar la clase hija.

### Fuentes citadas: 
- https://coddy.tech/learn/es/java/object_oriented_programming/introduction_to_interfaces

- https://platzi.com/cursos/python-poo/clases-abstractas-en-python-con-abc-y-ab/

---

## <p align="center">Tema 3: Inyección de dependencias por constructor</p>

**1.¿Qué significa "programar contra una interfaz y no contra una implementación"?**

Esta frase significa separar el "qué hace" un componente de "cómo lo hace.

En lugar de que el código dependa de una clase concreta y específica, depende de una abstracción (un contrato o interfaz), sin importar que clase específica los esté ejecutando por detrás.

**2. ¿Qué es la inyección de dependencias por constructor? Explíquenlo con un
PersonajeService que necesita guardar personajes.**

La inyección de dependencias por constructor es un patrón de diseño donde los objetos o servicios que una clase (sus dependencias) necesita, se le pasan a su método constructor al momento de crear la instancia.

En vez de que una clase cree por sí misma sus dependencias usando el operador `new`, las recibe ya listas e intanciadas a través de su constructor.

### __***Ejemplo:***__

Imaginemos que nuestro `PersonajeService` (la capa de negocio) necesita obligatoriamente un repositorio para poder registrar y guardar los personajes creados. Con la inyección por constructor, el servicio no sabe cómo se conecta el repositorio (si es en memoria, en MySQL o en MariaDB), solo sabe que lo necesita para funcionar.

- ### En Java (Javalin)
    En Java, declaramos el repositorio como un campo final (para asegurar que sea inmutable) y lo recibimos como parámetro en el constructor de la clase:

    ```
    public class PersonajeService {
        // 1. Declaramos la dependencia como final
        private final PersonajeRepository personajeRepository;

        // 2. Inyección por constructor
        public PersonajeService(PersonajeRepository personajeRepository) {
            this.personajeRepository = personajeRepository;
        }

        public void registrarPersonaje(Personaje nuevoPersonaje) {
            // Lógica de negocio (ej. validar estadísticas)
            if (nuevoPersonaje.getNivel() <= 0) {
                nuevoPersonaje.setNivel(1);
            }
            
            // Usamos la dependencia inyectada
            personajeRepository.guardar(nuevoPersonaje);
            System.out.println("Personaje registrado exitosamente en el RPG Manager.");
        }
    }
    ```

    ¿Cómo se inicializa esto en el sistema principal (Main.java o el enrutador de Javalin)?

    ```
    // Creamos la implementación concreta de la capa de datos
    PersonajeRepository repository = new PersonajeRepositoryEnMemoria();

    // Inyectamos la dependencia pasándosela al constructor del Service
    PersonajeService personajeService = new PersonajeService(repository);
    ```

<br>

- ### En Python (Flask o Fast API)
    En Python, hacemos exactamente lo mismo utilizando el método constructor `__init__`:

    ```
    class PersonajeService:
        def __init__(self, personaje_repository):
        # Inyección de la dependencia por constructor
        self.personaje_repository = personaje_repository

        def registrar_personaje(self, nuevo_personaje):
            # Lógica de negocio
            if nuevo_personaje.nivel <= 0:
                nuevo_personaje.nivel = 1
            
            # Usamos la dependencia inyectada
            self.personaje_repository.guardar(nuevo_personaje)
            print("Personaje registrado exitosamente en el RPG Manager.")
    ```

    ¿Cómo se instancia en la aplicación?
    
    ```
    # Instanciamos el repositorio
    repositorio = PersonajeRepositoryEnMemoria()

    # Inyectamos el repositorio al servicio al instanciarlo
    servicio_personaje = PersonajeService(repositorio)
    ```
<br>

**3. ¿Qué ventaja tiene que el Service reciba el repositorio en vez de crearlo él mismo?**

La principal ventaja de que un Service reciba el repositorio en lugar de instanciarlo él mismo es el **desacoplamiento** usando la inyección de dependencias que vimos previamente.

Al delegar la creación del repositorio a un agente externo, nuestro código obtiene múltiples beneficios estructurales:

- **Facilitar las pruebas unitarias (Unitarias Testing):** Si el Service crea su propio repositorio, quedaremos atados a una base de datos real, mientras que al recibirlo, podemos hacer un mock durante las pruebas. Lo que permite probar la lógica de negocio del Service de forma aislada, rápida y sin necesidad de conectarnos a una base de datos real, ni de usar datos reales en producción o desarrollo.

- **Principio de Inversión de Dependencias (SOLID):** Al recibir el repositorio, el Service depende de una abstracción y no de una implementación concreta, lo que hace que al Service solo le importe que el objeto tenga los métodos necesarios en vez de que sepa que base de datos usa al crearlo él mismo.

- **Mayor reutilizacion y control del ciclo de vida:** Cuando el Service instancia el repositorio, perdemos el control sobre cómo se gestiona esa instancia. Al recibirlo, el contenedor de dependencias puede decidir si el repositorio debe ser un **Singleton** (una unica instancia para toda la aplicación) o si debe tener un ciclo de vida compartido con otras clases.


### Fuentes citadas:
- https://builtin.com/articles/dependency-injection (Trad. Español)

- https://medium.com/@Masoncoding/programming-to-an-interface-not-an-implementation-024d01815070 (Trad. Español)

## <p align="center">Tema 4: Configuración y secretos</p>

**¿Como se lee un archivo .properties en Java y un archivo .ini en Python?
Nombren la clase o el modulo que se usa. **

Para leer archivos de configuración en Java y Python se utilizan las siguientes herramientas:

Java (`.properties`):
Clase:** `java.util.Properties`
Cómo se lee:** Se instancia la clase `Properties` y se utiliza su metodo `load(InputStream inStream)` o `load(Reader reader)` para cargar el contenido del archivo de propiedades en memoria.

Python (`.ini`):
Modulo: `configparser`
Como se lee: Se importa el modulo, se inicializa un objeto con `configparser.ConfigParser()` y se utiliza el metodo `read('nombre_archivo.ini')` para procesar el archivo.

### __***Ejemplo:***__

En el caso del proyecto RPG manager podriamos utilizarlos para definir parametros iniciales del juego, como los datos del héroe por defecto en la taberna, los puertos del servidor o las reglas de validación de la siguiente manera: 

---

### 1. Versión Java (`RPG Manager` con Javalin)

En Java utilizamos la clase `java.util.Properties` para definir los valores iniciales de la taberna y la configuración del puerto del servidor web.

### Archivo de configuración (`config.properties`)
Ubicación: `src/main/resources/config.properties`

```properties
server.port=7070
taberna.default_hero_name=Aragorn
taberna.default_hero_class=Guerrero
taberna.default_hero_level=10
taberna.default_hero_vida=100
```

### Implementación en `RPG_manager.java`

´´´java
package com.mycompany.rpg_manager;

import io.javalin.Javalin;
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;
import controlador.PersonajeController;

public class RPG_manager {
    public static void main(String[] args) {
        Properties props = new Properties();
        
        // Cargar el archivo config.properties desde el classpath/recursos
        try (InputStream input = RPG_manager.class.getClassLoader().getResourceAsStream("config.properties")) {
            if (input != null) {
                props.load(input);
            } else {
                System.out.println("Archivo de propiedades no encontrado, se usarán valores por defecto.");
            }
        } catch (IOException ex) {
            System.err.println("Error al cargar el archivo de configuración: " + ex.getMessage());
        }

        // Obtener valores del archivo .properties (con valores de respaldo)
        int puerto = Integer.parseInt(props.getProperty("server.port", "7070"));
        String nombreHeroe = props.getProperty("taberna.default_hero_name", "Aventurero");
        String claseHeroe = props.getProperty("taberna.default_hero_class", "Guerrero");
        int nivelHeroe = Integer.parseInt(props.getProperty("taberna.default_hero_level", "1"));
        int vidaHeroe = Integer.parseInt(props.getProperty("taberna.default_hero_vida", "100"));

        // 1. Agregar el personaje inicial usando la configuración leída
        Taberna.repositorioPersonajes.add(new Personaje(nombreHeroe, claseHeroe, nivelHeroe, vidaHeroe));

        // 2. Iniciar Javalin en el puerto obtenido
        Javalin app = Javalin.create(config -> {
            config.staticFiles.add("/public");
        }).start(puerto);

        // 3. Mapear las rutas HTTP
        app.get("/personajes", PersonajeController::obtenerTodos);
        
        System.out.println("Servidor RPG Manager iniciado en el puerto: " + puerto);
    }
}
```

---

### 2. Versión Python (`RPG Manager` con Flask)

En Python utilizamos el módulo nativo `configparser` para gestionar reglas de negocio (rangos de niveles permitidos) y configuraciones generales en la capa de servicios.

### Archivo de configuración (`config.ini`)
Ubicación: Raíz del proyecto (`Proyect_RPG_Python/config.ini`)

´´´ini
[GameRules]
min_nivel = 1
max_nivel = 100
vida_por_defecto = 100

[Server]
port = 5000
debug = true
```

### Implementación en la Capa de Servicios (`services/personaje_service.py`)

```python
import configparser
import os

# Cargar el archivo config.ini desde la raíz del proyecto
config = configparser.ConfigParser()
config_path = os.path.join(os.path.dirname(__file__), '../config.ini')
config.read(config_path)

# Leer reglas de negocio desde el archivo .ini (usando conversión de tipos con getint)
MIN_NIVEL = config.getint('GameRules', 'min_nivel', fallback=1)
MAX_NIVEL = config.getint('GameRules', 'max_nivel', fallback=100)
VIDA_DEF = config.getint('GameRules', 'vida_por_defecto', fallback=100)

def es_valido(nombre, clase, nivel):
    """
    Valida los datos del personaje utilizando las reglas 
    cargadas dinámicamente desde el archivo config.ini.
    """
    # 1. Validar nombre no vacío
    if not nombre or not nombre.strip():
        return False, "El nombre del personaje no puede estar vacío."

    # 2. Validar clases permitidas
    clases_permitidas = ["Guerrero", "Mago", "Arquero"]
    if clase.capitalize() not in clases_permitidas:
        return False, f"Clase inválida. Debe ser una de: {clases_permitidas}"

    # 3. Validar nivel dentro del rango configurado en config.ini (1 a 100)
    if not (MIN_NIVEL <= nivel <= MAX_NIVEL):
        return False, f"El nivel debe estar comprendido entre {MIN_NIVEL} y {MAX_NIVEL}."

    return True, "OK"
```

**2.¿Para qué sirve .gitignore y qué pasa si un archivo ya se subió antes de ignorarlo?**

¿Para que sirve .gitignore?

- Especifica archivos intencionalmente no rastreados que Git debe ignorar.

- Su proposito es asegurar que ciertos archivos que no son rastreados por Git permanezcan sin rastrear.

¿Que pasa si un archivo ya se subio antes de ignorarlo?

- Los archivos que ya están siendo rastreados por Git no se ven afectados.

- Para dejar de rastrear un archivo actual, se debe usar git rm --cached para eliminarlo del índice.

- Posteriormente, el nombre del archivo se puede añadir al archivo .gitignore para evitar que se vuelva a introducir en commits futuros.

**3. Busquen un caso real de contraseñas o llaves filtradas en GitHub: qué pasó y quéconsecuencias tuvo**

En 2016, Uber sufrió una brecha que afectó a unos 57 millones de usuarios y conductores. Los atacantes entraron a un repositorio privado de GitHub con credenciales robadas, encontraron allí una llave de acceso de AWS escrita en el código y con ella accedieron a un bucket S3 con los datos. Uber pagó 100.000 dólares a los atacantes para mantenerlo en secreto y luego tuvo consecuencias legales por ocultarlo. Lección: una llave dentro del código es un riesgo aunque el repositorio sea privado.

### Fuentes citadas:
- https://docs.oracle.com/javase/8/docs/api/java/util/Properties.html
- https://www.freecodecamp.org/news/how-to-parse-ini-config-files-in-python-with-configparser/
- https://git-scm.com/docs/gitignore
- https://www.securityweek.com/uber-settles-federal-investigators-over-2016-data-breach-coverup/

---

## <p align="center">Tema 5: MariaDB desde el codigo</p>

 **1. ¿Qué es JDBC y qué hace un driver? ¿Qué librería cumple ese papel en Python paraMariaDB?**

Java Database Connectivity (JDBC) es la API estandar de Java para hablar con la base de datos relacionales. El driver es aquel que traduce las llamadas estandar al protocolo de un motor especifico.
En Python este palpel lo puede cumplir la libreria PyMySQL.

**2. ¿Qué es la inyección SQL? Muestren un ejemplo con el nombre de un personaje y expliquen por que PreparedStatement (Java) o los parámetros %s (Python) la evitan.**

Inyección SQL Ocurre cuando el texto que escribe el usuario se pega dentro de la consulta y termina interpretandose como codigo SQL.

### __***Ejemplo:***__

    ´´´SELECT * FROM personajes WHERE nombre = 'ARAGORN';```

- ### Comportamiento normal 
Si el usuario busca Aragorn, la consulta busca exactamente ese nombre.

- ### Ataque de inyección SQL
Si en lugar de un nombre legítimo, el usuario ingresa el siguiente texto malicioso en el campo de entrada:

    ´´´' OR '1'='1```

La consulta final que se ejecuta en la base de datos se transforma en:

    ´´´SELECT * FROM personajes WHERE nombre = '' OR '1'='1';```

Como la condición '1'='1' siempre es verdadera (TRUE), la base de datos interpretará que debe devolver todos los registros de la tabla personajes, exponiendo información confidencial o sensible.

Tanto PreparedStatement (en Java) como el uso de parametros seguros (en Python) evitan la inyección SQL porque separan estructura del codigo SQL de los datos proporcionados por el usuario.

**3. ¿Qué es una transacción y qué hace commit()? ¿Qué pasa en PyMySQL si no se llama?**

- ### ¿Qué es una transacción?

Una transacción incluye todas las sentencias que se ejecutan entre el inicio de la misma y una sentencia de finalización (como COMMIT o ROLLBACK). Garantizando que los cambios se realicen de forma adecuada y manteniendo la DB coherente.

- ### ¿Qué hace COMMIT?

La sentencia COMMIT confirma y guarda de forma permanente todos los cambios de de archivos realizados. Si se ejecuta correctamente, los cambios se graban en el disco.

- ### ¿Qué pasa en PyMySQL si no se llama?

Si realizas operaciones que modifican datos (como INSERT, UPDATE o DELETE) y no llamas explicitamente a commit(), los cambios se quedarán almacenados temporalmente de forma provisional no se guardarán de manera permanente en la base de datos una vez que la conexión termine.

**4. ¿Por qué hay que cerrar las conexiones? ¿Qué es try-with-resources en Java y with en Python?**

- ### ¿Por qué hay que cerrar las conexiones?

Si no se hace, el sistema operativo considerará que el recurso sigue en uso y la información necesaria para liberarlo se perderá.

- ### ¿Qué es try-with-resources en Java?

Es una sentencia try que declara uno o más recursos (objetos que deben cerrarse al terminar de usarse, implementando java.lang.AutoCloseable o java.io.Closeable). Asegura automáticamente que cada recurso sea cerrado al finalizar la sentencia, sin importar si esta se completa de manera normal o abrupta debido a una excepción.

- ### ¿Qué es with en Python?

Es una sentencia compuesta que permite la ejecución de codigo de inicialización y finalización alrededor de un bloque de codigo definido, gestiona el uso de contextos de manera segura.

### Fuentes citadas:

- https://www.ibm.com/docs/es/informix-servers/12.10.0?topic=started-what-is-jdbc

- https://docs.oracle.com/cd/E19509-01/820-5069/6ngg83nak/index.html

- https://entrenamiento-python-avanzado.readthedocs.io/es/latest/leccion6/mysql.html

- https://towardsdev.com/java-libraries-for-sql-injection-prevention-47c2018f5829

- https://www.ibm.com/docs/es/iis/11.5.0?topic=functions-commit-statement

- https://docs.python.org/3/reference/compound_stmts.html#the-with-statement

- https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html 

---

## <p align="center">Tema 6: MongoDB desde el codigo</p>

**1.Que es un (`ObjectId`) y que informacion guarda por dentro?**

Es el atributo primario del objeto que le da una identificación única para diferenciarlo de todos los objetos existentes y por existir, que guarda? guarda una combinacion de datos especificos donde la primera parte de 4 bytes guarda la fecha y hora de creacion, la segunda parte es un identificador aleatorio asignado por la maquina y el ultimo el contador incremental

**2.En que se parece un documento MongoDB a un (`HashMap`) de Java o a un (`dict`) de python ?**

se parecen en q todos funcionan bajo el modelo Clave-Valor, todos son muy flexibles y dinamicos a diferencia de sql con sus esquemas rigidos  y tambien soportan multiples tipos de datos para sus valores.

**3. ¿Qué diferencia hay entre MongoDB local, MongoDB Atlas y mongoplayground.net? ¿Cuál de los tres puede usar un programa en Java o Python?**

MongoDB local es el motor de la base de datos q se instala y se ejecuta directamente en una computadora con un servidor local
MongoDB Atlas es la version de la nube que esta alojada en servidores de proovedores como AWS, etc. Se accede a la base de datos a traves de internet mediante una URL de conexion.
mongoplayground.net es una web interactiva y de entorno de pruebas, sirve unicamente para probar y practicar en general.

Tanto el MongoDB local como el Atlas pueden usar un programa de Java y Python, el mongoplayground.net no ya que este es solo una interfaz grafica web para simular consultas.



## <p align="center">Tema 7: ORM </p>

**1.Que es un ORM?**

un ORM (Object Relational Mapping) es una forma de programar que permite interactuar con una base de datos utilizando un lenguaje de programacion como la programacion orientada a objetos en lugar de hacer consultas SQL directas basicamente funcionando como un puente entre estos dos

**2.Nombren uno de Java y uno de Python**

Para java uno de los ORM mas utilizados es el (`Java: Hibernate`) que es el ORM estandar y mas utilizado en el ecosistema Java/Jakarta EE.
Para phyton  es el  (`SQLAlchemy`)  o el (`Django ORM`) si se trabaja dentro del framework Django.

**3.Den una ventaja y una desventaja de usar un ORM en lugar de escribir el SQL a mano**

Una de sus mayores ventajas en su velocidad de desarrollo y su mantenibilidad ya que el codigo de la base de datos se escribe en el mismo lenguaje del proyecto donde se puede reducir el codigo repetitivo y ayuda a un desarrollo del CRUD mas rapido

Una de sus desventajas es que hay una perdida de rendimiento y de control en consultas ya muy complejas.




### Fuentes citadas:
- https://coddy.tech/learn/es/java/logic_and_flow/what_is_a_hashmap

- https://ed.team/blog/por-que-usar-un-orm-ventajas-y-desventajas

- https://www.mongodb.com/products/platform/atlas-database/features


