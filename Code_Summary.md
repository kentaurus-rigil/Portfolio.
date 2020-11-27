# Code Summary for Python Live Project 

## views.py

    from django.shortcuts import render, get_object_or_404, redirect
    from django.http import HttpResponseRedirect
    from .forms import bikeForm, countryForm, detailsForm, ApiForm
    from django.views.generic import View
    from .models import bikeTrail, Country
    import requests
    import json



###  Trails Home Page

    def Trail_Home(request):
      return render(request, 'TrailsApp/Trails_home.html')



###  Bike Trails entry form

    def bike_create_view(request):
      form = bikeForm() 
      if request.method == "POST":
        form = bikeForm(request.POST)
        if form.is_valid(): 
            form.save()
            return redirect('Trail_Home') 
    context = {
        'form': form   
    }
    return render(request, "TrailsApp/bike_create_view.html", context)
    

###  View all trails in the database

    def trails_index(request):
      all_trails = bikeTrail.bikeTrails.all()
      all_country = Country.countries.all()
      context = {
        'all_trails': all_trails,
        'all_country': all_country
      }
      return render(request, "TrailsApp/trails_index.html", context)
    
    
### Display data from the database

    def trails_details(request, pk):
      pk = int(pk)  
      trail = get_object_or_404(bikeTrail, pk=pk)  
      context = {'trail': trail}  
      return render(request, "TrailsApp/trails_details.html", context)


### Create a function to allow user to edit data

    def edit_details(request, pk):
      pk = int(pk)  
      trail = get_object_or_404(bikeTrail, pk=pk) 
      form = bikeForm(vars(trail)) 
      if request.method == "POST":  
        form = bikeForm(request.POST, instance=trail)  
        if form.is_valid(): 
            form.save()  
            return redirect('findTrails')  
      context = {'form': form}  
      return render(request, "TrailsApp/trails_edit.html", context)
    

### Allow user to request data from the API and display it

    def trails_api(request, id0):
      #url = "https://www.hikingproject.com/data/get-conditions?ids=7001635"
      if request.method == "POST":
          id0 = request.POST["trails_id"]#
      url = "https://www.hikingproject.com/data/get-conditions?ids="+str(id0)
      result = requests.get(url)
      result_dict = json.loads(result.text)
      inner_dict = result_dict["0"]#inner_dict is the first dictionary
      form = ApiForm()
      return render(request, "TrailsApp/trails_api.html", {"trail": inner_dict, "form": form})
      
      
  ## Models.py
      from django.db import models
      from django.forms import ModelForm

      DIFFICULTY_CHOICES = (
          ('easy', 'easy'),
          ('moderate', 'moderate'),
          ('difficult', 'difficult')
      )

      CLIMATE_CHOICES = (
          ('dry', 'dry'),
          ('highland', 'highland'),
          ('polar', 'polar'),
          ('temperate', 'temperate'),
          ('tropical', 'tropical')

      )


      class bikeTrail(models.Model):
          name = models.CharField(max_length=60, default="", blank=False, null=False)
          distance = models.DecimalField(default=0.00, max_digits=10000, decimal_places=2)
          difficulty = models.CharField(max_length=60, choices=DIFFICULTY_CHOICES)
          description = models.TextField(max_length=300, default="", blank=True)
          country = models.ForeignKey('Country', on_delete=models.CASCADE, default="")


    bikeTrails = models.Manager()
    
 ## forms.py
    
    class countryForm(forms.ModelForm):

    class Meta:
        model = Country
        fields = '__all__'

    class detailsForm(forms.ModelForm):
    class Meta:
        model = bikeTrail
        fields = '__all__'

    class ApiForm(forms.Form):
      trails_id = forms.CharField(label='Trail ID', max_length=None)
      
  ## API HTML
      
      {% extends 'TrailsApp/Trails_base.html' %}
      {% load staticfiles %}

      {% block apicontent %}
         <form action="" method='POST'>{% csrf_token %}
          {{ form.as_p }}


         <input type='submit' value='Submit'/>
         </form>
           <br>

      {% for key, value in trail.items %}
        <div class="trailApi"> {{ key }} {{ value }}</div>
      {% endfor %}

      {% endblock %}
      
 ## CSS
 
        {% load static %}

        .topnav a {
          float: left;
          color: #f2f2f2;
          text-align: center;
          padding: 14px 16px;
          text-decoration: none;
          font-size: 17px;
        }

        .dbtable {
            color: white;
            width: 95%;
            height: 120px;
            align: center;
            margin: 20px auto;
            border: 1px solid black;
            border-collapse: collapse;
            background-color: #4CAF50;
            padding: 15px;
        }

        table {
            color: white;
            border: 1px solid black;
            background-color: grey;
            padding-bottom: 5px;
            padding-left: 5px;
        }

        /*========================================
                        Background Image
        =========================================*/

        html {
            background-image: url('/static/TrailsApp/images/twoHikers.jpg');
            background-size: cover;
            background-attachment: fixed;
            background-position: center;
            position: relative;
            -webkit-background-size: cover;
            -moz-background-size: cover;
            -o-background-size: cover;
            height: 100%;
        }
        
        /*========================================
                        Get Conditions
        =========================================*/
        .trailApi {
            color: White;
            background-color: grey;
            padding-bottom: 5px;
            padding-left: 5px;
            border: 1px solid black;
        }

