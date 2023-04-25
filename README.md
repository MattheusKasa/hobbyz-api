# HobbyzAPI


Hobbyz is a community-driven website that provides a platform for hobby enthusiasts to share their interests with like-minded people. This portion of the project serves as the backend API database, designed to support the ReactJS frontend. It is built using the Django Rest Framework.

## Table of Contents
+ [User Stories](#user-stories "User Stories")



### User Stories:
---

Every User Story has been recorded in a separate file, the link to which is available [HERE](static/userstories.md).

I have provided links to the [GitHub Issues](https://github.com/MattheusKasa/hobbyz/issues) related to this project, as well as the [KANBAN board](https://github.com/users/MattheusKasa/projects/6) for your reference.

## Testing:
---
### Manual Testing:
- Manually verified that each created url path opens without errors.
- Verified that CRUD functionality is available in each app.
- Ensured search feature for Posts returns results.

### Validator Testing: 
All files passed through [PEP8](http://pep8online.com/) without error.

## Technologies Used:
---
### Languages:
- Python
### Frameworks, Libraries, and Tools:
- Django
- Django RestFramework
- Cloudinary
- Heroku
- Pillow
- Django Rest Auth
- PostgreSQL
- Cors Headers


## Deployment:
---
### Project Creation:
1. Create a GitHub repository.
2. Set up the project app on [Heroku](heroku.com).
3. Add the Postgres package to the Heroku app via the Resources tab.
4. After launching the GitHub repository on GitPod, install the necessary packages using the `pip install` command:
```
'django<4'
dj3-cloudinary-storage
Pillow
djangorestframework
django-filter
dj-rest-auth
'dj-rest-auth[with_social]'
djangorestframework-simplejwt
dj_database_url psycopg2
gunicorn
django-cors-headers
```
5. Initialize the Django project with this command:
```
django-admin startproject project_name .
```
6. Go back to [Heroku](heroku.com), and under the Settings tab, add the following configvars:
 - Key: SECRET_KEY | Value: hidden
 - Key: CLOUDINARY_URL | Value: cloudinary://hidden
 - Key: DISABLE_COLLECTSTATIC | Value: 1
 - Key: ALLOWED_HOST | Value: api-app-name.herokuapp.com
7. Once the ReactApp has been created, add two more configvars:
 - Key: CLIENT_ORIGIN | Value: https://react-app-name.herokuapp.com
 - Key: CLIENT_ORIGIN_DEV | Value: https://gitpod-browser-link.ws-eu54.gitpod.io
  - Ensure the trailing slash `\` at the end of both links is removed.
  - Gitpod may occasionally update the browser preview link. If this happens, update the CLIENT_ORIGIN_DEV value accordingly.

8. Create the env.py file and add the following variables. Obtain the value for DATABASE_URL from the Heroku configvars in the previous step:
```
import os

os.environ['CLOUDINARY_URL'] = 'cloudinary://hidden'
os.environ['DEV'] = '1'
os.environ['SECRET_KEY'] = 'hidden'
os.environ['DATABASE_URL'] = 'postgres://hidden'
```
### In settings.py: 
<!-- For reference, refer to: [DRF-API walkthrough settings.py](https://github.com/Code-Institute-Solutions/drf-api/blob/2c0931a2b569704f96c646555b0bee2a4d883f01/drf_api/settings.py) -->
9. Add the following to INSTALLED_APPS to accommodate the newly installed packages:
```
'cloudinary_storage',
'django.contrib.staticfiles',
'cloudinary',
'rest_framework',
'django_filters',
'rest_framework.authtoken',
'dj_rest_auth',
'django.contrib.sites',
'allauth',
'allauth.account',
'allauth.socialaccount',
'dj_rest_auth.registration',
'corsheaders',
```
10. Import the database, the regular expression module, and env.py:
```
import dj_database_url
import re
import os
if os.path.exists('env.py'):
    import env
```

11. Add the following variable for Cloudinary below the import statements:
```
CLOUDINARY_STORAGE = {
    'CLOUDINARY_URL': os.environ.get('CLOUDINARY_URL')
}

MEDIA_URL = '/media/'
DEFAULT_FILE_STORAGE = 'cloudinary_storage.storage.MediaCloudinaryStorage'
```
- Set site ID below INSTALLED_APPS:
```
SITE_ID = 1
```
12. Add the REST_FRAMEWORK below BASE_DIR, including page pagination to enhance app loading times, pagination count, and date/time format:
```
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [(
        'rest_framework.authentication.SessionAuthentication'
        if 'DEV' in os.environ
        else 'dj_rest_auth.jwt_auth.JWTCookieAuthentication'
    )],
    'DEFAULT_PAGINATION_CLASS':
        'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
    'DATETIME_FORMAT': '%d %b %Y',
}
```
13. Set the default renderer to JSON:
```
if 'DEV' not in os.environ:
    REST_FRAMEWORK['DEFAULT_RENDERER_CLASSES'] = [
        'rest_framework.renderers.JSONRenderer',
]
```
15. Then add:
```
REST_AUTH_SERIALIZERS = {
    'USER_DETAILS_SERIALIZER': 'project_name.serializers.CurrentUserSerializer'
}
```
16. Update the DEBUG variable to:
```
DEBUG = 'DEV' in os.environ
```
17. Update the DATABASES variable to:
```
DATABASES = {
    'default': ({
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    } if 'DEV' in os.environ else dj_database_url.parse(
        os.environ.get('DATABASE_URL')
    )
    )
}
```
18. Add the Heroku app to the ALLOWED_HOSTS variable:
```
os.environ.get('ALLOWED_HOST'),
'localhost',
```
19. Below ALLOWED_HOST, add the CORS_ALLOWED variable:
```
if 'CLIENT_ORIGIN' in os.environ:
    CORS_ALLOWED_ORIGINS = [
        os.environ.get('CLIENT_ORIGIN')
    ]

if 'CLIENT_ORIGIN_DEV' in os.environ:
    extracted_url = re.match(r'^.+-', os.environ.get('CLIENT_ORIGIN_DEV', ''), re.IGNORECASE).group(0)
    CORS_ALLOWED_ORIGIN_REGEXES = [
        rf"{extracted_url}(eu|us)\d+\w.gitpod.io$",
    ]
```
20. Add the following code to the top of MIDDLEWARE:
```
'corsheaders.middleware.CorsMiddleware',
```
- During a deployment issue, i added the following lines of code below CORS_ALLOW_CREDENTIALS:
```
CORS_ALLOW_ALL_ORIGINS = True
CORS_ALLOW_HEADERS = list(default_headers)
CORS_ALLOW_METHODS = list(default_methods)
CSRF_TRUSTED_ORIGINS = [os.environ.get(
    'CLIENT_ORIGIN_DEV', 'CLIENT_ORIGIN',
)]
```
- Additionally, i added the following import statement at the top of the settings.py file:
```
from corsheaders.defaults import default_headers, default_methods
```

### Final Requirements:
21. Create a Procfile and add these two lines:
```
release: python manage.py makemigrations && python manage.py migrate
web: gunicorn project_name.wsgi
```
22. Migrate the database:
```
python3 manage.py makemigrations
python3 manage.py migrate
```
23. Freeze requirements:
```
pip3 freeze --local > requirements.txt
```
24. Add, commit, and push changes to GitHub.
25. Go back to Heroku, and under the 'Deploy' tab, connect the GitHub repository.
26. Deploy the branch.

### Deploying to ElephantSQL:
* The project has been deployed using [ElephantSQL](https://www.elephantsql.com/) by following these [instructions](https://code-institute-students.github.io/deployment-docs/41-pp5-adv-fe/pp5-adv-fe-drf-01-create-a-database).

## Credits:
---
### Content:
- The foundation for this API database was laid using the step-by-step guide provided by the Code Institute's DRF-API walkthrough project.
- All classes and functions have been properly credited.
---
[🔼 Back to top](#hobbyzapi)