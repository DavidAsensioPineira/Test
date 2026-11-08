# Funcionament d'un servidor Django

Aquest document explica com funciona una aplicació web feta amb **Django**, utilitzant com a exemple el projecte `ProjecteTest` i l'aplicació `WebApp`.

## 1. Què és Django?

Django és un framework web escrit en Python. Proporciona eines per crear aplicacions web sense haver de programar des de zero aspectes habituals com ara l'encaminament d'URL, les respostes HTTP, les plantilles HTML, l'accés a bases de dades, l'autenticació i el panell d'administració.

Django segueix una arquitectura sovint descrita com a **MVT**:

- **Model**: representa les dades i la seva relació amb la base de dades.
- **View (vista)**: rep una petició, executa la lògica necessària i prepara la resposta.
- **Template (plantilla)**: defineix com es presenta la informació, normalment en HTML.

## 2. Engegar el servidor

Des del terminal, situat a la carpeta on hi ha `manage.py`, executem:

```powershell
python manage.py runserver
```

Per defecte, Django inicia un servidor de desenvolupament a:

`http://127.0.0.1:8000/`

També podem indicar una altra adreça o port, per exemple:

```powershell
python manage.py runserver 0.0.0.0:8000
```

Aquesta segona forma permet escoltar en totes les interfícies de xarxa disponibles, però perquè altres equips hi puguin accedir cal tenir en compte el tallafoc, la xarxa i la configuració `ALLOWED_HOSTS`.

El servidor de desenvolupament és pràctic per programar i provar l'aplicació. **No s'ha d'utilitzar com a servidor de producció**.

## 3. Estructura del projecte

Una estructura senzilla com la que estem utilitzant és:

```text
ProjecteTest/
│
├── manage.py
│
├── ProjecteTest/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
│
└── WebApp/
    ├── migrations/
    ├── static/
    │   └── WebApp/
    │       └── css/
    │           └── styles.css
    ├── templates/
    │   └── WebApp/
    │       └── index.html
    ├── urls.py
    ├── views.py
    ├── models.py
    └── admin.py
```

Els elements principals són:

- `manage.py`: eina de línia d'ordres per administrar el projecte.
- `ProjecteTest/settings.py`: configuració general del projecte.
- `ProjecteTest/urls.py`: encaminament principal de les URL.
- `WebApp/urls.py`: encaminament propi de l'aplicació.
- `WebApp/views.py`: funcions o classes que processen les peticions.
- `templates/`: plantilles HTML.
- `static/`: fitxers estàtics com CSS, JavaScript i imatges.
- `models.py`: definició de les dades de l'aplicació.
- `migrations/`: canvis versionats de l'estructura de la base de dades.

Un projecte Django pot contenir diverses aplicacions. Cada aplicació agrupa una part funcional del sistema i pot tenir les seves pròpies vistes, URL, models, plantilles i fitxers estàtics.

## 4. Què passa quan un navegador demana una pàgina?

Quan l'usuari escriu `http://127.0.0.1:8000/`, el procés simplificat és:

1. El navegador envia una petició HTTP al servidor Django.
2. Django consulta les rutes definides a `ProjecteTest/urls.py`.
3. La ruta principal delega l'encaminament a `WebApp/urls.py`.
4. La ruta de l'aplicació indica quina vista s'ha d'executar.
5. La vista prepara les dades i selecciona una plantilla.
6. Django processa la plantilla HTML.
7. El servidor retorna una resposta HTTP al navegador.
8. El navegador interpreta l'HTML i sol·licita, si cal, els fitxers CSS, JavaScript i les imatges.

Esquema:

```text
Navegador
    |
    | Petició HTTP: /
    v
ProjecteTest/urls.py
    |
    | include('WebApp.urls')
    v
WebApp/urls.py
    |
    | views.index
    v
WebApp/views.py
    |
    | render(...)
    v
WebApp/templates/WebApp/index.html
    |
    | Resposta HTML
    v
Navegador
    |
    | Sol·licitud de CSS
    v
WebApp/static/WebApp/css/styles.css
```

## 5. Configuració de les URL

Al fitxer `ProjecteTest/urls.py` definim les rutes generals del projecte:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('WebApp.urls')),
]
```

La línia `include('WebApp.urls')` indica que les peticions que coincideixin amb aquesta ruta es continuaran resolent a les URL de l'aplicació `WebApp`.

Al fitxer `WebApp/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

Aquesta configuració associa la ruta buida de l'aplicació amb la vista `index`.

És important no incloure el mateix fitxer d'URL dins d'ell mateix: això provocaria una recursió d'encaminament.

## 6. Les vistes: views.py

Una vista és el punt on Django executa la lògica necessària per respondre una petició.

Exemple a `WebApp/views.py`:

```python
from django.shortcuts import render

def index(request):
    return render(request, 'WebApp/index.html')
```

- `request` conté informació sobre la petició del navegador.
- `render()` carrega la plantilla, la processa i crea una resposta HTTP.
- El resultat és el contingut que Django envia al navegador.

Més endavant, la vista també pot consultar models, validar formularis, comprovar permisos o enviar dades a la plantilla.

## 7. Plantilles HTML

Les plantilles permeten separar la presentació HTML de la lògica Python.

Exemple de `WebApp/templates/WebApp/index.html`:

