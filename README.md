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

