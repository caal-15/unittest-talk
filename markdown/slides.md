layout: true

---
class: middle, center, general
# Testing en Python
## Una introducción a Python Unittest
Una pequeña charla por [Carlos González](http://caal-15.github.io)

Síguela en [caal-15.github.io/unittest-talk](http://caal-15.github.io/unittest-talk)
.footnote[.alt-link[[Documentación de Python Unittest](https://docs.python.org/3/library/unittest.html)]]

---
class: center, middle

# Antes de Empezar

---
class: left, middle

# Pruebas Automatizadas

.big[
  La meta principal de las _Pruebas Automatizadas_ es conseguir un código
  libre de errores por parte del programador, son la base del __Test Driven
  Development__ (__TDD__).
]

---
class: left, middle

# Una Nota Sobre TDD

.big[
* __TDD__ es una _metodología_ de desarrollo que ha ganado bastante tracción por
su premisa simple pero efectiva: _prueba primero, codifica después_.

* Esto promueve _buenas prácticas_ de desarrollo, y puesto en práctica
correctamente garantiza que ningún defecto crítico salga a producción.

* Esta _metodología_ se adapta bien con nuevas _tecnologías_ como
__Continuous Integration__ y __Contiuous Deployment__.
]

---
class: left, middle

# ¿Por qué usar Pruebas Automatizadas?

* .big[Porque somos _humanos_.]

* .big[Facilita el mantinimiento del código e integración de código nuevo.]

* .big[Evita tirarse el ambiente de producción y mejor aún __tener que explicar
  por qué no funciona el ambiente de producción!__.]


---
class: center, middle

# Pruebas Automatizadas en Python, Conceptos Básicos

---
class: left middle

# Unittest

.big[
* En Python, la herramienta por defecto para realizar pruebas es _unittest_,
hace parte de las librerías __estándar__ de Python y está inspirada en
_JUnit_.

* Como toda librería de testing soporta cuatro conceptos básicos a través de su
implementación: _fixtures_, _test cases_, _test suites_, y _test runners_.
]

.footnote[.alt-link[[JUnit para los curiosos...](https://junit.org/junit5/)]]

---
class: left middle

# Fixtures

.big[
* Es el procedimiento de _preparación_ previo a un __test__ o
__conjunto de tests__, esto puede incluir acciones como instanciar una clase,
o establecer una conexión a una base de datos

* En _unittest_, se implementan las funciones __setUp__ y __tearDown__ de una
__subclase__ de la clase __TestCase__, y proveen una forma sencilla de programar
tareas pre y post test (se ejecutan antes y después de _cada_ test de la clase).

* Si se desea que las tareas pre y post test solo corran _una vez_ antes de
__todas__ y las pruebas y una vez después de __todas__ las pruebas se pueden
implemenentar __setUpClass__ y __tearDownClass__.
]

---
class: left middle

# Test Case

.big[
* Es un caso particular de prueba, una __prueba unitaria__ como tal que pretende
evaluar un caso _específico_ del software que se pone a prueba.

* En _unittest_, se implementan la clase como funciones de una __subclase__ de
la clase __TestCase__ que utilizan funciones __assert__ para probar las
condiciones deseadas.
]

---
class: left middle

# Test Suite

.big[
* Es simplemente una _colección_ de __Test Cases__, cada subclase de la clase
__TestCase__ es un __Test Suite__.

* Una conjunto de __Test Suites__ también es un __Test Suite__.
]

---
class: left middle

# Test Runner

.big[
* Es el encargado de _correr_ las preubas y mostrarle los _resultados_ al
usuario.

* En el caso de _unittest_ suele usarse a través de __línea de comando__:
]

```bash
python -m unittest
```

---
class: center, middle

# Poniéndolo todo junto, pruebas básicas


---
class: left, middle

# Un ejemplo sencillo

```python
import unittest
import os
from mymodule import create_sqlite_db
from myothermodule import MyUserClass

class MyUserClassTestClass(unittest.TestCase):

  def setUp(self):
    self.db = create_sqlite_db('/tmp/test.db')
    self.db.store_user(
        {'id': 1, 'name': 'Pablo'})
    self.subject = MyUserClass(self.db)
  ...
```

---
class: left, middle

```python
  ...
  def test_when_user_exists_it_returns_it_correcly(self):
    user = self.subject.get_from_db(1)
    self.assertEqual(user['id'], 1)
    self.assertEqual(user['name'], 'Pablo')

  def test_when_user_doesnt_exist_it_returns_none(self):
    user = self.subject.get_from_db(2)
    self.assertIsNone(user)

  def tearDown(self):
    os.remove('/tmp/test.db')
```

---
class: left, middle

# Y para correr los test

.big[Asumiendo que nuestro archivo se llame _tests.py_]

```bash
python -m unittest tests.py
```

---
class: left, middle

# Organizando el directorio

.big[
  El commando _python -m unittest_ puede descubrir y ejecutar nuestras pruebas
  automáticamente incluso si están en __archivos separados__ si organizamos
  nuestro directorio:
]

```bash
myproject/
..tests/
....__init__.py
....test_module_1.py
....test_module_2/
......__init__.py
......test_submodule_1.py
```

.footnote[Esto es enteramente opcional, se puede correr _python -m unittest_
discover si se tiene otra estructura.]

---
class: left, middle

# Ventajas

.big[
* Podemos ejecutar _todas_ las pruebas de todos nuestros módulos con un sólo
comando.

* Podemos tener la _granularidad_ que queramos al correr las pruebas, incluso
correr un solo __test case__, por ejemplo para ejecutar una sola de nuestras
clases:
]

```bash
python -m unittest \
tests.test_module_1.MyTestClass
```

---
class: center, middle

# Mocks

---
class: left, middle

# ¿Cuál es su propósito?

.big[
En ocasiones necesitamos _modificar_ el comportamiento, o _verificar_ cómo se
han usado otras clases o funciones de las que puedan depender nuestros tests,
para este fin _unittest_ nos ofrece la librería __Mock__ (unittest.mock).
]

.footnote[.alt-link[[Documentación de Mock](https://docs.python.org/3/library/unittest.mock.html)]]


---
class: left, middle

# Usos básicos: autospec

.big[La idea básica es verificarhacer un Mock de una clase con sus atributos
para _verificación_ o _alteración_ de su funcionamiento:]

```python
from unittest.mock import Mock
...
# MyClass has method resist
MockClass = Mock(spec=MyClass)
thing = MyOtherClass(MockClass())
# MyOtherClass.desist calls MyClass.resist
thing.desist()
MockClass.resist.assert_called_once()
...
```

---
class: left, middle

# Usos básicos: patch

.big[Sirve para hacer modificar atributos o comportamientos de _clases_ o _métodos_
(o métodos de clases):]

```python
from unittest.mock import patch
...
with patch.object(MyClass, 'method') as m:
  m.return_value = 3
  thing = MyClass
  self.assertEqual(thing.method(), 3)
  m.assert_called_once()
...
```

---
class: left, middle:

# Cheatsheet

.big[
* Hereda siempre de __unittest.TestCase__ al crear tu Subclase.

* Implementa __setUp__ para preparaciones previas _fixtures_.

* Todas tus pruebas (funciones de tu Subclase) deben empezar por _test_.

* Utiliza las funciones _assert_ heredadas de __unittest.TestCase__
(ej: __self.asssertEqual__) para las condiciones de tu prueba.

* Implementa __tearDown__ para ejecutar acciones de limpieza después de las
pruebas.

* _Organiza_ el código de tus pruebas como lo discutimos antes.

* Usa __python -m unittest__ para correr todas tus pruebas.
]

---
class: center, middle

# Ahora a practicar!, Muchas Gracias por su Atención.

.footnote[_Me pueden invitar una cerveza despúes de la charla_]