```html
{% load static %}

<!DOCTYPE html>
<html lang="ca">
<head>
    <meta charset="UTF-8">
    <title>El meu primer Django</title>
    <link rel="stylesheet" href="{% static 'WebApp/css/styles.css' %}">
</head>
<body>
    <h1>Hola! Benvingut a Django</h1>
    <p>Aquesta és la meva primera pàgina web.</p>
</body>
</html>
```

Django disposa d'un llenguatge de plantilles que permet inserir variables, executar estructures condicionals i repetir contingut.

Per exemple, una vista pot enviar una variable:

```python
def index(request):
    context = {'nom': 'David'}
    return render(request, 'WebApp/index.html', context)
```

I la plantilla la pot mostrar així:

```html
<h1>Hola, {{ nom }}!</h1>
```

## 8. Fitxers estàtics: CSS, JavaScript i imatges

Els fitxers estàtics són recursos que el navegador descarrega, però que no necessiten ser generats per una vista en cada petició. Alguns exemples són els fulls d'estil CSS, els scripts JavaScript, les icones i les imatges.

En el nostre projecte, el CSS és a:

`WebApp/static/WebApp/css/styles.css`

Exemple:

```css
body {
    font-family: Arial, sans-serif;
    background-color: #f0f2f5;
    margin: 30px;
}

h1 {
    color: #2563eb;
}
```

A la plantilla cal carregar l'etiqueta `static`:

```django
{% load static %}
```

I referenciar el fitxer amb:

```html
<link rel="stylesheet" href="{% static 'WebApp/css/styles.css' %}">
```

Així evitem escriure rutes fixes i deixem que Django construeixi la URL correcta del recurs estàtic.

## 9. El fitxer settings.py

`settings.py` concentra la configuració del projecte. Entre altres opcions, hi trobem:

- `INSTALLED_APPS`: aplicacions que formen part del projecte.
- `MIDDLEWARE`: components que intervenen en el processament de les peticions i respostes.
- `TEMPLATES`: configuració del motor de plantilles.
- `DATABASES`: configuració de les bases de dades.
- `STATIC_URL`: prefix de les URL dels fitxers estàtics.
- `DEBUG`: activa o desactiva informació de depuració.
- `ALLOWED_HOSTS`: dominis o noms d'amfitrió autoritzats.

Perquè Django reconegui l'aplicació, cal incloure-la a `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    # Aplicacions de Django...
    'WebApp',
]
```

## 10. Models i base de dades

Els models defineixen les dades de l'aplicació mitjançant classes Python. Django pot convertir aquests models en taules de la base de dades.

Exemple simplificat a `WebApp/models.py`:

```python
from django.db import models

class Client(models.Model):
    nom = models.CharField(max_length=100)
    correu = models.EmailField()

    def __str__(self):
        return self.nom
```

Quan canviem els models, habitualment executem:

```powershell
python manage.py makemigrations
python manage.py migrate
```

- `makemigrations` crea els fitxers que descriuen els canvis.
- `migrate` aplica aquests canvis a la base de dades.

En una instal·lació nova, Django utilitza SQLite per defecte, però també es pot configurar per treballar amb altres sistemes de bases de dades.

## 11. Servidor de desenvolupament i servidor de producció

Durant el desenvolupament, `python manage.py runserver` és suficient per provar l'aplicació localment. Inclou funcions útils per programar, però no està dissenyat per suportar un servei real a Internet.

En producció, habitualment s'utilitza un servidor d'aplicacions compatible amb **WSGI** o **ASGI**, sovint darrere d'un servidor intermediari com Nginx o d'una plataforma d'allotjament gestionada.

Abans de publicar una aplicació cal, entre altres coses:

- Desactivar la depuració amb `DEBUG = False`.
- Configurar correctament `ALLOWED_HOSTS`.
- Protegir les claus i credencials; no publicar-les al repositori.
- Configurar HTTPS.
- Gestionar adequadament els fitxers estàtics i els fitxers pujats pels usuaris.
- Configurar còpies de seguretat i una base de dades adequada.
- Seguir la llista de desplegament oficial de Django per a la versió utilitzada.

Els fitxers `wsgi.py` i `asgi.py` són els punts d'entrada perquè els servidors de producció compatibles puguin comunicar-se amb l'aplicació Django. No són, per si sols, servidors web.

## 12. Resum de responsabilitats

| Element | Responsabilitat |
|---|---|
| `manage.py` | Administrar i executar ordres del projecte |
| `settings.py` | Configuració general |
| `urls.py` | Decidir quina vista correspon a cada URL |
| `views.py` | Processar la petició i preparar la resposta |
| `models.py` | Definir i gestionar les dades |
| `templates/` | Construir la presentació HTML |
| `static/` | Emmagatzemar CSS, JavaScript i imatges |
| `wsgi.py` / `asgi.py` | Punt d'entrada per al servidor d'aplicacions |

## 13. Idea clau

Django coordina diferents peces. Les URL indiquen quina vista s'ha d'executar; la vista aplica la lògica i, si cal, consulta els models; la plantilla genera l'HTML; i els fitxers estàtics aporten l'estil i altres recursos del navegador.

Aquesta separació facilita mantenir el codi, ampliar l'aplicació i treballar amb diverses aplicacions dins d'un mateix projecte.

---

Document introductori basat en l'estructura de l'exemple `ProjecteTest` / `WebApp`. Per a desplegaments reals, consulta també la documentació oficial de Django: https://docs.djangoproject.com/
