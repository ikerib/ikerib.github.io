---
layout: single
title: Symfony 4 Change locale
excerpt: "Symfony 4: Change locale"
date: 2019-07-04
classes: wide
header:
    teaser: /assets/images/changeLocale.png
    teaser_home_page: true
categories:
- Symfony
tags:
- Symfony
---

First of all, we need to config our app, to do so edit your `config/services.yaml` and add this:

```yaml
  parameters:
    locale: 'eu'
    app_locales: 'es|eu'
  services:
    _defaults:
        autowire: true
        autoconfigure: true 
        bind:
            $locales: '%app_locales%'
            $defaultLocale: '%locale%'
```

Now add your default locale in `config/packages/framework.yaml`

```yaml
framework:
    ide: 'phpstorm://open?url=file://%%f&line=%%l'
    secret: '%env(APP_SECRET)%'
    default_locale: eu
```

Edit your routes and add the ‘locale’ parameter. The fastest way is to edit `congig/routes/annotations.yaml`

```php
controllers:
    resource: ../../src/Controller/
    type: annotation
    prefix: /{_locale}
    requirements:
        _locale: '%app_locales%'
    defaults:
        _locale: '%locale%'
```

Be aware that probably you need to modify your access control path on `config/packages/security.yaml`

```yaml
access_control:
    - { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
    - { path: '^/(%app_locales%)/admin', roles: ROLE_ADMIN }
```

Everything is ready. It’s time to setup our twig template to change the locale. Create a new file into `templates` folder called `_locale_switcher.html.twig` with this content: (In my case, I use only 2 locales so I used a button to make development faster, maybe it’s a better idea to do that with a select)
{% raw %}
```twig
    {% set route = app.request.attributes.get('_route') %}
    {% set route_params = app.request.attributes.get('_route_params') %}
    {% set params = route_params|merge(app.request.query.all) %}

    {% set applocales=locales|split('|') %}
    {% for locale in applocales if locale != app.request.locale %}
        <a class="" href="{{ path(route, params|merge({ _locale: locale })) }}">
            {{ "Aldatu hizkuntza" | trans }}
        </a>
    {% endfor %}
```
{% endraw %}

Add this code fragment in your template:

{% raw %}
```twig
<ul class="control-sidebar-menu">
    <li>
        {% include '_locale_switcher.html.twig' %}
    </li>
</ul>
```
{% endraw %}

## How to change the locale within the controller

We use LDAP like a User database, so all our Apps authentication system is against this. In LDAP we have a field called ‘preferredLanguage’ where we save the locale user prefers, so the first thing after a correct login in our app is to setup a locale using this ‘preferredLanguage’ from LDAP.

```php
... GET LDAP info such a group membership, set a ROLE depending in witch group is member of... store this data in session and then/**
 * User objectu berria sortu, rol berriekin
 */
$token = new UsernamePasswordToken(
    $this->getUser(),
    null,
    'main',
    $roles
);
$this->get( 'security.token_storage' )->setToken( $token );

/** @var Session $session */
$session = $request->getSession();

/* Localea zehaztu */
$ldapLanguage = $entry->getAttribute('preferredLanguage');
if ($ldapLanguage) {
    $this->get('session')->set('_locale', $ldapLanguage[0]);
    $request->setLocale($ldapLanguage[0]);
}


return $this->redirectToRoute('ROUTE');
```