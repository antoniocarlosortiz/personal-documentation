#Bugs that to took me a while to solve and other notes.

####FLASK API TEMPLATE
* ```no module named shared libs``` while running alembic: virtualenv for web app must have not been started.

* ```rfc code url error``` while running alembic: Its either the url at local_config.ini is wrong or theres not local_config.ini to begin with.

* While running endpoints ```sqlalchemy.orm.exc.DetachedInstanceError: Instance <LinkedInAccount at 0x7fa6e6f4ded0> is not bound to a Session; attribute refresh operation cannot proceed```: include expire_on_commit=False on the backend functions.

* ```no module named <any>```: Normally this is caused when the virtualenv for the project was not activated. The command `python setup.py develop` initialized PYTHONPATH within the virtualenv and not on your local.

* On using HTTPBasicAuth for flask: Make sure the authentication wrapper is below the route wrapper else authentication will not work.

* When tox can't find dependencies even though you've installed them:
  * http://stackoverflow.com/questions/16061514/pylint-pylint-unable-to-import-flask-ext-wtf
  * or delete ```rm -rf .tox/```. Tox will rebuild this folder if its not there.


* If dependencies can't be found on running scripts: try running ```python setup.py develop```

* SQLAlchemy/psycopg2 ```Cant adapt type error```: You must have inserted an object or something unexpected to an SQLAlchemy query. Check the types of the data you pass to the query. Queries normally can't accept objects.

* SQLAlchemy ```Timeout Error```: Some part of the code is creating too many sessions at the same time. Find and fix it.

* When adding new folders or files, make sure to reinstall the app in the virtual env or else the flask environment will have a difficult time looking for folders and files.


####DJANGO E-COMMERCE SITE

* Django memory leak with gunicorn: CPU usage is off the charts with just a simple app. add ```-max requests <number>``` where the number is the number of requests before the gunicorn worker restarts.

####SPECTRUM ONE WEBSITE on Django

* On calling the list of a certain foreign model to a subject model in Django, Its better to just use `related_name` on the ForeignKey and then just use in the template: `{{ <main_model_name>.<related_name_used>.all }}`

* To know what SITE_ID does: [http://stackoverflow.com/questions/25468676/django-sites-model-what-is-and-why-is-site-id-1](http://stackoverflow.com/questions/25468676/django-sites-model-what-is-and-why-is-site-id-1).

* The `SECRET_KEY` on settings.py is used as salt mainly for csrf_token and cookie generation. Related link:  [http://stackoverflow.com/questions/15170637/effects-of-changing-djangos-secret-key](http://stackoverflow.com/questions/15170637/effects-of-changing-djangos-secret-key).

* Once Debug is false, Django will no longer handle static and media files. The production web server should take care of that (NGINX or Gunicorn)

####Texting System using Django
* On supposed memory leaks caused by Django: [http://blog.gingerlime.com/2011/django-memory-leaks-part-i/](http://blog.gingerlime.com/2011/django-memory-leaks-part-i/)

####Using Django with SQLite3 inside Vagrant using NFS as the file sharing system
* NFS semantics are different than regular filesystems and does not play well with sqlite3. If you still try to use it though, `runserver` will take a while but will eventually load to a `database is locked` error.

 Source:
  * [http://stackoverflow.com/questions/9907429/locking-sqlite-file-on-nfs-filesystem-possible](http://stackoverflow.com/questions/9907429/locking-sqlite-file-on-nfs-filesystem-possible)
  * [https://www.sqlite.org/faq.html](https://www.sqlite.org/faq.html)
  * 
  
####Generic Django Site
* Problem loading fixtures; Error saying Django could not find `contrib.auth.models.User`: Load the user fixture first then load all the other fixtures next. Apparently Django loads fixtures async. 
