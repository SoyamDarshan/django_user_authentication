# DJANGO USER AUTHENTICATION

##Creating initial directories:

The following command will create all the required directories that we will be using in our app.
Create them in the project directory not the app directory.


```
mkdir templates static media media/profile_pics templates/django_user_auth
```

## Modifying the settings.py

We add the directories we created earlier.

```
# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

TEMPLATE_DIR = os.path.join(BASE_DIR,'templates')
STATIC_DIR = os.path.join(BASE_DIR,'static')
MEDIA_DIR = os.path.join(BASE_DIR,'media')
```

Add the <app_name> in the **INSTALLED_APPS**.

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'django_user_auth'
]
```

Add *TEMPLATE_DIR* under **DIRS** in **TEMPLATES**.

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [TEMPLATE_DIR,],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
] 
```


Add these lines for static, media and templates.

```
STATIC_URL = '/static/'
STATICFILES_DIR = [STATIC_DIR,]

MEDIA_ROOT = [MEDIA_DIR,]
MEDIA_URL = '/media/'

LOGIN_URL = '/django_user_auth/user_login'
```

## Creating Models for our app.

Import this ```from django.contrib.auth.models import User``` in your models.py for accessing the User model from django authentication system.

```
from django.db import models
from django.contrib.auth.models import User

class UserProfileInfo(models.Model):

    user = models.OneToOneField(User, on_delete=models.CASCADE)
    portfolio_site = models.URLField(blank=True)
    profile_pic = models.ImageField(upload_to='profile_pics', blank=True)

    def __str__(self):
        return self.user.username

```

Now migrate your models.

Run these commands in your terminal:

```
python manage.py makemigration
python manage.py migrate
```



## Creating Forms for our app.

Import this ```from django import Forms``` in your forms.py for accessing the Forms module in django.

```
from django import forms
from .models import UserProfileInfo
from django.contrib.auth.models import User

class UserForm(form.ModelForm):
    password = forms.CharField(widget=forms.PasswordInput())

    class Meta():
        model = User
        fields = ('username', 'password', 'email')


class UserProfileInfoForm(forms.ModelForm):

    class Meta():
        model = UserProfileInfo
        fields = ('portfolio_site', 'profile_pic')
```

## Registering our model.

Models are registered in admin.py of the app so that we can have admin control over them for performing certain operations as an admin.

```
from django.contrib import admin
from .models import UserProfileInfo


admin.site.register(UserProfileInfo)
```


## Creating our view class.

To implement our views we have to import certain things which are necessary.
HttpResponseRedirect and HttpResponse from the django.http library

HttpResponseRedirect: This should be used only to redirect to another page. (HTTP code 302 (Found/Moved temporarily))
HttpResponse: only use this for really small responses. (HTTP code 200 (OK))


```
from django.shortcuts import render
from django_user_auth.forms import UserForm, UserProfileInfoForm
from django.contrib.auth import authenticate, login, logout
from django.http import HttpResponseRedirect, HttpResponse
from django.urls import reverse
from django.contrib.auth.decorators import login_required
```


The index will be used to redirect to our index page.

```
def index(request):
    return render(request, 'django_user_auth/index.html')
```


Special function will be invoked after login successful.
```
@login_required
def special(request):
    return HttpResponse("Logged in!")
```


user_logout will be invoked for logging out.
```
@login_required
def user_logout(request):
    logout(request)
    return HttpResponseRedirect(reverse('index'))
```


Register function will be used when we want to register a new user.
It will access the user forms and user profile forms we declared in our forms.py file.
After receiving all information, it will save the data. 
```
def register(request):
    registered = False
    if request.method == 'POST':
        user_form = UserForm(data=request.POST)
        profile_form = UserProfileInfoForm(data=request.POST)
        if user_form.is_valid() and profile_form.is_valid():
            user = user_form.save()
            user.set_password(user.password)
            user.save()
            profile = profile_form.save(commit=False)
            profile.user = user
            if 'profile_pic' in request.FILES:
                print('found it')
                profile.profile_pic = request.FILES['profile_pic']
            profile.save()
            registered = True
        else:
            print(user_form.errors, profile_form.errors)
    else:
        user_form = UserForm()
        profile_form = UserProfileInfoForm()
    return render(request, 'django_user_auth/registration.html', {'user_form': user_form,
                                                                  'profile_form': profile_form,
                                                                  'registered': registered
                                                                  })
```

This will be invoked when the user would log-in.
```
def user_login(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        user = authenticate(username=username, password=password)

        if user:
            if user.is_active:
                login(request, user)
                return HttpResponseRedirect(reverse('index'))
            else:
                return HttpRespose("Your account has been suspended")
        else:
            print("Someone tried to login and failed")
            print("They used username: {} and password: {}".format(username, password))
            return HttpRespose("Invalid login details provided")
    else:
        return render(request, 'django_user_auth/login.html', {})
```






