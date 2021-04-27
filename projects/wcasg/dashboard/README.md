**Note: This README is extracted from the original source project that I created. Please see [WARNING & COPYRIGHT NOTICE](../../../README.md#warning--copyright-notice) for more info.**

- [Product :: Plan for Sites](#product--plan-for-sites)
  - [Stripe Product](#stripe-product)
  - [Stripe Plan](#stripe-plan)
  - [Dashboard Plans DB Table](#dashboard-plans-db-table)
- [JSON Schema](#json-schema)
  - [Example](#example)
- [Site Context Restraints](#site-context-restraints)
  - [Example](#example-1)
- [Roles](#roles)
  - [Admin: Tester (`admin-tester`)](#admin-tester-admin-tester)
    - [Permissions](#permissions)
    - [Setup](#setup)
- [Artisan Commands](#artisan-commands)
  - [`site:generate-statistics`](#sitegenerate-statistics)
  - [`user:create`](#usercreate)
    - [Creating a Basic User](#creating-a-basic-user)
    - [Creating an Activated User](#creating-an-activated-user)
    - [Specifying a Password](#specifying-a-password)
    - [Assigning a Plan](#assigning-a-plan)
- [Development](#development)
  - [Base App Installation](#base-app-installation)
  - [Setup Local Dev](#setup-local-dev)
  - [Starting Services](#starting-services)
  - [Testing](#testing)
    - [Troubleshooting: Not Hooking to Test DB](#troubleshooting-not-hooking-to-test-db)
  - [Deployment (Testing)](#deployment-testing)
  - [Deployment (Production)](#deployment-production)
    - [Services](#services)
  - [Remotely Connect to MySQL](#remotely-connect-to-mysql)
- [Widget API Setup](#widget-api-setup)
  - [Plans Table](#plans-table)
  - [Users Table](#users-table)
  - [Subscriptions Table](#subscriptions-table)
  - [Sites Table](#sites-table)
  - [Request / Response](#request--response)
  - [Examples](#examples)
- [Accessibility Statements](#accessibility-statements)
  - [Statement Templates](#statement-templates)
    - [Statement Template Syntax](#statement-template-syntax)
    - [Configuration](#configuration)
  - [Default Statement Template](#default-statement-template)
  - [Admin Statement Template Views](#admin-statement-template-views)
  - [Admin Statement Views](#admin-statement-views)
  - [Site-Specific Accessibility Statements](#site-specific-accessibility-statements)
  - [Backend](#backend)
- [Models / Relationships](#models--relationships)
  - [User](#user)
  - [Plan](#plan)
  - [Subscription](#subscription)
  - [Site](#site)
  - [Coupon](#coupon)
- [Database](#database)
  - [Reset](#reset)
- [File Storage](#file-storage)
- [Webhook Leads](#webhook-leads)
- [Site Statistics](#site-statistics)
  - [Amazon SQS](#amazon-sqs)
  - [Coeus Users](#coeus-users)
  - [TTS Request Statistics](#tts-request-statistics)
  - [DB & Collection](#db--collection)
- [Glossary](#glossary)
  - [Widget](#widget)
  - [Client](#client)
  - [Customer](#customer)
  - [User (or End-User)](#user-or-end-user)
  - [Dashboard](#dashboard)

## Product :: Plan for Sites

### Stripe Product

```json
{
  "id": "prod_H3ekPDcHD4OmUB",
  "object": "product",
  "active": true,
  "description": "Accessibility Widget",
  "name": "Accessibility Widget",
  "statement_descriptor": "Accessibility Widget",
  "type": "service",
  "unit_label": "site"
}
```

### Stripe Plan

```json
{
  "id": "plan_H3ekwrhfyH9NFI",
  "object": "plan",
  "active": true,
  "amount": 500,
  "amount_decimal": "500",
  "billing_scheme": "per_unit",
  "currency": "usd",
  "interval": "month",
  "interval_count": 1,
  "metadata": {
    "max_sites": 5
  },
  "nickname": "5 Sites per Month",
  "product": "prod_H3ekPDcHD4OmUB",
  "trial_period_days": 14,
  "usage_type": "licensed"
}
```

### Dashboard Plans DB Table

| id                  | product_id          | amount | currency | interval | nickname           | active | trial_period_days | context                                                                         | coupon_id                    |
| ------------------- | ------------------- | ------ | -------- | -------- | ------------------ | ------ | ----------------- | ------------------------------------------------------------------------------- | ---------------------------- |
| plan_HDj3a0y455rnsh | prod_HDj3KTwSH0UCXr | 5000   | usd      | month    | 5 Sites per month  | 1      | 0                 | {"plan": {"restraints": [{"value": 5, "entity": "site", "comparator": "<="}]}}  | NULL                         |
| plan_HDj3chzvuez7XI | prod_HDj3KTwSH0UCXr | 15000  | usd      | month    | 15 Sites per month | 1      | 0                 | {"plan": {"restraints": [{"value": 15, "entity": "site", "comparator": "<="}]}} | EARLY_ADOPTER_20_OFF_FOREVER |
| plan_HDj3dAstQCTPQj | prod_HDj3KTwSH0UCXr | 1000   | usd      | month    | 1 Sites per month  | 1      | 0                 | {"plan": {"restraints": [{"value": 1, "entity": "site", "comparator": "<="}]}}  | ALPHA_100_OFF_3_MONTH        |
| plan_HDj3ueUFkXAJdT | prod_HDj3KTwSH0UCXr | 25000  | usd      | month    | 25 Sites per month | 1      | 0                 | {"plan": {"restraints": [{"value": 25, "entity": "site", "comparator": "<="}]}} | NULL                         |

## JSON Schema

- Schema: [resources/schemas/wcasg.json](resources/schemas/wcasg.json)
- Specification: http://json-schema.org/specification.html
- Tests: [tests/Unit/Schema/SchemaTest.php](tests/Unit/Schema/SchemaTest.php)

This schema is used to define resource-specific business logic that can be saved alongside a database record in JSON format. Using JSON Schema specification allows us to validate data before saving to the database.

Each `restraint` object forms a tuple used for business logic validation against the database model containing the given JSON object.

### Example

```json
{
  "plan": {
    "restraints": [
      {
        "entity": "site",
        "comparator": "<=",
        "value": 5
      }
    ]
  }
}
```

Here, a `Plan` database model may have the above JSON object associated with it in the database. This informs the rest of the app to perform a `restraint` check against `Site` models related to the `Plan` instance and ensure there are `5` or fewer.

## Site Context Restraints

The `sites` database table's `context` field supports a JSON context object using the [JSON Schema](#json-schema) specification. Any Dashboard request that attempts to alter the number of active `Sites` is validated against all `Plans` subscribed by the `User`. Any `Plan` with a valid `context` object is validated against the requested `Site` change.

### Example

The following `subscriptions` table record indicates `User` `1` has an active subscription with a `Plan` id of `plan_H4Hm2JT2SJQDQs`:

| id                 | plan_id             | user_id | stripe_status |
| ------------------ | ------------------- | ------- | ------------- |
| sub_H4HmCDl8z5gWfV | plan_H4Hm2JT2SJQDQs | 1       | active        |

The matching `plans` table entry shows the details of the `Plan`, including its `context` JSON object defined by the project [resources/schemas/wcasg.json](resources/schemas/wcasg.json) schema:

| id                  | product_id          | nickname | active | context                                                                        |
| ------------------- | ------------------- | -------- | ------ | ------------------------------------------------------------------------------ |
| plan_H4Hm2JT2SJQDQs | prod_H4HmTlAFxIur1P | Monthly  | 1      | {"plan": {"restraints": [{"value": 5, "entity": "site", "comparator": "<="}]}} |

In this case, the only `restraint` is a limit on the number of active `Sites`, which must be `5` or fewer.

The `sites` table shows the collection of `Sites` assigned to this user's `Subscription`:

| id  | domain         | active | name                         | token                                | subscription_id    |
| --- | -------------- | ------ | ---------------------------- | ------------------------------------ | ------------------ |
| 1   | localhost:84   | 1      | Koepp, Bins and Feeney       | 30c818375b2ee1fbfb37f8a5cc2e8d05b5bb | sub_H4HmCDl8z5gWfV |
| 3   | jakubowski.com | 1      | Bogan, Stamm and Zboncak     | 89fa6356b0506342d82988af0368f5f14260 | sub_H4HmCDl8z5gWfV |
| 4   | mertz.com      | 0      | Maggio, Emmerich and Kreiger | 75d93c400f1c82cdeee75828289e60cb55d7 | sub_H4HmCDl8z5gWfV |
| 5   | boyle.com      | 1      | Romaguera LLC                | 1f4badc60238adeb4bae365f5b3c9c71878b | sub_H4HmCDl8z5gWfV |
| 6   | russel.com     | 1      | Wolf-McKenzie                | 81f460bb9956a7f40de35ee7720022757824 | sub_H4HmCDl8z5gWfV |
| 7   | lynch.com      | 1      | Roberts-Konopelski           | 9fd37afd37f792168f933f1eda2aa9830d62 | sub_H4HmCDl8z5gWfV |

The user currently has `5` **active** sites, so attempting to add a new `Site` _or_ alter `Site.id = 4` to an active state fails and returns an error to the user indicating their current `Plan` does not support any additional sites.

## Roles

### Admin: Tester (`admin-tester`)

Description of role

#### Permissions

- `ignore-site-limit`: Ignores site limits

#### Setup

1. Create `admin-tester` role
2. Create `ignore-site-limit` permission
3. Assign `ignore-site-limit` permission to `admin-tester` role

## Artisan Commands

### `site:generate-statistics`

Generates fake Site request statistics for Sites owned by special users (devs, admins, etc).

- **Signature:** `php artisan site:generate-statistics {count?}`

Optional `count` argument determines the number of fake requests generated **per Site**. Defaults to `10` requests per Site.

[MockRequests](tests/Mock/Request.php) are used to generate each request record, which includes fake elements and emulates a random `timestamp` within the past year.

### `user:create`

Create a new User with the specified `email` address and optional password, plan, coupon, and activation status.

- **Signature:** `php artisan user:create {email} {password?} {--activated : Activate user.} {--plan= : Plan id to assign to user.} {--coupon= : Coupon id to apply to subscription.}`

#### Creating a Basic User

```
$ php artisan user:create john2@example.com
Creating new user of id 1 and email john@example.com.
Generated password: 3T81tQDANmIo
Save this password somewhere secure as it cannot be recovered.
Sending activation email.
```

#### Creating an Activated User

```
$ php artisan user:create john@example.com --activated
Creating new user of id 1 and email john@example.com.
Generated password: u5UEkIOxtE3W
Save this password somewhere secure as it cannot be recovered.
```

#### Specifying a Password

```
$ php artisan user:create john@example.com password --activated
Creating new user of id 1 and email john@example.com.
```

#### Assigning a Plan

```
$ php artisan user:create john@example.com --activated --plan=plan_HMGmstfZ3MRFtr
Creating new user of id 1 and email john@example.com.
Generated password: 6R5fJ0lae5gV
Save this password somewhere secure as it cannot be recovered.
Creating subscription.
```

## Development

### Base App Installation

1. Copy old commit to directory.
2. `npm install`
3. Install [Composer](https://getcomposer.org/)
4. `composer install`
5. `php artisan key:generate`
6. `php artisan migrate:fresh`
7. `php artisan db:seed`
8. `php artisan passport:install`
9. `php artisan storage:link`

### Setup Local Dev

1. `git clone` repo.
2. Copy content of the [.env.development](https://gitlab.solarixdigital.com/solarix/wcasg/dashboard/snippets/13) snippet to the a `<root>/.env` file.
3. Run `composer install`.
4. Run `yarn run db:dev` to create a local MySQL Docker.
5. Open `http://localhost:8090/`, login with `root:example`, and add a new database named `voydev_wcasgadmin`.
6. Add db user specified in `.env`.
7. Run `yarn run db:reset` to run all db migrations and seed the database.
8. Run `php artisan storage:link` (just once).
9. Run `yarn run dev` to rebuild the client app, then navigate on your local PHP server to `localhost` to view the site.

### Starting Services

1. Open SSH tunnel to remote MongoDB: `yarn run dev:db:mongo:ssh-tunnel`.
2. Start up local `pa11y-webservice-laravel` app. If installed in sibling directory (i.e. `../audit`) it can be run with `yarn run dev:webservice:start`.
3. Start up local Laravel work queue to handle jobs: `yarn run artisan:queue:work`.
4. Start watcher to monitor for JS / view changes: `yarn run watch`.

### Testing

- Run `yarn run test` to execute PHPUnit test suite.

Tests will begin by deleting, rebuilding, and seeding the test database, as defined in `phpunit.xml` (default is MySQL `voydev_wcasgadmin_testing`).

#### Troubleshooting: Not Hooking to Test DB

- Double-check `phpunit.xml`, `.env`, and `config/database.php` settings.
- Clear configuration cache: `php artisan config:clear`.

### Deployment (Testing)

1. Push changes to new repository branch.
2. GitLab CI will create a new pipeline and migrate the new build to the testing server: https://dashboard.widget.wcasg.solarix.dev

- (Optional) If a testing environment database reset is required this can be performed as part of a `testing` pipeline. Open the pipeline page in GitLab and manually execute the `testing:build:db:reset` Build stage job.

### Deployment (Production)

1. Create a merge request to `master` branch.
2. Upon MR acceptance, GitLab CI will push new build to production: https://dashboard.widget.wcasg.solarix.host

#### Services

- Ensure `php artisan:queue:work` is running to handle Audit services.
- Ensure `php artisan:queue:work:sqs` is running to handle Statistic/Coeus services.

### Remotely Connect to MySQL

Both testing (`srn:rds:wcasg:widget:wcasg:testing::db/mysql`) and production (`srn:rds:wcasg:widget:wcasg:production::db/mysql`) RDS instances use the Solarix Dev MySQL security group (`srn:vpc:solarix::sg/dev-mysql`) to provide external access.

1. Add your local public IP address to the `srn:vpc:solarix::sg/dev-mysql` security group's **Ingress** rules for port `3360`.
2. You can use http://checkip.amazonaws.com/ to get your IP.
3. With the AWS CLI you can get the `group-id` of the `srn:vpc:solarix::sg/dev-mysql` security group:

```
$ aws ec2 describe-security-groups --filters Name=tag:srn,Values=srn:vpc:solarix::sg/dev-mysql --query "SecurityGroups[0].[GroupId]" --output text

sg-00cb767069e9979fc
```

4. Authorize `TCP` on `3306` port to your IP with the following command (replace `123.123.123.123` with your IP) and verify the same `group-id` as returned above:

```
aws ec2 authorize-security-group-ingress --group-id=sg-00cb767069e9979fc --protocol tcp --port 3306 --cidr 123.123.123.123/32
```

5. The RDS servers will now accept incoming connections to port `3306` from your IP.

## Widget API Setup

The `<root>/api/widget` endpoint is the primary entry for all incoming `Widget` JavaScript requests made by individuals viewing a `User's` website with a `<script src='<root>/api/widget' crossorigin='anonymous' defer></script>` tag injected.

Consider the following table entries (some columns truncated for simplicity):

### Plans Table

| id  | name    | stripe_plan_id     | interval | amount | active | trial_period_days |
| --- | ------- | ------------------ | -------- | ------ | ------ | ----------------- |
| 1   | Primary | ipsum_ducimus_fuga | month    | 325.00 | 1      | 3                 |

### Users Table

| id  | first_name | last_name | username | email                   | activated |
| --- | ---------- | --------- | -------- | ----------------------- | --------- |
| 1   | Gabe       | Wyatt     | gabestah | gabe@solarixdigital.com | 1         |

### Subscriptions Table

| id  | user_id | name | stripe_id          | stripe_status | stripe_plan_id     | plan_id |
| --- | ------- | ---- | ------------------ | ------------- | ------------------ | ------- |
| 1   | 1       | main | sub_GgT5ufj9MFaZZ5 | active        | ipsum_ducimus_fuga | 1       |

### Sites Table

| id  | domain        | active | token                               | subscription_id |
| --- | ------------- | ------ | ----------------------------------- | --------------- |
| 3   | gabewyatt.com | 1      | iVGUxr5vwxBypx6oOtQvB4YvHB8eyIT9Ecl | 1               |

1. An `Admin` creates the `Primary` plan, which generates the `stripe_plan_id` value and creates a matching plan via the Stripe API.
2. The `User` (`gabe@solarixdigital.com`) subscribes to the `Primary` plan, which generates the `Subscription` model entry associated with that `Primary` plan. It also creates a matching subscription via the Stripe API (`sub_GgT5ufj9MFaZZ5`).
3. The `User` adds a `Site` to their new `Subscription`, entering a domain of `gabewyatt.com`. This auto-generates a unique `token` for that `Site` record.
4. The `User` copies the provided `<script>` tag `/api/widget` endpoint with the associated `token` and pasted it into the header of their website:

```html
<script
  crossorigin="anonymous"
  src="https://app.wcasg.com/api/widget/iVGUxr5vwxBypx6oOtQvB4YvHB8eyIT9Ecl"
  defer
></script>
```

### Request / Response

Any request made to `/api/widget` is handled by the [CheckValidWidgetRequest](app/Http/Middleware/Api/Widget/CheckValidWidgetRequest.php) middleware. This middleware completes a series of validations and authorization checks to determine if the request is valid and matches an active subscription. If any check fails, an appropriate exception is thrown and the API response sends a short `<script></script>` snippet that outputs the error message to the end-user's console log.

1. It first checks for the `origin` request header, which is only present for `GET` requests (the type of request made by `<script>` tags) that have `CORS` enabled. Thus, the need for the `crossorigin="anonymous"` attribute in the script widget provided to `Customers`. If no `origin` is found, a [`MissingOrigin`](app/App/Exceptions/Api/Widget/MissingOriginException.php) exception is thrown and outputs the [localized message](resources/lang/en/error.php) for that error.
2. A missing `token` parameter is checked next.
3. Authentication is based on the passed `token` found within the `Sites.token` database entries. If no match is found an `InvalidToken` exception is thrown.
4. The `host` value of the `Sites.domain` field is compared to the `host` value of the `origin` header from which the request originated. A match continues and a failure throws `InvalidOrigin`.
5. The `Site` is then checked to ensure it `isActive`.
6. The final check is to ensure the matching `Sites` entry is associated with a `Subscription` that is active / valid. Failure throws an `InvalidSubscription` error.

If all middleware checks succeed the [WidgetController](app/Http/Api/Controllers/Widget/WidgetController.php) handles the response by pulling the Widget script contents from local storage, assigning appropriate headers, and responding with the full script content that is then injected into the `Customer's` site.

### Examples

Below are examples are the API responses provided to the end user based on various forms of `<script>` tags, assuming database values are as seen above.

- `<script src="http://app.wcasg.com/api/widget/iVGUxr5vwxBypx6oOtQvB4YvHB8eyIT94Ecl" defer></script>`: Console output: `[WCASG]: Request origin cannot be determined. Cross Origin request has failed.`
- `<script crossorigin="anonymous" src="http://app.wcasg.com/api/widget" defer></script>`: Console output: `[WCASG]: A token parameter is required to complete this request.`
- `<script crossorigin="anonymous" src="http://app.wcasg.com/api/widget/1234567890" defer></script>`: Console output: `[WCASG]: Token parameter is invalid.`
- `<script crossorigin="anonymous" src="http://app.wcasg.com/api/widget/iVGUxr5vwxBypx6oOtQvB4YvHB8eyIT94Ecl" defer></script>`: All checks pass and Widget script is returned and rendered in browser.
- Same valid script tag as above, but it is being injected into a page hosted on the domain `hackers.net`. Console output: `[WCASG]: Request origin is invalid.`
- Same valid script tag as above, but this time the `Subscription` of id `1` is set to invalid. Console output: `[WCASG]: The subscription associated with this site is disabled or invalid.`

## Accessibility Statements

### Statement Templates

Accessibility statements start as a `Statement Template` model instance. Seed data contains a few initial templates (as seen in [resources/assets/templates/](resources/assets/templates/)).

#### Statement Template Syntax

A template can be any combination of plain text, valid HTML tags, and custom `{% %}` tags used to inject runtime data.

`{% %}` tags must enclose an `object` `name` and a bracketed `key`. Each valid object refers to a collection of relevant data. `Object name` can be any of the following:

- `app`: The Dashboard application config. For example, `app['app.url']` inserts the URL of the Dashboard app. See [Laravel config](https://laravel.com/docs/master/helpers#method-config) and local [config](config) directory.
- `config`: The custom `Statement` configuration, otherwise the parent `Statement Template` configuration. For example, `{% config['includeContactInfo'] %}` inserts the current configuration value of the `includeContactInfo` key.
- `site`: The `Site` model object that is using this `Statement`. For example, `{% site['domain'] %}` inserts the `Site's` domain.
- `subscription`: The `Subscription` model object associated with `Site`. For example, `{% subscription['ends_at'] %}` inserts the ending date of the `Subscription`, if applicable.
- `timestamp`: Current timestamp.
- `user`: The owning `User` model object of the associated `Site`. For example, `{% user['email'] %}` inserts the email address of the owning `User`.

A custom `{% %}` tag can surround any valid `object['key']` combination and be inserted anywhere in the statement HTML, e.g.:

```html
<h1>{% site['name'] %} Accessibility Statement</h1>
<p>Updated: {% timestamp %}.</p>
<p>
  {% site['domain'] %} makes available the
  <a
    href="{% app['app.url'] %}"
    alt="{% app['app.name'] %} Widget"
    title="{% app['app.name'] %} Widget"
    >{% app['app.name'] %} Widget</a
  >
  that is powered by a dedicated accessibility server. The software allows {%
  site['domain'] %} to improve its compliance with the Web Content Accessibility
  Guidelines (WCAG 2.1).
</p>
```

Some of Laravel Blade's [directives](https://laravel.com/docs/master/blade#control-structures) can be used, but are sanitized to prevent malicious code injection. The following directives are allowed:

- `if`
- `unless`
- `isset`
- `empty`

A directive can be used to check the truthy/falsey value of a custom `{% object['key'] %}` value and optionally render or discard the enclosed content. For example:

```html
@if({% config['includeContactInfo'] %})
<h2>Contact Us</h2>
<p>
  If you wish to report an accessibility issue, have any questions or need
  assistance, please contact {% site['domain'] %} Customer Support as follows:
</p>
<p>Email: <a href="mailto:{% user['email'] %}">{% user['email'] %}</a></p>
@endif
```

The parser will initially replace inner content such as `{% site['domain'] %}` and `{% user['email'] %}`, then it will perform validation of the `@if ... @endif` directive. In this case, an `if` directive checks for a truthy value, so if the result of `{% config['includeContactInfo'] %}` is truthy the inner content will be included in the rendered statement.

The following directives are disabled for security purposes.

- `php`
- `for`
- `foreach`
- `forelse`
- `while`

**Any text within a disabled directive will be discarded and not rendered.**

#### Configuration

Each `Statement Template` has a `default_config` that is a JSON object defining the initial configuration settings for all sites/statements that will use this template.

```json
{
  "includeContactInfo": true
}
```

An `Admin` can add or modify `Statement Templates` under the `Admin > Statements` dashboard view.

### Default Statement Template

Once instance of an existing `Statement Template` is always marked as the current `default_template_statement` in the `settings` db table. This ensures that `User`-generated `Sites` will always have a default `Statement Template` to fall back to.

An `Admin` will receive an error if attempting to delete the current default `Statement Template` and must assign the default to a different template before deletion.

### Admin Statement Template Views

- [Viewing](https://dashboard.widget.wcasg.solarix.dev/admin/statement-templates/1) a `Statement Template` as an `Admin` shows the HTML-ified template prior to dynamic data injection.
- [Editing](https://dashboard.widget.wcasg.solarix.dev/admin/statement-templates/1/edit) a `Statement Template` allows the `Admin` to change the `name`, `is_default`, `default_config`, and template `content` via a WYSIWYG editor. As with the default, Laravel Blade syntax is mixed with the properly rendered HTML in the editor, allowing for advanced data injection.
  - The `is_default` field is disabled if the currently edited template is already the default to prevent user from trying to unset the sole default without first assigning a new default.
- At present, `Admin` can only delete a `Statement Template` not assigned to anything else (`Statements`).
- The [create](https://dashboard.widget.wcasg.solarix.dev/admin/statement-templates/create) `Statement Template` view uses the `default.html` template, but can be overridden by `Admin` as needed.

### Admin Statement Views

`Admin` can also view and adjust some `Statement` settings.

### Site-Specific Accessibility Statements

From within the [account/sites](https://dashboard.widget.wcasg.solarix.dev/account/sites) view a `User` can adjust the accessibility `Statement` assigned to each `Site`.

- [Viewing](https://dashboard.widget.wcasg.solarix.dev/account/sites/1/statement) the `Statement` assigned to a `User's` `Site` shows the fully-rendered, injected statement content based on the current `User`, `Site`, `Statement`, and parent `Statement Template`.
  - The `User` can download a [text](https://dashboard.widget.wcasg.solarix.dev/account/sites/1/statement/download/text) or [html](https://dashboard.widget.wcasg.solarix.dev/account/sites/1/statement/download/html) copy of the rendered statement.
- The [Edit](https://dashboard.widget.wcasg.solarix.dev/account/sites/1/statement/edit) view shows the rendered statement using the currently-assigned `Statement Template`. Choosing a different template in the dropdown shows a preview of the `Statement` content using the selected template.
  - Here the `User` can also customize the `Statement` configuration (currently a JSON object. **TODO: Update to a more user-friendly form**). The default configuration for the parent `Statement Template` will be used unless the `User` overrides the configuration with their own `config` settings.

### Backend

## Models / Relationships

This section describes models and relationships as they are developed.

### User

- Globally unique
- Identified by `email`
- Can select one or more `Plans` to pay for, each of which creates a unique `Subscription` associated with `User`.

### Plan

- Globally shared
- Directly created by `Admin`
- Details a payment model that `User` can pay for via a `Subscription`, e.g.:

| Name          | Interval | Price | plan type   | Site limit | Team limit | Date           | Status |
| ------------- | -------- | ----- | ----------- | ---------- | ---------- | -------------- | ------ |
| Basic Plan    | month    | 5.00  | Normal plan | 1          |            | 11 minutes ago | Active |
| Advanced Plan | month    | 15.00 | Normal plan | 3          |            | 9 minutes ago  | Active |

### Subscription

- Unique to `User`
- Indirectly created by `User` when signing up for a selected `Plan`.
- Associated with one-to-many `Site`.
- While `Subscription` is `active`, API requests to `api/widget:get()` that originate from an associated `Site` respond with a valid Widget JavaScript injection snippet.

### Site

- Unique to `User` and `Subscription`.
- Directly created by associated `User` and related to selected `Subscription`.
- Associated with one `Subscription` and one `User`.
- Each `Site` can be assigned to a given `Subscription` by `User`.
- Multiple `Sites` can be associated with a `Subscription` and `User`, but no more than `Subscription > Plan > SiteLimit` `Sites` can be `active` at one time.

### Coupon

| Column             | Type                               | Comment                                                                                                            |
| ------------------ | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| id                 | varchar(50)                        |                                                                                                                    |
| currency           | enum('usd') [usd]                  |                                                                                                                    |
| duration           | enum('forever','once','repeating') | Describes how long a customer who applies this coupon will get the discount.                                       |
| duration_in_months | int NULL                           | If duration is repeating, the number of months the coupon applies. Null if coupon duration is forever or once.     |
| max_redemptions    | int NULL                           | Maximum number of times this coupon can be redeemed, in total, across all customers, before it is no longer valid. |
| metadata           | json NULL                          |                                                                                                                    |
| name               | varchar(255)                       | Name of the coupon displayed to customers on for instance invoices or receipts.                                    |
| percent_off        | double(8,2)                        | Percent that will be taken off the subtotal of any invoices for this customer for the duration of the coupon.      |
| redeem_by          | timestamp NULL                     | Date after which coupon can no longer be redeemed.                                                                 |
| valid              | tinyint(1) [1]                     | Taking account of the above properties, whether this coupon can still be applied to a customer                     |

- `id` is the promo/coupon code identifier and must be a unique string.
- Coupon model is generally a 1-to-1 match of the Stripe Coupon counterpart.
- [CouponObserver](app/Domain/Coupon/Observers/CouponObserver.php) performs necessary Stripe Coupon updates for Coupon model lifecycle events.
- `Coupon.toString()` is a convenience for getting a human-readable string indicating what the Coupon does (e.g. `100% Off, First 3 Months`).
- Each Coupon must be associated with one or more Plans to have any affect. New Subscriptions generated from a Coupon-associated Plan will have the discount applied.

## Database

### Reset

- Run `yarn run db:reset`. **WARNING**: Deletes existing data, reruns migrations, and reseeds.

## File Storage

`Disks` are used to store files locally or remotely based on application site.

```php
# Saves a file to 'local' disk.
Storage::disk('local')->put('file.txt', 'Contents');
# Check if a file exists on 'widget' disk.
Storage::disk('widget')->exists('file.jpg');
```

- `local`: All other files.
- `widget`: All Widget-related files.
- `public`: All publicly available files.

See: https://laravel.com/docs/5.8/filesystem

## Webhook Leads

- Webhook request is made to webhook endpoint with private `token`, `type`, and `id`, e.g. `/api/webhook?token=fae376381710061bc7052892c4485ffbc174&id=plan_HDj3chzvuez7XI&type=funnel`
- Request body contains some identifying information that can be used to verify the Stripe transaction. In the example below, the `id_at_gateway` corresponds to a Stripe `BalanceTransaction` object id:

```json
{
  "transaction": {
    "gateway": "stripe",
    "id_at_gateway": "txn_1GiN2XAIFSjPGiCSPwFWEPRQ",
    "status_at_gateway": "succeeded"
  }
}
```

- The `ValidateWebhookRequest` middleware performs a series of request validations:
  - Checks for required `id`, `type`, and `token` values.
  - Checks that the `id` corresponds to an object (currently only `Plan` supported).
  - Verifies the `token` and `id` match. Since the `id` is a Stripe identifier that is publicly available elsewhere (such as via the front page) the `token` is a private identifier to reduce chance of malicious requests.
  - `type` is validated (currently only `funnel` is supported).
  - Identifying data is validated. `transaction.status_at_gateway` and `transaction.gateway` are checked for success.
  - Lastly, the `transaction.id_at_gateway` value is queried from Stripe to get the associated records and ensure it is valid.
- If every check passes the request is processed by the `WebhookController` and a new `Lead` is generated containing:
  - User data retrieved from Stripe.
  - `Plan.id`
  - `metadata.source.type` is set to `stripe_charge`
  - `metadata.source.data` is set to the full Stripe Charge object content, for reference later in the pipeline.
- The `WebhookLeadGenerated` event is fired which triggers the `WebhookLeadGeneratedListener` listener.
- A new `LeadToRegistrationMail` is sent to the user's email address requesting them to click through to complete the registration process. This email has a valid confirmation token that expires after `14 days` (this can easily be adjusted).
- Clicking the button in the email redirects the user to `/register`.
- Fields are pre-filled with available data from the Stripe transaction record.
- Once the user signs up via normal registration an activate email is sent requesting them to activate their account.

## Site Statistics

Site statistics are based on end-user device requests to `/api/widget`. In addition to responding with widget payload, a background event records the request within Coeus.

Text-to-Speech statistics are generated via AWS Lambda events.

### Amazon SQS

- SRN: `srn:sqs:wcasg:widget:dashboard:testing::queue`
- URL: `https://sqs.us-west-2.amazonaws.com/696585593443/wcasg-widget-dashboard-testing-statistics`

1. An incoming Widget request is handled via the `/api/widget` endpoint.
2. Prior to payload return a new [WidgetRequested](app/Domain/Api/Widget/Events/WidgetRequested.php) event is generated.
3. A [RecordWidgetRequest](app/Domain/Api/Widget/Jobs/RecordWidgetRequest.php) job is dispatched via Amazon SQS.
4. The `php artisan queue:work sqs` queue catches incoming Amazon SQS messages.
5. The `RecordWidgetRequest` handler makes an API request to Coeus as the `wcasg-widget-dashboard-statistics` User
6. A Coeus `/data/insert` body contains widget request and site data, e.g.:

```json
{
  "db": "wcasg",
  "collection": "srn:coeus:wcasg:widget:dashboard::collection/statistics",
  "document": [
    {
      "_id": { "$oid": "5f71ccc83e6a7a5a88612c10" },
      "request": {
        "bytes": 241215,
        "fingerprint": "d885e8754d7fc7e1215a7007b5135f7365c3016",
        "headers": {
          "host": "wcasg.test",
          "connection": "keep-alive",
          "origin": "http://localhost:5000",
          "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.121 Safari/537.36",
          "accept": "*/*",
          "referer": "http://localhost:5000/",
          "accept-encoding": "gzip, deflate",
          "accept-language": "en-US,en;q=0.9"
        },
        "host": "wcasg.test",
        "ip": "127.0.0.1",
        "timestamp": 1601293510760,
        "url": "http://wcasg.test/api/widget/c48014811431eaaa5c3df81d0fe6dc78b45405d3d460bbd9d9b09778c023b3ac56f79a9676073217b317a59d564a9c9200a7a1373f3897f991093f595812b4a9?XDEBUG_SESSION_START=1"
      },
      "site": {
        "id": 1,
        "domain": "localhost:84",
        "active": true,
        "name": "Gerlach, Robel and Treutel",
        "widget_request_count": 0,
        "token": "cb8fd34d1fa3731593a503f9476127f9dce1",
        "created_at": "2020-09-26 08:12:20",
        "updated_at": "2020-09-26 08:12:20",
        "subscription_id": "sub_I5jzKUmjlmEmdo",
        "statement_id": 1
      },
      "subscription": {
        "id": "sub_I5jzKUmjlmEmdo",
        "plan_id": "plan_I5jz3eT9Krfiew",
        "user_id": 1,
        "name": "25 Sites per month",
        "stripe_status": "active",
        "stripe_id": "sub_I5jzKUmjlmEmdo",
        "stripe_plan": "plan_I5jz3eT9Krfiew",
        "quantity": 1,
        "trial_ends_at": null,
        "ends_at": null,
        "created_at": "2020-09-26 08:12:17",
        "updated_at": "2020-09-26 08:12:17"
      },
      "user": {
        "id": 1,
        "first_name": "Gabe",
        "last_name": "Wyatt",
        "username": "gabestah",
        "profile_image": "/img/avatar.png",
        "email": "gabe@solarixdigital.com",
        "phone": "606-447-4553",
        "activated": 1,
        "created_at": "2020-09-26 08:12:13",
        "updated_at": "2020-09-26 08:12:16",
        "deleted_at": null,
        "stripe_id": "cus_I5jzLyc09MiV8H",
        "card_brand": "mastercard",
        "card_last_four": "4444",
        "trial_ends_at": null,
        "company_name": "Schamberger Ltd",
        "address1": "7750 Tyrel Lakes",
        "address2": "Suite 487",
        "city": "Rolfsonside",
        "state": "Maryland",
        "country": "BTN",
        "postal_code": "60868",
        "name": "Gabe Wyatt"
      }
    }
  ]
}
```

### Coeus Users

The following Coeus Users are registered via the `/user/register` endpoint:

- **Basic WCASG Statistic User**: Allows full `data` service privileges to all `wcasg.*` resources. This User handles all Dashboard-related statistic requests.

```json
{
  "email": "dev@solarixdigital.com",
  "org": "wcasg",
  "password": "<password>",
  "policy": {
    "statement": [
      {
        "action": "data:*",
        "constraints": [
          { "type": "maxRequests", "value": 500 },
          {
            "type": "ip",
            "value": ["54.201.114.22", "127.0.0.1", "35.167.22.233"]
          }
        ],
        "resource": "wcasg.*"
      }
    ],
    "version": "1.1.0"
  },
  "username": "wcasg-widget-dashboard-statistics"
}
```

- **WCASG TTS Statistic User**: Only allowed to insert data in the statistics collection. This User is invoked by AWS Lambda to handle TTS request events.

```json
{
  "email": "dev@solarixdigital.com",
  "org": "wcasg",
  "password": "<password>",
  "policy": {
    "statement": [
      {
        "action": "data:insert",
        "constraints": [{ "type": "maxRequests", "value": 120 }],
        "resource": "wcasg.srn:coeus:wcasg:widget:dashboard::collection/statistics"
      }
    ],
    "version": "1.1.0"
  },
  "username": "wcasg-widget-dashboard-tts"
}
```

### TTS Request Statistics

End-user TTS requests via the widget are passed to an Amazon API Gateway endpoint to keep internal Google TTS Service request secure. Therefore, securely generating TTS request statistics within Coeus requires a multi-stage process:

1. Each WCASG Dashboard `/api/widget` API request now includes encrypted `WcasgWidgetTTSRequestData` with the widget payload. This data includes basic Site information and Coeus environment URL:

```ts
protected function getWidgetTTSRequestDataPayload(Site $site) {
  $encrypted = '';
  try {
    $encrypted = Version2::encrypt(
      json_encode([
        'coeus' => [
          'url' => config('solarix.coeus.url')
        ],
        'request' => [
          'timestamp' => (new DateTime())->getTimestamp(),
          'type' => 'tts'
        ],
        'site' => $site->toArraySimple()
      ]),
      new SymmetricKey(config('app.paseto.secret'))
    );
  } catch (PasetoException $e) {
    // noop
  } catch (SodiumException $e) {
    // noop
  } finally {
    return webpackify('WcasgWidgetTTSRequestData', $encrypted);
  }
}
```

2. An end-user makes a TTS request via to the loaded widget, which is handled by the API Gateway endpoint (`https://9qytmg2tn1.execute-api.us-west-2.amazonaws.com/Prod`). The widget adds a custom `X-Wcasg-Widget-Tts-Request-Data` header to the request to identify this request for further pipeline processing.
3. The API Gateway handles the TTS request via the [lambda-to-google-tts function](https://gitlab.solarixdigital.com/solarix/wcasg/lambda-to-google-tts).
4. Along with responding to the end-user with the converted speech audio stream, [a new Lambda function](https://gitlab.solarixdigital.com/solarix/wcasg/lambda-to-google-tts/-/blob/master/srn-lambda-wcasg-widget-dashboard_coeus-insert-tts-request.js) is invoked to record the TTS request via the Coeus API.
5. If the payload is valid and can be decrypted then the [srn-lambda-wcasg-widget-dashboard_coeus-insert-tts-request](https://us-west-2.console.aws.amazon.com/lambda/home?region=us-west-2#/functions/srn-lambda-wcasg-widget-dashboard_coeus-insert-tts-request?tab=configuration) Lambda function makes a Coeus `/data/insert` request and adds the TTS request record to the `wcasg.srn:coeus:wcasg:widget:dashboard::collection/statistics` resource.
6. This request includes TTS-specific data such as audio options and input text, along with originating Site data:

```json
{
  "_id": { "$oid": "5f7a54064dc3ba58d60c4116" },
  "request": {
    "body": {
      "audioConfig": {
        "audioEncoding": "LINEAR16",
        "speakingRate": 1,
        "pitch": 2,
        "sampleRateHertz": 24000,
        "volumeGainDb": 0,
        "effectsProfileId": ["handset-class-device"]
      },
      "input": {
        "text": "\n          IF THIS STORY IS to be something resembling my book of deeds, we must\n          begin at the beginning. At the heart of who I truly am. To do this,\n          you must remember that before I was anything else, I was one of the\n          Edema Ruh.\n        "
      },
      "voice": {
        "languageCode": "en-US",
        "name": "en-US-Wavenet-H",
        "ssmlGender": "FEMALE"
      }
    },
    "bytes": 747871,
    "headers": {
      "Host": "127.0.0.1:3000",
      "Connection": "keep-alive",
      "Content-Length": "527",
      "Pragma": "no-cache",
      "Cache-Control": "no-cache",
      "Sec-Ch-Ua": "\"\\\\Not;A\\\"Brand\";v=\"99\", \"Google Chrome\";v=\"85\", \"Chromium\";v=\"85\"",
      "X-Wcasg-Widget-Tts-Request-Data": "v2.local.V0zObJ9uVkQMixq1nEDP_h3h7LqYC3GXq74HrB4d56i2qSPUKQYejpcGKyp1ic6U9Do077sHqWDTh9FysJBkoLLUDN_-2W8_uKLg0HCcTrDaqcDLhUlFW1zE6Vdz2abUiG0uGDZrnsl5Mp6gRWmnvcj3aduqTK5aIw2FqL8ByGFuT3Ljnb3GpdbAQaPwGjIpNXsZXrFLsBQa-jiSCiJkrvPzH6aYRcRjyfKIbXlq5-fCjigF8HQr6ZEyUnzC6Ixly70uststpMHWF3M3JDcjTuW9KlwsnhXSUM_gToBg9ZM0YsQY4kE-fSI231XMqdx9Kje-oeJ21SaEM76u7-9iXZpYIbX9JsTWWfUtp6VOyNjGWCITgfsUcbmraqp9Zgw0iKjBO3HuSGG00oVq5W03lit-JIGnuxMMgALuZFqlUIRyIV9Q6TQ4LWqU1e_D5epcp7xsobwCfTdOEsbj3KRAbZmNsn4sikBWabCWxwKc2QH1itiP-AFZeUqo9uOWlIfs3v8VKKQM26rGw8p5RzHZmPtAW9Q0y8eqEdc",
      "Sec-Ch-Ua-Mobile": "?0",
      "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.121 Safari/537.36",
      "Content-Type": "application/json",
      "Accept": "*/*",
      "Origin": "http://localhost:5000",
      "Sec-Fetch-Site": "cross-site",
      "Sec-Fetch-Mode": "cors",
      "Sec-Fetch-Dest": "empty",
      "Referer": "http://localhost:5000/",
      "Accept-Encoding": "gzip, deflate, br",
      "Accept-Language": "en-US,en;q=0.9",
      "X-Forwarded-Proto": "http",
      "X-Forwarded-Port": "3000"
    },
    "timestamp": 1601851072,
    "type": "tts"
  },
  "site": {
    "id": 1,
    "domain": "localhost:84",
    "active": true,
    "name": "Gerlach, Robel and Treutel",
    "widget_request_count": 0,
    "token": "cb8fd34d1fa3731593a503f9476127f9dce1",
    "created_at": "2020-09-26 08:12:20",
    "updated_at": "2020-09-26 08:12:20",
    "subscription_id": "sub_I5jzKUmjlmEmdo",
    "statement_id": 1
  }
}
```

7. The WCASG Dashboard's [StatisticsService](app/Solarix/Statistics/StatisticsService.php) makes a `/data/aggregate` Coeus API call to retrieve both widget and TTS request statistics for the Site(s) in question.

### DB & Collection

- DB: `wcasg`
- Collection: `srn:coeus:wcasg:widget:dashboard::collection/statistics`

## Glossary

### Widget

The running, interactive version of the `snippet`. References to `widget` usually refer to the interactive popup that an end `user` manipulates to adjust personal accessibility.

### Client

The sole party working with Solarix and paying to have the `widget` and `dashboard` developed.

### Customer

Any non-`admin` individual using the SaaS website / `dashboard`. Will ideally be paying for service to inject `snippet` into their websites as identified by unique sites.

### User (or End-User)

An individual using the `widget` once it has been injected as a `snippet` into a `customer's` site.

### Dashboard

Backend of SaaS website that allows both `customers` and `admins` use to adjust interact with `client's` SaaS business.
