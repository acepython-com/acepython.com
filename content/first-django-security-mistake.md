Title: The first security mistake people make in Django
Date: 2026-04-14
Category: posts
Tags: django, security
Slug: first-django-security-mistake
Summary: Oops I committed my secret key!

## You've just started your first Django project...

```python
uv venv && uv pip install django
uv run django-admin startproject mysite djangotutorial
git init . && git commit -m "First commit" && git push
```

Oops. Django created a secret key for you, and you just committed it to your git repo. Your app is now insecure, but how? And what do you do about it?

If you look inside your `settings.py`, which in the case above would be `djangotutorial/mysite/settings.py`, you'll see a `SECRET_KEY` variable right near the top, along with a strong warning to keep the production secret key, well, secret. In modern Django versions, the key is prefixed with `django-insecure`, just to make it super-duper clear this shouldn't be the real key you use.

Django tries so hard to keep you from accidentally leaking your secret key because it's the foundation of authentication inside the framework. There's two critical places secret key is used: Inside the `PasswordResetTokenGenerator` and in `get_cookie_signer`. Let's handle these one at a time.

Django's built-in authorization system includes utilities for handling password resets. When a reset is requested for a user, the `PasswordResetTokenGenerator` generates a salted and hashed token that can be sent to the user. When the user loads the password reset view with the token, a token using the same salt and key is generated, and the user-provided token is compared to this new token. The password reset token is not stored in the database. It's generated on the fly and compared for authenticity. 

```python
# pseudocode of what's happening

possible_match_token = PasswordResetTokenGenerator.make_token(user)
if user_supplied_token == possible_match_token:
    let_user_change_password()
else:
    reject_password_change_attempt()
```

If your secret key is compromised, then an attacker can generate a correct password reset token to reset any user's password.

Sadly that's not the worst part. `get_cookie_signer` uses the secret key to sign session cookies. Django's session cookies are what the browser stores to authenticate users. If an attacker gets access to your secret key, they can forge session cookies and impersonate any user, including admin users.

So, if leaking the secret key is so bad, and unfortunately so easy to casually commit, what do you do when it happens? The best way is to use the same function Django does:

```python
cd djangotutorial && uv run python manage.py shell
...
>>> from django.core.management.utils import get_random_secret_key; get_random_secret_key()
'e_yc)hw!+t7t^^l1k=1tpamz1)*hhopn^!ne-bqegqa7*nwmfe'
>>> 
```

Once you have your new secret key, the best practice would be to have your settings file read that from an environment variable or other secret store, but that's something we'll cover in a future post.
