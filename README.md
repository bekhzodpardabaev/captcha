# captcha
## install
```
pip install django-recaptcha
```
```
pip install django-environ
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
#### path:venv/lib/django/contirb/admin/templates/admin/login.html
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

## settings.py
```
TEMPLATES = [
    {
        ...
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        ...
]
```
