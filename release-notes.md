Version History and Release Notes
=================================

Releases of [SaaS Pegasus: The Django SaaS Boilerplate](https://www.saaspegasus.com/) are documented here.

## Version 0.14.4

A very minor release:

- Don't include `djstripe` and associated settings in `INSTALLED_APPS` if not using subscriptions

*June 30, 2021*

## Version 0.14.3

A very minor release:

- Update Django to 3.2.4
- Remove `DEFAULT_AUTO_FIELD` declaration from settings. 
  This was causing issues with allauth migrations being generated, as a result of [this issue](https://github.com/pennersr/django-allauth/pull/2829).
  Until more libraries have worked around these migration issues, Pegasus will just ship with the default setting.

*June 17, 2021*

## Version 0.14.2

This release upgrades `dj-stripe` to version `2.4.4` which should fix cross-environment migration issues.

**Upgrade notes:**

Projects using a "clean" 0.14 or 0.14.1 release may have been affected by [this issue in dj-stripe](https://github.com/dj-stripe/dj-stripe/issues/1344)
where running `makemigrations` automatically generated a migration file inside the `dj-stripe` library code.
Local app migrations would then add a dependency to this migration, which would not be available on other environments.

This often manifested as an error along the lines of the following when setting up a second environment:

```
django.db.migrations.exceptions.NodeNotFoundError: 
  Migration .0003_auto_20210524_1532 dependencies reference nonexistent parent node ('djstripe', '0008_auto_20210521_2314')`
```

To fix this there are a few steps:

1. You should explicitly uninstall and reinstall `dj-stripe` instead of just installing requirements to ensure the 
   generated migration file is removed. `pip uninstall dj-stripe && pip install dj-stripe==2.4.4`
2. You should rebuild your own app's migrations to no longer depend on the generated `dj-stripe` migration,
   by deleting the bad files and re-running `makemigrations` on a fresh environment.
3. You will need to "fake" migrations for any existing DBs, by manually deleting relevant rows from the `django_migrations`
   table and then running `./manage migrate <appname> --fake` for each affected app.
   
Please reach out for support on Slack if you run into any issues with this, and I'm happy to help!

*May 26, 2021*

## Version 0.14.1

This is a minor release including the latest Django security updates, the official Bootstrap 5.0 version,
and a few small features and fixes.

- Fix incorrectly generated `urls.py` if you built with teams and without subscriptions
- Add password confirmation field to sign up pages if configured by allauth
- Add `project_settings` example for passing project-level settings to templates via context processor
- Update `bootstrap` to `5.0.1`.
- Update `django` from `3.2` to `3.2.3`
- Remove unused `kombu` dependencies from requirements.

**Changes affecting [experimental features](/experimental) only:**

- Slightly improved styling of Tailwind sign up page

*May 19. 2021*

## Version 0.14

Version 0.14 is a major release with a focus on Teams and package upgrades.
It also upgrades Pegasus to Django's 3.2 LTS release.
Details are below:

**Teams Upgrade**

- Make team name optional on signup, and auto-generate a team if none specified.
- Update team-based URL-routing to be more consistent (put all team urls behind `/a/team-slug/`)
- Migrate teams React code to functional components / hooks
- Add URL-routing and back button support to teams page
- Restrict ability to edit/delete teams to team admins only
- Restrict ability to manage team invitations via API to team admins only
- Add manage/view team page to main team navigation
- Other minor teams JS cleanup (replace `var` with `const`, etc.)
- Delete `team_nav.html` when not using teams
- Improve loading screen for teams page and extract to shared component

**Package Upgrades**

- Upgrade all python packages to latest versions (including Django to 3.2 LTS)
- Updated settings file to the version that ships with Django 3.2
- Update formatting of requirements files to the latest version used by `pip-tools`
- Upgrade all JS packages to latest versions 
- Switch from deprecated node-sass to dart-sass

**Other updates**

- Set base URL in React object demo from Django
- Fix doc strings of npm-related `make` commands
- Extract logic for getting CoreAPI JavaScript client to a shared function
- Add support for celery workers to Digital Ocean deployments (and [updated docs](/deployment))
- Add `make dbshell` docker command to get a database shell
- Remove some tailwind files that were accidentally included in non-tailwind builds

**Changes affecting [experimental features](/experimental) only:**

- Improve Tailwind styling of examples, teams and other built-in pages
- Add `@tailwind/forms` plug in for improved form styling
- Split Pegasus Tailwind CSS classes into a new file to mirror Bootstrap/Bulma implementations
- Fix collectstatic errors when building with tailwind and whitenoise (affected Heroku and Digital Ocean deployments)
- Remove broken landing page and pricing page examples, and point people to Tailwind UI instead 

**Upgrade notes:**
- Django added the new [Default primary key field type](https://docs.djangoproject.com/en/3.2/ref/settings/#default-auto-field)
  setting. If you wish to use the default in 3.2 you will have to add DB migrations for it. Otherwise you
  can change the value of `DEFAULT_AUTO_FIELD` to `'django.db.models.AutoField'` in your `settings.py` file.
- Bootstrap users may need to run `npm update bootstrap` before building static assets to get all styling 
  of the examples working again.

*April 28, 2021*

## Version 0.13.2

This is a minor release primarily focusing on an improved Docker experience and
updates to the experimental TailwindCSS support:

- Update development `Dockerfile` from `alpine` to `buster` and install front-end dependencies.
- Add `Makefile` with self-documenting targets for various common operations ([see docs](/docker/))
- Update generated `README` with better Docker instructions
- Use double quotes for description and name settings to reduce issues with apostrophes

Changes affecting [experimental features](/experimental) only:

- Properly support for using PurgeCSS with Tailwind
- Include (purged) Tailwind bundle files with Pegasus

*March 10, 2021*

## Version 0.13.1

This is a minor maintenance release with a few small changes and fixes.

- Fix Docker image when deploying to Heroko containers
- Fix SSL mixed content issues on Heroku Docker builds
- Remove teams JavaScript files if not using teams.
- Add `body_wrapper` block for overriding the whole base template
- Label closing `endblock` tags in base template
- Add `.DS_Store` files to `.gitignore`
- Minor compatibility fixes to `teams` CSS.

*February 24, 2021*

## Version 0.13.0

This release adds support for the [Bootstrap CSS framework](https://getbootstrap.com/) and includes several changes
to how the CSS files are structured in Pegasus.
See the new [CSS documentation](/css/) for an overview of the new structure.

Major changes:

- Full support for Bootstrap CSS!
- Restructure CSS files, creating framework-level subfolders, splitting `site-base` and `site-<framework>` files out, 
  and making Sass imports implicit in the Bulma files.
- Examples CSS overhaul, converting styles to `pg-` classes and adding the `pegasus/<framework>.sass` files to
  style examples and JavaScript.

Minor changes:

- Improved UI of object demo tables on small screens.
- Remove unused "tagline" CSS styles and replace with the CSS frameworks' vertical centering utilities.
- Update progress bar styles to use CSS variables instead of Sass variables.
- Remove unused redundant "section" classes from a number of places.
- Split profile page into multiple HTML templates.
- Remove duplicate cookie-handling code JavaScript on profile page and use the same JS code used in other places.
- Remove unused `plan-price` and `plan-tagline` classes from subscription page.
- Fixed bad reference to `subsriptions.sass` when not building with subscriptions.
- Remove unused `has-vcentered-cells` class.
- Start splitting out teams JavaScript into multiple files.
- Remove redundant `custom-file-upload` CSS class.
- Add /404 and /500 endpoints for viewing custom 404 and 500 pages.
- Remove unnecessary raw string formatting from severeal `urls.py` files.
- Upgrade Django to `3.1.6`


*February 8, 2021*

## Version 0.12.1

This release continues the overhaul of the Pegasus CSS and templates to include experimental support for 
the Bootstrap CSS framework. Details on using Bootstrap [can be found here](/experimental).

Changes affecting everyone:

- Fix "view subscription" page start date display, and accommodate when start date is not set in Stripe.
- Default `ACCOUNT_LOGIN_ON_EMAIL_CONFIRMATION = True` to avoid double-login after email confirmation.
- Move navbar avatar styling to standalone `navbar.sass` file to be shared across CSS frameworks.
- Change navbar avatar class and CSS from `.avatar` to `.navbar-avatar`.
- Split subscription details on the "view subscription" page into its own component templates.
- Split hero section on the "upgrade subscription" page into its own component templates.
- In `subscriptions.sass`, use css variables instead of sass variables for colors.
- On subscription page, change "upgrade-features" selector from an ID to a class in HTML and CSS. 
- Remove unused styles from `subscriptions.sass`
- Clean up whitespace in a number of files


Changes affecting experimental features:

- Booststrap CSS build! [Details here](https://docs.saaspegasus.com/experimental#bootstrap-css-support).
- Building for multiple CSS frameworks now includes Bootstrap (along with Bulma and Tailwind)
- Building for Tailwind now overwrites Bulma templates instead of maintaining them in a separate
  directory.

*January 26, 2021*

## Version 0.12.0

This release is a set of changes laying the groundwork for future Pegasus improvements.

*Existing Pegasus users will need to upgrade their installer to use the latest version
(`pip install --upgrade pegasus-installer`)*

**Experimental support for Tailwind CSS**

More details can be found in the [Tailwind documentation](/experimental).

*Note: this feature is not yet fully supported and is in an experimental pre-release.*

**Simplified upgrade process**

Pegasus now saves your configuration when you install it, to simplify the upgrade process.
Instructions on upgrading can be found on the ["upgrading" page](/upgrading).

**Additional related changes and fixes**

- Remove `package-lock.json` from Pegasus for improved compatibility across NPM / OS versions. 
  It is recommended to run `npm install` and then check in the resulting `package-lock.json` file to source control.
  More details can be found in the [front end docs](/front-end). 
- Switch some styling from Sass variables to CSS variables (e.g. colors) for compatibility with multiple CSS frameworks
- Add a `site-base.sass` file for base Pegasus styles that aren't dependent on any CSS framework.
- Fix issue with `PlanSerializer` and `dj-stripe` version 2.4.1
- Remove no-longer-used `subscription_title.html` template.
- Remove no-longer-used tooltip styles
- Remove unused HTML from subscriptions upgrade page and split it out into partial templates
- Change a few default settings, e.g. project and author name.
- Upgrade django to 3.1.5.

*January 13, 2021*

## Version 0.11.3

This release overhauls the Payments example to use the Stripe `PaymentIntent` API 
and [support 3DS / SCA cards](https://stripe.com/docs/strong-customer-authentication).

Minor changes:

- Upgrade `dj-stripe` to `2.4.1`
- Pull `DJSTRIPE_WEBHOOK_SECRET` setting from environment variable if available 

If you are upgrading from a previous installation, you may need to change
the following two values in `settings.py`:

```python
DJSTRIPE_FOREIGN_KEY_TO_FIELD = 'id'  # change to 'djstripe_id' if not a new installation
DJSTRIPE_USE_NATIVE_JSONFIELD = True  # change to False if not a new installation
```

*December 17, 2020*

## Version 0.11.2

- Fixes to subscription workflow when using a trial period - automatically refresh page after
  card is accepted and support 3D-Secure validation for credit cards when using a trial. 

*December 9, 2020*

## Version 0.11.1

- Fix bug in development Dockerfile from a new OS dependency introduced by the `django-allauth` upgrade.

*December 7, 2020*

## Version 0.11.0

This release is a grab-bag of fixes and upgrades based on recent feedback.

- Force users to reconfirm their email when changing it and email verification is enabled
- Switch from `extract-text-webpack-plugin` to `mini-css-extract-plugin` for CSS handling in Webpack
- Rename `assets/index.js` to `assets/site.js` to support more styles in the future.
- Use randomly generated identicons for users instead of a default empty profile picture
- Move `.env.dev.example` to `.env.dev` and `.env.production.example` to `.env.production` so
  they don't have to be copied to get running (Docker builds only)
- Clean up payments UI a little
- Remove accidentally included `team_admin.html` template when not building with teams
- Remove accidentally included `Dockerfile.dev` template when not building with Docker
- Remove unused CSS classes from examples 
- Upgrade various NPM packages to latest versions
- Upgrade `django-allauth` to `0.44.0`
- Upgrade `django` to `3.1.4`

*December 2, 2020*

## Version 0.10.5

This release adds native support for deploying to Digital Ocean app platform.
See the [deployment guide](/deployment/#digital-ocean) for details.

Additional updates:

- Remove duplicate `ACCOUNT_EMAIL_VERIFICATION` declaration in `settings.py`
- Rename development `Dockerfile` to `Dockerfile.dev` for clarity and ease of deployment to other platforms
- Fix SSL / mixed content errors when deploying on Google Cloud Run

*Nov 17, 2020*

## Version 0.10.4

This release adds experimental native support for deploying to Google Cloud Run.
More details can be found in the [deployment guide](/deployment/#google-cloud).

Additional updates:

- Fix static file references to favicons.
- Upgrade `certifi` to `2020.11.8`
- Rename `heroku-requirements.txt` to `prod-requirements.txt` to be consistent across platforms (Heroku builds only)
- Switch production Docker images from Python-alpine to Python-slim (Docker builds only)

*Nov 10, 2020*

## Version 0.10.3

This release adds native support for deploying to Heroku using Docker containers.
More details can be found in the new [deployment guide](/deployment/).

Additional minor updates:

- Upgrade `urllib` to `1.25.11`
- Allow requirements to load from multiple sources when using Docker
- Add static directories and config files to `.gitignore`

*Nov 5, 2020*

## Version 0.10.2

This release adds support for using [Docker](https://www.docker.com/) in development.
More details can be found in the new [Docker documentation](/docker/).

*Oct 28, 2020*

## Version 0.10.1

This release adds [Heroku](https://www.heroku.com/) deployment support.
More details can be found in the new [deployment guide](/deployment/).

*Oct 26, 2020*

## Version 0.10.0

This is largely a maintenance release with mostly minor updates and fixes, 
but there are enough library upgrades that it warrants a version bump.

- Upgrade all Python packages including upgrading to Django 3.1.2
- Upgrade all npm packages
- Add a new API for products and metadata at `/subscriptions/api/active-products/`.
  Also includes adding some serialization classes and helper functions to subscriptions models.
  *Subscriptions only.* 
- Fix a bug where clicking on "Dashboard" didn't always take you to the right team.
  Also set team ID in the request session so it can be accessed across requests, and
  add `get_default_team` helper function to pull the last/current team from a request.
  *Teams only.*
- Fix default styling of textarea widgets in Django forms
- Use `const` and `let` in subscriptions page JavaScript
- Add `has-vcentered-cells` table formatting class to center table rows, and use
  in teams UI and object lifecyle demos
- Remove unnecessary `subscriptions.sass` file when Pegasus is built without subscriptions enabled

*Oct 14, 2020*

## Version 0.9.0

This release is a large overhaul of the React example that ships with Pegasus, including:

- Add url-routing support. Add/edit URLs now update and are linkable. 
  This also enables back button support
- Add validation feedback missing / bad data
- Switch all components from using classes to [hooks](https://reactjs.org/docs/hooks-intro.html)
- Split React components out into their own files
- Better loading UX

Other minor updates:

- Upgrade Django to 3.0.10 (3.1 support coming soon)
- Upgrade various JavaScript dependencies
- Generate random `SECRET_KEY` for each new installation
- Upgrade to Bulma 0.9.0
- Remove some spacing utility classes in favor of the ones that ship with bulma

Upgrading:

**Existing Pegasus users will need to upgrade the installer to run this.**
```
pip install --upgrade pegasus-installer>=0.0.2
```

*Sep 4, 2020*

## Version 0.8.3

- Fix default styling of number inputs in forms.
- Default `ACCOUNT_CONFIRM_EMAIL_ON_GET` to `True` if using email confirmation.
- Fix issue building front end on certain newer versions of nodejs/npm
- Upgrade `celery-progress` from `0.0.10` to `0.0.12`


*Aug 31, 2020*

## Version 0.8.2

This release fixes an edge case in the invitation accepting logic that didn't work if
a user left the page and came back later. 

*Aug 24, 2020*

## Version 0.8.1

A minor maintenance/bugfix release:

- Fixed a bug in the invitation workflow that prevented invitations from being accepted
  when creating accounts with social logins. Note that this requires changing the `ACCOUNT_ADAPTER`
  setting to  `'apps.teams.adapter.AcceptInvitationAdapter'`
- Make site branding in the navigation stay visible on mobile devices
- Make it more obvious that `settings.SECRET_KEY` should be overridden.
- Upgrade Django to 3.0.9

*Aug 21, 2020*

## Version 0.8.0

Stripe billing portal integration is here!

You can now use Stripe's new [customer portal](https://stripe.com/docs/billing/subscriptions/customer-portal)
to manage subscriptions in your app.

Supported operations and configurations include upgrading and downgrading subscriptions,
subscription cancellations (both immediately, and at period end), and subscription
renewals.

Additional documentation can be found in the [subscription docs](/subscriptions/).

Details and other changes:

- Added "manage billing" redirect to subscription pages if you have an active subscription,
  which goes to the Stripe customer portal
- Added webhook functionality to sync subscription changes and cancellations made via the Stripe portal
- Removed built-in "Cancel" option and supporting code.
- Upgraded `stripe` library to version `2.48.0`
- Removed the `.customer` field from `CustomUser` object. Customers are now always accessed via their
  associated subscriptions. 
- Removed a lot of no-longer-needed code from `SubscriptionModelMixin` related to accessing subscriptions
  from the `.customer` field. References to `.stripe_subscription` should be changed to simple `.subscription`
  on the `CustomUser` and `Team` objects.
- Added `avatar` to `CustomUser` admin.
- Fixed bug where "http://" was incorrectly assigned to the `Site.domain` object (fixes issues using `absolute_url`)

*July 16, 2020*

## Version 0.7.4

This is another minor release with mostly small fixes and updates to the front end.

- Added number formatting of Salary to object demo examples
- Fixed styling of number form inputs
- Removed unnecessary imports from `assets/index.js`
- Fixed incorrect distinction between `dependencies` and `devDependencies` in `package.json`
- Upgraded React to 16.13.1
- Moved React object lifecycle example to a `react` subfolder
- Started splitting up React object lifecycle demo into multiple files, and refactoring it to use hooks
- Renamed React object lifecycle example bundle file from `object-lifecycle-bundle.js` to `react-object-lifecycle-bundle.js`
- Removed unused "bower_components" exclusion in `webpack.config`

*July 13 2020*

## Version 0.7.3

- Upgraded to Django 3.0.7
- Fixed display of renewal details in subscription view to work with the latest Stripe Prices API
- Added ability to resend invitations from the team management page
- Added sort order to team member list (by email address)
- Cleaned up teams JavaSript (removed console logging statements, updated whitespace, removed commented code)

*June 30 2020*

## Version 0.7.2

- Improved styling of Stripe credit card forms in subscriptions and payments examples
- Fixed bug in subscriptions where not setting a default plan prevented the UI from working
- Fixed bug where monthly subscriptions would not work if you also had a quarterly or 6-month price configured
- Changed the order of some examples in examples home page and navigation

*June 17 2020*

## Version 0.7.1

- **Added ability to cancel a Subscription directly on the site.**
  [Demo](https://www.saaspegasus.com/subscriptions/) (you have to create a Subscription first)
- Don't show password change links if using social authentication (thanks Yaniv!)

*June 11, 2020*

## Version 0.7.0

Pegasus now supports [Vue.js](https://vuejs.org/)!

Version 0.7 adds a Vue.js implementation of the Object Lifecycle demo so you can
start with a foundation of either React or Vue.

Minor changes:

- Added `Membership` inline admin editing to `Teams` model (thanks Troels!)
- Added a few more spacing utility css classes to `utilities.sass`

*June 5, 2020*

## Version 0.6.1

- Upgrade `requests` to version `0.23.0` to fix installation version conflict.

*May 26, 2020*

## Version 0.6.0

This release begins the move of Pegasus's core functionality out of the "pegasus" app and into
the user-defined apps. It's a relatively big release from a code perspective, although there is very
little new / changed in terms of functionality.

### Philosophy

The philosophy guiding this change is "your starting code base should be as understandable as possible".

Historically, Pegasus has attempted to separate "Pegasus-owned" files from "user-owned" files.
The thinking behind this structure was that Pegasus upgrades and code merges would be as easy as possible since - 
in theory - users would not need to modify anything in the "Pegasus-owned" space.

In practice, the line between "Pegasus-owned" and "user-owned" was fuzzy, and customizing
an application often required editing files in the "Pegasus-owned" space.
So the benefit was not realized.

Furthermore, this split caused the initial codebase to be more confusing, since core functionality was split
across two places.

### Changelog

Changes related to the restructure above include:

- Moved all base templates from `templates/pegasus/` to `templates/web/`
- Moved team invitation templates from `templates/pegasus/email/` to `templates/teams/email/`
- Merged `pegasus/apps/users` and all related code into `apps/users`
- Merged `pegasus/apps/teams` and all related code into `apps/teams`
- Removed `pegasus/apps/components` and moved code to more specific apps - further details below
- Moved `promote_user_to_superuser` management command into `users` app
- Moved `bootstrap_subscriptions` management command into `subscriptions` app
- Moved `google_analytics_id` context processor to web app
- Moved `meta.py` from pegasus app to web app
- Moved `pegasus/utils/subscriptions.py` to `apps/subscriptions/helpers.py`
- Added `apps.utils` and moved most of `pegasus.utils` there
- Moved `pegasus/decorators.py` to `apps/utils/decorators.py`
- Moved `PegasusBaseModel` to `apps.utils` and renamed to `BaseModel`
- Removed unused `stripe.py` file (replacing functionality with equivalent functions in `djstripe`)
- Removed "Pegasus" from API url names
- Moved non-example JS imports out of `pegasus/Pegasus.js` and into `App.js`
- Lowercased (and kebab-cased) all JS file names for consistency
- Moved sass files from `assets/styles/pegasus`, to `assets/styles/app`
- Renamed various layout classes, e.g. `pegasus-two-column`, `pegasus-message`, etc. to `app-[name]`
- Moved `static/images/pegasus/undraw/` folder to `static/images/undraw`
- Removed unused `pegasus/teams/serializers.py` file


Other changes and fixes include:

- Fix accepting an invitation if you're already signed in
- Remove subscription-related fields from team and user models if not using subscriptions
- Only ship base migration files for users and teams, and have applications manage their own migrations 
- Improved checks and error-handling around accepting invitations multiple times
- Upgraded `bulma` CSS framework to `0.8.2`
- Upgraded `node-sass` to `4.14.1`
- Ran `npm audit fix` to update other JS library dependencies
- Upgraded `pip-tools` and other packages in `dev-requirements.txt`
- Added backend check to the payments example, to show how to prevent client-side exploits
- Improve password reset email copy
- Use `$link` color in `$navbar-item-active-color` and `$menu-item-active-background-color`
- Externalized styles on progress bar demo

### A Note on Database Migrations

Historically, Pegasus has shipped with complete database migrations.
However, maintaining a set of migrations for each possible Pegasus configuration or forcing all configurations
to use the same DB schema has proven unwieldy. Thus, migrations are now expected to be managed *outside of Pegasus*.

For new users, the only change is that prior to running `./manage.py migrate` for the first time,
you must first run `./manage.py makemigrations`.

For existing users you can either keep your current migrations folder, or you can run 
`./manage.py makemigrations` and then `./manage.py migrate --fake`. If you have changed the user or team
models, then you should keep your current folder.

*May 19, 2020*

## Version 0.5.2

- Fixed default Postgres DB settings (adding host and port)
- Removed Stripe webhooks from project urls if not using subscriptions
- Fixed but where subscription and teams templates were still being included even if not enabled for a project
- upgrade `celery-progress` to 0.0.10
- Cleaned up progress bar demo
  - Used javascript cookie library for CSRF token
  - Switched from `var` to `const` in a few places
  - Removed debug logging statements

*May 7, 2020*

## Version 0.5.1

Version 0.5.1 is a minor maintenance release with a few minor bug fixes and bits of cleanup:

- Upgraded Django to 3.0.5
- Fixed bug where certain input types were getting overridden in Django Forms (thanks Yaniv for reporting!) 
- Fixed bug with Object Lifecycle and Charts demos not working on Team installations (thanks Greg for reporting!)
- Moved example chart JavaScript to the webpack pipeline and share Api access variables (thanks Greg for the suggestion!)
- Switch `admin.py`  files to use the `@admin.register` decorator syntax

*April 17, 2020*

## Version 0.5

This is the biggest release to Pegasus since it's launch.
Read below for all the details.

### Subscriptions!

Added the [Stripe Subscriptions feature](https://www.saaspegasus.com/content/subscriptions-overview).

Documentation for subscriptions can be [found here](/subscriptions).

#### Model changes

- Added a `subscription` field to `Team` and `CustomUser` objects, and a `customer` field to `CustomUser`.
- Added `SubscriptionModelMixin` helper class for accessing / checking subscription status on a model.

### Javascript build changes

- Added `Pegasus.js` and made different modules available in front end code 
  (see subscriptions upgrade page example usage).

### Sass / CSS changes

- Added `tooltip` utilities.
- Added a few margin helper classes (e.g. `my-1`, `my-2` )

### New Python Library Dependencies

- [attrs](https://www.attrs.org/en/stable/)
- [dj-stripe](https://dj-stripe.readthedocs.io/en/stable/)

### New JavaScript Library Dependencies

- [js-cookie](https://github.com/js-cookie/js-cookie)

### Small fixes and changes:

- Moved app-specific templates from inside the apps to global templates directory as recommended by 
  Two Scoops of Django
- Remove redundant raw prefix on some `path` url declarations
- Reduced some duplicate access to `team` object when already available via the `request` object.
- Made team permission template tags more consistent with rest of site (also allow access to superusers)
- Removed `PEGASUS_USING_TEAMS` and `pegasus_settings` context processor. All config is now handled at installation
  time instead of by settings variables.
- Catch Stripe card errors in the payments example
- Upgraded various `npm` packages
- Upgraded Bulma to 0.8.0

### Documentation

- Added release notes page (this one)
- Added [subscriptions overview page](/subscriptions)
- Updated "delete teams code" cookbook to reflect latest team changes 
  (all the backend work is now done for you on installation)

*March 30, 2020*

## Version 0.4 and earlier

Release notes for earlier versions are tracked at [https://www.saaspegasus.com/releases/](https://www.saaspegasus.com/releases/).
