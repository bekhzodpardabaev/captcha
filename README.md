# captcha
## install
```
pip install django-recaptcha
```
```
pip install django-environ
```
```
pip install python-dotenv
```

## .env
```
RECAPTCHA_PUBLIC_KEY=6LdtyR0iAAAAAJ7rUZsGnM52_zYcIeoRCDLQIXDv
RECAPTCHA_PRIVATE_KEY=6LdtyR0iAAAAAFoD69lODqv5Figk00g_jF1A4cJQ
```

## settings.py
```
from dotenv import load_dotenv
load_dotenv()
```
```
INSTALLED_APPS = [
    ...
    'captcha',
    ...
]
```
```
TEMPLATES = [
    {
        ...
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        ...
]
```
```
RECAPTCHA_PUBLIC_KEY = str(os.environ.get("RECAPTCHA_PUBLIC_KEY"))
RECAPTCHA_PRIVATE_KEY = str(os.environ.get("RECAPTCHA_PRIVATE_KEY"))
```

#### path: venv/lib/django/contrib/admin/templates/admin/login.html
add: 56 line -> ```<div class="form-row"> {{ form.captcha.errors }} {{ form.captcha }} </div>```

## templates/login.html
```
{% extends "admin/base_site.html" %}
{% load i18n static %}

{% block extrastyle %}{{ block.super }}
    <link rel="stylesheet" type="text/css" href="{% static "admin/css/login.css" %}">
    {{ form.media }}
{% endblock %}

{% block bodyclass %}{{ block.super }} login{% endblock %}

{% block usertools %}{% endblock %}

{% block nav-global %}{% endblock %}

{% block nav-sidebar %}{% endblock %}

{% block content_title %}{% endblock %}

{% block breadcrumbs %}{% endblock %}

{% block content %}
    {% if form.errors and not form.non_field_errors %}
        <p class="errornote">
            {% if form.errors.items|length == 1 %}{% translate "Please correct the error below." %}{% else %}
                {% translate "Please correct the errors below." %}{% endif %}
        </p>
    {% endif %}

    {% if form.non_field_errors %}
        {% for error in form.non_field_errors %}
            <p class="errornote">
                {{ error }}
            </p>
        {% endfor %}
    {% endif %}

    <div id="content-main">

        {% if user.is_authenticated %}
            <p class="errornote">
                {% blocktranslate trimmed %}
                    You are authenticated as {{ username }}, but are not authorized to
                    access this page. Would you like to login to a different account?
                {% endblocktranslate %}
            </p>
        {% endif %}

        <form action="{{ app_path }}" method="post" id="login-form">{% csrf_token %}
            <div class="form-row">
                {{ form.username.errors }}
                {{ form.username.label_tag }} {{ form.username }}
            </div>
            <div class="form-row">
                {{ form.password.errors }}
                {{ form.password.label_tag }} {{ form.password }}
                <input type="hidden" name="next" value="{{ next }}">
            </div>
            <div class="form-row">
                {{ form.captcha.errors }} {{ form.captcha }}
            </div>
            {% url 'admin_password_reset' as password_reset_url %}
            {% if password_reset_url %}
                <div class="password-reset-link">
                    <a href="{{ password_reset_url }}">{% translate 'Forgotten your password or username?' %}</a>
                </div>
            {% endif %}
            <div class="submit-row">
                <input type="submit" value="{% translate 'Log in' %}">
            </div>
        </form>

    </div>
{% endblock %}
```

## Admin.py
```
from captcha import fields
from django.contrib.auth.forms import AuthenticationForm
import environ

env = environ.Env()
env.read_env(".env")


class LoginForm(AuthenticationForm):
    captcha = fields.ReCaptchaField()


if env.str("RECAPTCHA_PUBLIC_KEY", None) and env.str("RECAPTCHA_PRIVATE_KEY", None):
    admin.site.login_form = LoginForm
admin.site.login_template = "login.html"
```

### panel login
```
from django.urls import reverse_lazy
from django.contrib.auth.views import LoginView
from ###.admin import LoginForm
import environ

env = environ.Env()
env.read_env(".env")


class Login(LoginView):
    if env.str("RECAPTCHA_PUBLIC_KEY", None) and env.str("RECAPTCHA_PRIVATE_KEY", None):
        form_class = LoginForm
    template_name = "##/##/login.html"

    def get_success_url(self):
        return reverse_lazy("###")
```

### jazmin admin login
```
{% extends "registration/base.html" %}

{% load i18n jazzmin %}
{% get_jazzmin_settings request as jazzmin_settings %}
{% get_jazzmin_ui_tweaks as jazzmin_ui %}

{% block content %}
    <p class="login-box-msg">{{ jazzmin_settings.welcome_sign }}</p>
    <form action="{{ app_path }}" method="post">
        {% csrf_token %}
        {% if user.is_authenticated %}
            <p class="errornote">
                <div class="callout callout-danger">
                    <p>
                        {% blocktrans trimmed %}
                            You are authenticated as {{ username }}, but are not authorized to
                            access this page. Would you like to login to a different account?
                        {% endblocktrans %}
                    </p>
                </div>
            </p>
        {% endif %}
        {% if form.errors %}
            {% if form.username.errors %}
                <div class="callout callout-danger">
                    <p>{{ form.username.label }}: {{ form.username.errors|join:', ' }}</p>
                </div>
            {% endif %}
            {% if form.password.errors %}
                <div class="callout callout-danger">
                    <p>{{ form.password.label }}: {{ form.password.errors|join:', ' }}</p>
                </div>
            {% endif %}
            {% if form.captcha.errors %}
                <div class="callout callout-danger">
                    <p>{{ form.captcha.label }}: {{ form.captcha.errors|join:', ' }}</p>
                </div>
            {% endif %}
            {% if form.non_field_errors %}
                <div class="callout callout-danger">
                    {% for error in form.non_field_errors %}
                        <p>{{ error }}</p>
                    {% endfor %}
                </div>
            {% endif %}
        {% endif %}
        <div class="input-group mb-3">
            <input type="text" name="username" class="form-control" placeholder="{{ form.username.label }}" required>
            <div class="input-group-append">
                <div class="input-group-text">
                    <span class="fas fa-user"></span>
                </div>
            </div>
        </div>
        <div class="input-group mb-3">
            <input type="password" name="password" class="form-control" placeholder="{{ form.password.label }}" required>
            <div class="input-group-append">
                <div class="input-group-text">
                    <span class="fas fa-lock"></span>
                </div>
            </div>
        </div>
        <div class="input-group mb-3">
            {{ form.captcha }}
        </div>
        {% url 'admin_password_reset' as password_reset_url %}
        {% if password_reset_url %}
            <div class="mb-3">
                <div class="password-reset-link" style="text-align: center;">
                    <a href="{{ password_reset_url }}">
                        {% trans 'Forgotten your password or username?' %}
                    </a>
                </div>
            </div>
        {% endif %}
        <div class="row">
            <div class="col-12">
                <button type="submit" class="btn {{ jazzmin_ui.button_classes.primary }} btn-block">
                    {% trans "Log in" %}
                </button>
            </div>
        </div>
    </form>
{% endblock %}
```
