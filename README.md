# Live Project
## Introduction
For two weeks of my time at the Tech Academy, I worked with a team of other students on a Django Python project to develop apps for an interactive website for managing one's collections of things related to various hobbies. Working in the Django framework and in a team setting was a great opportunity for learning version control, managing deadlines, and adding requested features. I saw how a good developer works with what they have and know to make a quality product. Within this two week sprint, I was given user stories of increasing difficulty to complete inorder to increase our problem solving mindest. I worked on several [back end stories](#back-end-stories) and front end stories of an application about destinations to visit in Los Angeles. Being a student there, I have always wanted to keep track of the places I have been. So, this was the perfect opportunity to incorporate something I am passionate about, into a project where I gained real world experience and improved my skills. 

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
