#Django Best Practices
<i>because I keep forgetting to use these.</i>

###settings.py
* always use environment variables; if it gets tedious always writing those, you can place it in the activate script of the virtualenv youre using.
* Handle missing secret key Exception using this:
```
import os
from django.core.exceptions import ImproperlyConfigured

def get_env_variable(var_name):
    try:
        return os.environ[var_name]
    except KeyError:
        error_msg = "Set the {} environment variable.".format(var_name)
        raise ImproperlyConfigured(error_msg)
```
* On defining BASE_DIR like setting, use the package `Unipath`
```
from unipath import Path

BASE_DIR = Path(__file__).ancestor(3)
MEDIA_ROOT = BASE_DIR.child("media")
STATIC_ROOT = BASE_DIR.child("static")
STATICFILES_DIRS = (
    BASE_DIR.child("assets"),
)
```
###models.py
* Create Generic TimeStampeModel to be inherited by almost all modules
```
#core/models.py
from django.db import models

class TimeStampedModel(models.Model):
    """
    An abstract base class model that provides self-updating ``created`` and ``modified`` fields
    """
    created = models.DateTimeField(auto_now_add=True)
    modified = models.DateTimeFIeld(auto_now=True)
    
    class Meta:
        abstract = True
```
