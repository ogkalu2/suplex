# Suplex

Simple state module to manage user auth and create database queries.

---

## Install

Add Suplex to your project.

```bash
uv add suplex
# or
pip install suplex
```

## Environment Variables

Where module is located, copy the .env.example to .env. Add in your url, keys, and jwt secret. You can also add service_role if you want to option to run client as admin (but be careful you understand how it works!). **If your suplex module is anywhere other than inside your .venv file, then ensure your .gitignore is updated to ignore .env files.**

```bash
cd /path/to/suplex
cp .env.example .env
nano .env
```

## Insert

Import Suplex, and subclass the module at the base layer.

```python
from suplex import Suplex

class BaseState(Suplex):
    # Your class below...
```

## Subclass

For any other classes within your Reflex project, subclass your BaseState to give them access to the auth information and query methods.

```python
class OtherState(BaseState):
    # Your class below...
```

---

## Auth

Suplex comes with built-in vars and functions to manage users, user objects, JWT claims and more. Because front-end handling is different from project to project, you'll need to create functions for how you'd like to update your UI, redirect on success/failure, and handle exceptions.

After logging user in, the access and refresh tokens are stored within your BaseState as

```python
self.access_token
# and
self.refresh_token
```

You won't need to do anything with those tokens directly, as there are a ton of helper vars and functions to extract relevant details, get user objects, and refresh sessions.



### Auth Functions

- sign_up()

- sign_in_with_password()

- sign_in_with_oauth()

- get_user()

- update_user()

- refresh_session()

- get_settings()

- log_out()

Check docstrings for params, returns and exceptions.

```python
from suplex import Suplex


class BaseState(Suplex):
    # Login example.
    def log_in(self, email, password):
        try:
            self.sign_in_with_password(email, password)
        except httpx.HTTPStatusError as e:
            if e.response.status_code == 401:
            yield rx.toast.error("Invalid email or password.")
        except Exception:
            yield rx.toast.error("Login failed.")

    # Update user example.
    def update_user_info(self, email, phone, password, user_metadata):
        try:
            self.update_user(email, phone, password, user_metadata)
        except httpx.HTTPStatusError as e:
            if e.response.status_code == 403:
            # Refresh token here and try again.
        except Exception:
            yield rx.toast.error("Updating user info failed.")


    def log_out(self):
        try:
            self.logout()
        except httpx.HTTPStatusError:
            yield rx.toast.error("Unable to logout remotely, local session cleared.")
        except Exception:
            yield rx.toast.error("Something went wrong during logout.")
```

### Auth Vars

There is a full set of vars that pull values from the signed JWT that gets provided from Supabase in the form of an access_token. These vars pull those claims. If you don't want to use local information and instead only want to rely on a user object pulled directly from Supabase then you will want to use the get_user() function and parse the user object directly.



- user_id

- user_email

- user_phone

- user_audience

- user_role

- claims_issuer

- claims_expire_at

- claims_issued_at

- claims_session_id

- user_metadata

- app_metadata

- user_aal

- user_is_authenticated

- user_is_anonymous

- user_token_expired

```python
# Frontend
def auth_component() -> rx.Component:
    # Show only if user is logged in.
    return rx.cond(
        BaseState.user_is_authenticated,
        rx.shown_if_authenticated(),
        rx.shown_if_not_authenticated()
)

def user_info_panel() -> rx.Component:
    # Show currently logged in user info.
    return rx.flex(
        rx.text(BaseState.user_id),
        rx.text(BaseState.user_email),
        rx.text(BaseState.user_phone),
        class_name="flex-col items-center w-full"
)

# Setup a page to use auth_flow. Redirects user who isn't logged in.
@rx.page("/recipes", on_load=BaseState.auth_flow)
def recipes() -> rx.Component:
    return rx.flex(
        rx.button("Get Recipes")
        on_click=BaseState.get_recipes()
)

class BaseState(Suplex):

    def auth_flow(self) -> rx.Event:
        if not self.user_is_authenticated:
            return rx.redirect("/login")
        
```

---

## Query

Once a user is signed in, calling the query class that is already instantiated inside of your BaseClass is how you build a query using the logged in user's credentials. The style of query is almost identical to the official Supabase python client located at - [Python API Reference | Supabase Docs](https://supabase.com/docs/reference/python/select). The only difference in Suplex syntax is the addition of the .query object at the start of the chain.

```python
from suplex import Suplex


class BaseState(Suplex):
    def get_all_ingredients(self) -> list:
        # Get all unique ingredients from a collection of recipes.
        try:
            ingredients = []
            results = self.query.table("recipes").select("ingredients").execute()
            for result in results:
                ingredients.extend(result["ingredients"])
            return list(set(ingredients))
        except Exception:
            rx.toast.error("Unable to retrieve ingredients.")


    def get_recipes_with_parmesan_cheese(self) -> list:
        # Get recipes with parmesan cheese as an ingredient.
        try:
            results = self.query.table("recipes").select("*").in_("ingredients", ["parmesan"]).execute()
            return results
        except Exception:
            rx.toast.error("Unable to retrieve recipes.")
```

### Query Methods

[Python API Reference | Supabase Docs](https://supabase.com/docs/reference/python/select)

- select()

- insert()

- upsert()

- update()

- delete()

### Query Filters (Incomplete)

[Python API Reference | Supabase Docs](https://supabase.com/docs/reference/python/using-filters)

- eq()

- neq()

- gt()

- lt()

- gte()

- lte()

- like()

- ilike()

- is_()

- in_()

- contains()

- contained_by()

### Query Modifiers (Incomplete)

[Python API Reference | Supabase Docs](https://supabase.com/docs/reference/python/using-modifiers)

- order()

---

## Notes

Generally this module is attempting to do the dirty work of setting up a request and turning the response into a python object. I'm leaving error handling, logging, and flows up to the devs so that you can more easily integrate this into your own flows.

If there is a feature you'd like added that keeps the spirit of flexibility but adds functionality then please let me know and I'll be happy to extend this module.

Documentation and structure of this project is **very early** so expect changes as I integrate this project and test all the features thoroughly.

## Thanks!
