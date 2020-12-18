# Live Project
## Introduction
For two weeks of my time at the Tech Academy, I worked with a team of other students on a Django Python project to develop apps for an interactive website for managing one's collections of things related to various hobbies. Working in the Django framework and in a team setting was a great opportunity for learning version control, managing deadlines, and adding requested features. I saw how a good developer works with what they have and know to make a quality product. Within this two week sprint, I was given user stories of increasing difficulty to complete inorder to increase our problem solving mindest. I worked on several [back end stories](#back-end-stories) and [front end stories](#front-end-stories) of an application about destinations to visit in Los Angeles. Being a student there, I have always wanted to keep track of the places I have been. So, this was the perfect opportunity to incorporate something I am passionate about, into a project where I gained real world experience and improved my skills. 

Below are descriptions and examples of some of the stories that I worked on within my Los Angeles Destination App.

# Back End Stories
* [Models and Forms](#models-and-forms)
* [Views Functions](#views-functions)

## Models and Forms
Here, I created a model for what information the user could enter into the database. To make it easier, I also included a model manager.
```python
from django.db import models

# limit choices for type and price
TYPE_CHOICES = [
    ('Site', 'Site'),
    ('Activity', 'Activity'),
    ('Food', 'Food'),
    ('Museum', 'Museum'),
]

PRICE_CHOICES = [
    ('Free', 'Free'),
    ('Under $25', 'Under $25'),
    ('$25 to $50', '$25 to $50'),
    ('$50 to $100', '$50 to $100'),
    ('$100+', '$100+'),
]

# model to create a new destination
class Destination(models.Model):
    name = models.CharField(max_length=200, default="", blank=False)
    type = models.CharField(max_length=60, choices=TYPE_CHOICES)
    description = models.TextField(max_length=300, default="")
    location = models.CharField(max_length=200)
    price = models.CharField(max_length=60, choices=PRICE_CHOICES)

    objects = models.Manager()

    def __str__(self):
        return self.name


```
Furthermore, I then created a model form in that utilizes all the fields from the model.
```python
from django.forms import ModelForm
from .models import Destination


class DestinationForm(ModelForm):
    class Meta:
        model = Destination
        fields = '__all__'
```
To ensure that the request to add another destination is not an attack on the server, I utilized the CSRF token to validate the form.
```Python 
{% extends 'LABucketListApp/LABucketListApp_base.html' %}
{% load static %}
{% block title %}Add Destination{% endblock %}
{% block header %}Add a Destination to the List {% endblock %}

{% block content %}
<!-- Add Destination Form -->

    <div class="formContainer">
        <form method="POST" action="{% url 'BucketList_create' %}">
            <div class="frmObject_container">
                <!-- Cross Site Request Forgery (csrf_token) protection -->
                {% csrf_token %}
                {{ form.non_field_errors }}
                {{ form.as_p }}
            </div>
            <div>
                <input type="submit"  value="Add Destination" name="Save_Item">
                <a href="{% url 'BucketList_home' %}"><button type="button" value="Cancel">Cancel</button></a>
            </div>
        </form>
    </div>

{% endblock %}
```

## Views Functions
The views page was vital to my application becuase it holds the functions that other pages call.

Here is the function that was needed in order to create a new destination in the database.
```Python
from django.shortcuts import render, redirect
from .forms import DestinationForm

# Add Destination to database
def bucketlist_create(request):
    form = DestinationForm(data=request.POST or None)
    if form.is_valid():
        form.save()
        return redirect('BucketList_home')
    context = {'form': form}
    return render(request, 'LABucketListApp/LABucketListApp_create.html', context)
```
This is the function needed to display all the desinations from the database. It utilizes a paginator that displays 6 destinations on a single page. 
```python
from django.shortcuts import render, redirect
from .forms import DestinationForm
from .models import Destination
from django.core.paginator import Paginator

# display all locations from database for index
def bucketlist_index(request):
    form = DestinationForm()
    destinations = Destination.objects.all()
    page = Paginator(destinations, 6)
    page_number = request.GET.get('page')
    page_obj = page.get_page(page_number)
    context = {'form': form, 'destinations': destinations}
    return render(request, 'LABucketListApp/LABucketListApp_index.html', {'page_obj': page_obj}, context)
```
From the index, the user can then select a specific location to display all the details of the destination, which called the below function.
```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Destination

# displays the details of each destination
def bucketlist_details(request, pk):
    destinations = get_object_or_404(Destination, pk=pk)
    context = {'destinations': destinations}
    return render(request, 'LABucketListApp/LABucketListApp_details.html', context)
```
These next two functions allowed the user to edit and delete a location.
```python
from django.shortcuts import render, redirect, get_object_or_404
from .forms import DestinationForm
from .models import Destination

# edit the destination
def bucketlist_edit(request, pk):
    destinations = get_object_or_404(Destination, pk=pk)
    form = DestinationForm(data=request.POST or None, instance=destinations)
    if request.method == "POST":
        if form.is_valid():
            form.save()
            return redirect('BucketList_details', pk)
    context = {'destinations': destinations, 'form': form}
    return render(request, 'LABucketListApp/LABucketListApp_edit.html', context)


# delete function
def bucketlist_delete(request, pk):
    destinations = get_object_or_404(Destination, pk=pk)
    if request.method == 'POST':
        destinations.delete()
        return redirect('BucketList_index')
    else:
        return redirect('BucketList_details', pk)
```

# Front End Stories
* [Base HTML Template](#base-html-template)
* [Template Inheritance](#template-inheritance)
* [Bootstrap Elements](#bootstrap-elements)

## Base HTML Template
This is the base HTML template that all other HTML files inherit from. I chose to include Bootstrap which makes adding elements to the application very easy. 

```html
{% load static %}
<!DOCTYPE html>

<html lang="en" >
	<head>
	    <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">

        <!-- Google Fonts -->
        <link rel="preconnect" href="https://fonts.gstatic.com">
        <link href="https://fonts.googleapis.com/css2?family=Playfair+Display&display=swap" rel="stylesheet">

        <!-- Bootstrap and CSS -->
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.3/dist/css/bootstrap.min.css" \
              integrity="sha384-TX8t27EcRE3e/ihU7zmQxVncDAy5uIKz4rEkgIXeMed4M0jlfIDPvg6uqKI2xXr2" crossorigin="anonymous">
        <link rel="stylesheet" type="text/css" href="{% static 'LABucketListApp/css/layout.css' %}">

        <title>{% block title %}{% endblock %}</title>
    </head>

    <body class="bg_img">
        {% include "./LABucketListApp_navbar.html" %}

        <div class="header-center">
            {% block header %}{% endblock %}
        </div>

        {% block content %} {% endblock %}
        <!-- for the modal -->
        <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js" integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj" crossorigin="anonymous"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>

    </body>

</html>
```
## Template Inheritance
The great thing about Django is having template inheritance. Once I had a base template, all I needed was to extend the base in all other HTML files. Below is the code snippet for my home page. As you can see, it was very concise, all because everything that I needed was inherited from the base template.
```python
{% extends "LABucketListApp/LABucketListApp_base.html" %}

{% block title %}Los Angeles Bucket List{% endblock %}

{% block header %}<h1 class="home-header">LOS ANGELES</h1>{% endblock %}

{% block content %}{% endblock %}
```
## Bootstrap Elements
Using bootstrap elements, I was able to utilize a navbar for my navigation header, cards for my [index page](https://github.com/Maria-hou/Django-Live-Project/blob/main/index.PNG), a carousel for my [details page](https://github.com/Maria-hou/Django-Live-Project/blob/main/details.png) and [update page](https://github.com/Maria-hou/Django-Live-Project/blob/main/edit.png), and a [modal](https://github.com/Maria-hou/Django-Live-Project/blob/main/delete.png) for a confirmation of deleteting a location.

