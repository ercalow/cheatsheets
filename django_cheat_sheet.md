# Django Cheat Sheet

## Sections
- [Some nice links](#some+nice+links)
- [Start the project + app](#start-the-project-+-app)
- [Views stuff](#views+stuff)
- [Templates stuff](#templates-stuff)
- [Models stuff](#models-stuff) and [CRUD operations](#django-orm-crud-operations)
- [Admin UI](#admin-ui)

## Some nice links
- [Dillinger](https://dillinger.io/) for makin markdown files
- [Github django cheat sheet](https://github.com/lucrae/django-cheat-sheet)
- [Eric's beloved cheat sheet](https://dev.to/ericchapman/my-beloved-django-cheat-sheet-2056)
- [Django Girls tutorial](https://tutorial.djangogirls.org/en/) - has installation for everything, bootstrap, css, deploying on gitub/pythonanywhere
- [Giant one](https://github.com/entirelymagic/Programming-Cheat-Sheet#41-cheat-sheet) that includes python, docker, other stuff
- [Installation on windows](https://www.stanleyulili.com/django/how-to-install-django-on-windows/ ) including venv
- [Puttin on Github](https://www.javatpoint.com/django-deploy-on-github)

## Start the project + app
In shell:
- `cd` to root project folder
- Do virtual environment here
    - To make it: `python -m venv venv`
    - To activate: `venv/Scripts/activate`
    - Install stuff: `pip install django`
    - To deactivate: `deactivate`
- Make new project: `django-admin startproject projectName`
- Make new app: `cd` to project folder, then `python manage.py startapp newApp`

Set up the folders:
```
project/
    templates/
        appNameFolder
    static/
        appNameFolder
```

In `settings.py`:
- Add app to `INSTALLED_APPS` - just `'appName'`
- Under “build paths” add: 
    ```python
    TEMPLATES_DIR = BASE_DIR / 'templates'
    STATIC_DIR = BASE_DIR / 'static'
    ```
- Under `TEMPLATES`, `DIRS`: add `TEMPLATES_DIR`
- Under static section: add `STATICFILES_DIRS = [STATIC_DIR]`

Useful shell commands
- To run the server: `python manage.py runserver`
- Check version of django: `python -m django -version`
- Install stuff: `pip install django` atom + `atom-django` and `autocomplete-pytho`n packages, mysql server, mysql workbench, `mysqlclient` (to python)

## Views stuff
Views = logic that handles everything, like the templates and models!

What views look like!
```python
def renderTemplate(request):
    return render(request, 'folder/index.html', dataDict)
```
Notice: the folder and html file are what is in your file directory not the URL’s

URLs.py: need to map each view to a URL
- `from productApp import views`
- Then add to `urlPatterns`: a thing for each view
```python
path('', views.index),
path('electronics/', views.electronics),
path('toys/', views.toys),
path('shoes/', views.shoes),
```
You can do a `urls.py` for each app if you want - have to remove `admin` and also import the apps into the project’s `urls.py`.

## Templates stuff
- Templates = HTML/UI/rendering!
- In atom, if you start typing `html` then hit tab, Atom will do a template for you

Template tags
- At the top of every html file, under `doctype`: `{% load static %}`
- To use data passed by views: use `{{data}}`
    - If the dict looks like `{"name": "Dexter"}` then use `{{name}}` to display `Dexter`.
- To use Python logic in your html: `{% stuff here %`} tags
    - `{% if True %}` `{% else %}` `{% endif %}`
    - `{% for emp in employees %}` `{% endfor %}`
    - `{%csrf_token%}` `{%x extends y%}`

Do CSS
- You made a folder called “static” under the project directory - create app-specific folders (same name as the app), then inside create “css” folder, then put css files there
    - Make an images file while you’re at it
- In `settings.py`
    - In the paths section: `STATIC_DIR = BASE_DIR / 'static'`
    - In the static files section: they have to be called these variables
        ```python
        STATIC_URL = 'static/'
        STATICFILES_DIRS = [STATIC_DIR]
        ```
- In the template
    - `{% load static %}` at the top
    - In the head: `<link rel="stylesheet" href="{% static 'css/products.css' %}">`

HTML reference
- Images:  `img src="{% static 'images/dester.png' %}"/>`
- Tables: display a database
```html
{% if employees %}
<table>
    <thead> 
        <th> Headers </th>
        <th> Go Here</th>
    </thead>
    {% for emp in employees %}
        <tr> 
            <td> {{emp.firstName}} </td>
            <td> {{emp.lastName}} </td>
    </tr>
    {% endfor %}
</table>
{% endif %}
```

## Models stuff
A Model does database stuff!

Get MySQL going, create the database
- MySQL should be running automatically
 - Open workbench 
    - Click “create new sql tab” (top left)
    - Run these commands: 
        ```sql
        create database employeedb;
        use employeedb;
        ```
- In `settings.py`, under databases
    ```python
    'ENGINE': 'django.db.backends.mysql',
    'NAME': 'employeedb',
    'USER': 'root',
    'PASSWORD': 'passwordgoeshere'
    ```
    - Change your password [here!](https://dev.mysql.com/doc/mysql-windows-excerpt/8.0/en/resetting-permissions-windows.html)
- Validate in shell (don’t have to be running the server)
    - `python manage.py shell`
    - `from django.db import connection`
    - `c = connection.cursor()` <- If this exists, you’re good - if not you have to fix it then restart the shell
- To see what MySQL is calling your table: run `show tables` and it's at the bottom.
- Then you can see what's in it: `select * from table_name`

Create the Model
- Go to `models.py` in the app
- Make a class that inherits from `models.Model` - this is where you define what’s gonna go in the database - what the columns are
    ```python
    class Employee(models.Model):
    firstName = models.CharField(max_length=30)
    salary = models.FloatField()
    ```

Migrate! (Do every time you change the model or db)
- prepare to migrate: `python manage.py makemigrations`
    - That creates a migrations folder under the app + sql code
    - If you wanna see stuff (not necessary): `python manage.py sqlmigrate empApp 0001`
- Migrate/execute the SQL and stuff: `python manage.py migrate`
- Then you can go to mysql workbench and play around with the data
    - Select the db: `use employeedb;`
    - `show tables;` (shows you all the admin tables, the one you made is at the end) - it will be called something like empApp_employee)
    - See everything in the database: `select * from empApp_employee;`
To insert stuff in mysql workbench: `insert into empApp_employee values(1, 'bharath', 'thippireddy', 100000,'a@b.com')`
    - The first value is the id, which you don’t normally have to do, but if you don’t, you have to provide all the column names

Use the model in a view
- In `views.py:` `from empApp.models import Employee`
- To do `select *` but in python: `employees = Employee.objects.all()`
- To get the database from the view to the template:
    ```python
    empDict = {'employees': employees}
    return render(request, 'empApp/employees.html', empDict)
    ```
    - Then you can use `{{key.attribute}}` tags to use info in the database

### Django ORM CRUD operations
(object relational mapping = querying with python syntax!)

Getting started:
- You gotta go to mysql workbench and `use ___` your database
- In shell, you gotta `cd` to the project directory
- Open shell: `python manage.py shell`
- Import model: `from empApp.models import Employee`

General
- When you’ve got a query object, use `qs.query` to see the SQL query

Read operations
- Everything, as “query set”: `qs = Employee.objects.all()`
- Just one entry, selected by `id`: `emp = Employee.objects.get(id=1)`
    - You can access the entries: `emp.firstName`
- Filtering: `emps = Employee.objects.filter(salary__gt=5000)`
    - All the filters: lt, gt, lte, gte, contains (case sensitive), icontains (not), in, startswith, endswith
    - How to use “in”: `employee.objects.filter(id__in=[1,2])`
- Or = `|`
    - Long way to use Or: `emps = Employees.objects.filter(firstName__startswith=’bh’) | Employee.objects.filter(lastName__startswith=’xy’)`
    - Short way:
        - Import Q: `from django.db.models import Q`
        - Use Q: `emps = Employee.objects.filter(Q(firstName__startswith=’bh’) | Q(lastName__startswith=’xy’))`
- And = `&`. Has 2 options
    - Use Q: `emps = Employee.objects.filter(Q(firstName__startswith=’bh’) & Q(lastName__startswith=’xy’))`
    - Or don’t: `emps = Employee.objects.filter(firstName__startswith=’bh’, lastName__startswith=’xy’)`
- Not = `exclude()` function
    - Example: `emps = Employee.objects.exclude(salary__gt=5000)`
- Selecting columns:
    - If you only want a few columns: `emps = Employee.objects.all().values_list(‘firstName’, ‘salary’)`
    - Or do this (it’s the same): `emps = Employee.objects.all().values(‘firstName’, ‘salary’)`
    = To include the `id` field: `emps = Employee.objects.all().only(‘firstName’, ‘salary’)`
- Aggregate functions (avg, max, min, sum, count)
    - Import: `from django.db.models import Avg, Sum, Max, Min, count`
    - Use: `avg = Employee.objects.all().aggregate(Avg(‘salary’))`
- Order By (sort function)
    - Get the records in ascending order by salary: `emps = Employee.objects.all().order_by('salary')`
    - Descending order - use a minus sign: `emps = Employee.objects.all().order_by('-salary')`
    - Get a particular record, like the first one: `emps = Employee.objects.all().order_by('salary')[0]`
        - For a range, use `[0:3]`
    - If the column is text, it will sort alphabetically
        - To ignore case though: `from django.db.models.functions import Lower`
        - Make lowercase: `emps = Employee.objects.all().order_by(Lower('firstName'))`
        - The entry it returns won’t be all lowercase - it’ll be the same
    - Make them randomly ordered! `ex = Exercise.objects.order_by('?')`

Create operations
- Check how many entries in the db: `Employee.objects.all().count()`
- One way to create:
    - Make a new one: `e = Employee(firstName’John’, lastName=’Bailey’, salary=10000)`
    - Put it in the db: `e.save()`
- Second way: `Employee.objects.create(firstName=’Bob’, lastName=’Bailey’, salary=20000)`
- Bulk create: just give it a list of Employee objects: `Employee.objects.bulk_create([Employee(firstName='Jeff', lastName='Ferguson', salary=5000), Employee(firstName='Guy', lastName='Guyerson', salary=500000)])`

Delete operations
- Select the one you want to delete: `e = Employee.objects.get(id=1)`
- Then delete: `e.delete()`
- Bulk delete, by filter: 
    - First, filter: `qs = Employee.objects.filter(salary__gt=5000)`
    - Can check the count: `qs.count()`
    - Then delete: `qs.delete()`
- Delete all: `Employee.objects.all().delete()`

Update operations
- With Django:
    - Fetch the record: `emp = Employee.objects.get(id=2)`
    - Update its first name: `emp.firstName = 'Bharath'`
    - Save to database: `emp.save()`
- Do it in sql workbench UI: 
    - Run command: `select * from empApp_employee;`
    - Click and manually change the field
    - Click outside of it
    - Click apply

## Admin UI
- See it: `go to localhost:8000/admin/`
- To make a login, in shell: `python manage.py createsuperuser` (skip the email)

Adding model to the Admin UI
- In `admin.py`
    - Imports: `from empApp.models import Employee`
    - Make this class, which lets you control how the database is displayed in the UI - like which fields are shown:
    ```python
    class EmployeeAdmin(admin.ModelAdmin):
        list_display = ['firstname', 'lastname'] # These are the fields you want to show
    ```
    - Register it below that: `admin.site.register(Employee, EmployeeAdmin)`
- Now, you can do all CRUD operations from the UI (create read update delete)
- Changes are reflected in mySQL workbench too


