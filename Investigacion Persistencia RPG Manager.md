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


  

## <p align="center">Tema 6: MongoDB desde el codigo</p>

**1.Que es un (`ObjectId`) y que informacion guarda por dentro?**
Es el atributo primario del objeto que le da una identificación única para diferenciarlo de todos los objetos existentes y por existir, que guarda? guarda una combinacion de datos especificos donde la primera parte de 4 bytes guarda la fecha y hora de creacion, la segunda parte es un identificador aleatorio asignado por la maquina y el ultimo el contador incremental

**2.En que se parece un documento MongoDB a un (`HashMap`) de Java o a un (`dict`) de python ?**
se parecen en q todos funcionan bajo el modelo Clave-Valor, todos son muy flexibles y dinamicos a diferencia de sql con sus esquemas rigidos  y tambien soportan multiples tipos de datos para sus valores.

**3. ¿Qué diferencia hay entre MongoDB local, MongoDB Atlas y mongoplayground.net? ¿Cuál de los tres puede usar un programa en Java o Python?**
MongoDB local es el motor ded la base de datos q se instala y se ejecuta directamente en una computadora con un servidor local
MongoDB Atlas es la version de la nube que esta alojada en servidores de proovedores como AWS, etc. Se accede a la base de datos a traves de internet mediante una URL de conexion.
mongoplayground.net es una web interactiva y de entorno de pruebas, sirve unicamente para probar y practicar en general.

Tanto el MongoDB local como el Atlas pueden usar un programa de Java y Python, el mongoplayground.net no ya que este es solo una interfaz grafica web para simular consultas.















### Fuentes citadas:
- https://builtin.com/articles/dependency-injection (Trad. Español)

- https://medium.com/@Masoncoding/programming-to-an-interface-not-an-implementation-024d01815070 (Trad. Español)
  
- https://coddy.tech/learn/es/java/logic_and_flow/what_is_a_hashmap

- https://www.mongodb.com/products/platform/atlas-database/features


