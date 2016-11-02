

DROPZONEJS + DJANGO: HOW TO BUILD A FILE UPLOAD FORM
-----
originally from - https://amatellanes.wordpress.com/2013/11/05/dropzonejs-django-how-to-build-a-file-upload-form/

DropzoneJS is an open source library that provides drag’n’drop file uploads with image previews. Moreover, the new version 2.0 DropzoneJS no longer depends on jQuery so it’s great.

Throughout this tutorial, we’ll see how building a file upload form using DropzoneJS and the backend will be handled by Django.


We’ll assume you have Python and Django installed already, but if you haven’t I recommend you seeing the official documentation at the Django website. This tutorial is written for Django 1.5 and Python 2.x.

Creating the project
---
We’ll create a project running the following command from the command line:

---
```python
$ django-admin.py startproject mysite 
```
This will create a mysite directory in our current directory. Now, we’ll move in the same directory as manage.py and type the following command to create a new app:

```python
$ python manage.py startapp main 
```
Now, we’ll edit mysite/settings.py and we’ll modify the next variables to set up the database (we’ll use SQLite3) and include main app that we’ve just created:
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': 'database.sqlite'
        'USER': '',
        'PASSWORD': '',
        'HOST': '',
        'PORT': '',
    }
}
 
INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'main',
)
```
Defining the URL dispatcher
---
Now, we’ll create and edit main/urls.py file so it looks loke this:

main / urls.py
```python
from django.conf.urls import patterns, include, url
 
 
urlpatterns = patterns('',
    url(r'^$', 'main.views.home', name='home'),
)
```
Now we’ll edit the mysite/urls.py to add a new namespace to our root URLconf:

mysite / urls.py
```python
from django.conf.urls import patterns, include, url
 
 
urlpatterns = patterns('',
    url(r'^$', include('main.urls', namespace="main", app_name="main")),
)
```
Creating the model
---
In our simple app, we’ll create only one model in main/models.py file: UploadFile:

main / models.py
```python
from django.db import models
 
 
class UploadFile(models.Model):
    file = models.FileField(upload_to='files/%Y/%m/%d')
```
We’ve defined a file-upload field. We’ll use the required argument upload_to where we’ll select a local filesystem path that will be appended to our MEDIA_ROOT setting to determine the value of the file url. We’ll use strftime() formatting, which will be replaced by the date of the file upload. With this configuration, the uploaded files will be saved in mysite/files/2013/11/05 today.

Creating the form
---
The next step is defining a form which validate itself and display itself as HTML. We’ll create main/forms.py file and then we’ll create our form from UploadFile model:

main / forms.py
```python
from django import forms
 
from models import UploadFile
 
 
class UploadFileForm(forms.ModelForm):
     
    class Meta:
        model = UploadFile
```
Creating the view
---
Let’s write the view where the uploaded file will be saved. This view handling the form will receive the file data in request.FILES, which is a dictionary containing a key for each FileField (or ImageField, or other FileField subclass) in the form. So the data from the above form would be accessible as request.FILES['file'].

Note that request.FILES will only contain data if the request method was POST and the that posted the request has the attribute enctype=”multipart/form-data”. Otherwise, request.FILES will be empty.

main / views.py
```python
from django.http import HttpResponseRedirect
from django.template import RequestContext
from django.core.urlresolvers import reverse
from django.shortcuts import render_to_response
 
from forms import UploadFileForm
from models import UploadFile
 
 
def home(request):
    if request.method == 'POST':
        form = UploadFileForm(request.POST, request.FILES)
        if form.is_valid():
            new_file = UploadFile(file = request.FILES['file'])
            new_file.save()
 
            return HttpResponseRedirect(reverse('main:home'))
    else:
        form = UploadFileForm()
 
    data = {'form': form}
    return render_to_response('main/index.html', data, context_instance=RequestContext(request))
```
Creating the template and using DropzoneJS
---
First, we’ll create a directory called templates in our main directory. Within the templates directory we have just created, create another directory called main, and within that create a file called index.html.

We’ll download dropzone.js and the css/dropzone.css, images/spritemap.png and images/spritemap@2x.png as well from the downloads folder. Now, we’ll include all this file in main/static/main directory. In this point, we should have next project structure:
```
├── database.sqlite
├── main
│   ├── forms.py
│   ├── __init__.py
│   ├── models.py
│   ├── static
│   │   └── main
│   │       ├── css
│   │       │   ├── basic.css
│   │       │   ├── dropzone.css
│   │       │   └── stylus
│   │       │       ├── basic.styl
│   │       │       └── dropzone.styl
│   │       ├── images
│   │       │   ├── spritemap@2x.png
│   │       │   └── spritemap.png
│   │       └── js
│   │           ├── dropzone-amd-module.js
│   │           ├── dropzone-amd-module.min.js
│   │           ├── dropzone.js
│   │           └── dropzone.min.js
│   ├── templates
│   │   └── main
│   │       └── index.html
│   ├── urls.py
│   ├── views.py
├── manage.py
└── mysite
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```
DropzoneJS will upload automatically the dropped files, so using JavaScript we will let build a queue and then upload all files with a single button click. You can learn more about hou to use DropzoneJS here. Put the following code in index.html template:
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>Upload a file in Django 1.5 using Dropzone.js</title>
        {% load staticfiles %}
        <link href="{% static 'main/css/dropzone.css' %}" type="text/css" rel="stylesheet"/>
    </head>
    <body>
 
        <!-- IMPORTANT enctype attribute! -->
        <form class="dropzone" action="{% url "main:home" %}" method="post" enctype="multipart/form-data">
            {% csrf_token %}
        </form>
        <button id="submit-all">
            Submit all files
        </button>
 
        <script src="{% static 'main/js/dropzone.js' %}"></script>
        <script type="text/javascript">
            Dropzone.options.myDropzone = {
 
                // Prevents Dropzone from uploading dropped files immediately
                autoProcessQueue : false,
 
                init : function() {
                    var submitButton = document.querySelector("#submit-all")
                    myDropzone = this;
 
                    submitButton.addEventListener("click", function() {
                        myDropzone.processQueue();
                        // Tell Dropzone to process all queued files.
                    });
 
                    // You might want to show the submit button only when
                    // files are dropped here:
                    this.on("addedfile", function() {
                        // Show submit button here and/or inform user to click it.
                    });
 
                }
            };
        </script>
    </body>
</html>
```
To check our app we’ll create the tables of each applications in the database running the following command:

'''python
$ python manage.py syncdb
```
Now, run the Django server with the following command:

'''python
$ python manage.py runserver
'''
Now that the server’s running, visit http://127.0.0.1:8000/ with your Web browser. You’ll see a page as below:

dropzonejs-django-web

Download source | Github

More info | Dropzone.js & Django documentation
