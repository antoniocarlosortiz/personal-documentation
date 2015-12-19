#MAJOR BUGS (That to took me a while to solve)

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

* When adding new folders or files, make sure to reinstall the app in the virtual env or else the flask environment will have a difficult time looking for folders and files

####CRESCENT MOON INNOVATIONS INTERNATIONAL DJANGO E-COMMERCE SITE

* Django memory leak with gunicorn: CPU usage is off the charts with just a simple app. add ```-max requests <number>``` where the number is the number of requests before the gunicorn worker restarts.
