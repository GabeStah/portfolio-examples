**Note: This README is extracted from the original source project that I created. Please see [WARNING & COPYRIGHT NOTICE](../../../README.md#warning--copyright-notice) for more info.**

Coeus is an HTTP API providing insightful answers through data.

In Greek mythology [Coeus](https://en.wikipedia.org/wiki/Coeus) is the son of Uranus and Gaia and is the ["Titan of intellect and the axis of heaven around which the constellations revolved"](https://en.wikipedia.org/wiki/List_of_Greek_mythological_figures#Titans_and_Titanesses). The name is derived from the Ancient Greek [κοῖος](https://en.wiktionary.org/wiki/%CE%9A%CE%BF%E1%BF%96%CE%BF%CF%82), meaning "what?, which?, of what kind?, of what nature?".

- [1. Conventions and Terminology](#1-conventions-and-terminology)
- [2. Development](#2-development)
  - [2.1. Dynamic Watches](#21-dynamic-watches)
  - [TLS Certificate](#tls-certificate)
- [3. Testing](#3-testing)
  - [3.1. Coverage](#31-coverage)
- [4. Production](#4-production)
- [5. Benchmarks](#5-benchmarks)
  - [5.1. Status](#51-status)
  - [5.2. Unauthorized Request](#52-unauthorized-request)
  - [5.3. Basic Data Retrieval](#53-basic-data-retrieval)
  - [5.4. Full Text Search](#54-full-text-search)
    - [5.4.1. Conclusion](#541-conclusion)
- [6. Rate Limits](#6-rate-limits)
  - [6.1. Global Rate Limits](#61-global-rate-limits)
  - [6.2. Policy-Based Rate Limits](#62-policy-based-rate-limits)
- [7. Database: MongoDB](#7-database-mongodb)
  - [7.1. Naming Conventions](#71-naming-conventions)
  - [7.2. Databases](#72-databases)
    - [7.2.1. Examples](#721-examples)
  - [7.3. Collections](#73-collections)
    - [7.3.1. Examples](#731-examples)
  - [7.4. Document](#74-document)
- [8. Identifiers](#8-identifiers)
- [9. Protected Database / Collections](#9-protected-database--collections)
- [10. Users, Authentication, and Authorization](#10-users-authentication-and-authorization)
  - [10.1. User Registration](#101-user-registration)
    - [10.1.1. /user/register Request Example](#1011-userregister-request-example)
  - [10.2. User Email Verification](#102-user-email-verification)
  - [10.3. User Login](#103-user-login)
    - [10.3.1. /user/login Request Example](#1031-userlogin-request-example)
  - [10.4. Authentication](#104-authentication)
    - [10.4.1. User Hash Map Cache](#1041-user-hash-map-cache)
  - [10.5. Authorization](#105-authorization)
    - [10.5.1. Policy](#1051-policy)
      - [10.5.1.1. Policy Statement: Property Case Sensitivity](#10511-policy-statement-property-case-sensitivity)
      - [10.5.1.2. Policy Statement: action](#10512-policy-statement-action)
      - [10.5.1.3. Policy Statement: resource](#10513-policy-statement-resource)
      - [10.5.1.4. Policy Statement: allow](#10514-policy-statement-allow)
      - [10.5.1.5. Policy Statement: constraints](#10515-policy-statement-constraints)
        - [10.5.1.5.1. MaxRequestsConstraint](#105151-maxrequestsconstraint)
        - [10.5.1.5.2. IpConstraint](#105152-ipconstraint)
        - [10.5.1.5.3. HostnameConstraint](#105153-hostnameconstraint)
      - [10.5.1.6. Example Policies](#10516-example-policies)
- [11. Routes](#11-routes)
- [12. Story Implementation Examples](#12-story-implementation-examples)
  - [12.1. WCASG Dashboard: Usage Stats](#121-wcasg-dashboard-usage-stats)
  - [12.2. Acme Logistics: Fruit](#122-acme-logistics-fruit)
  - [12.3. Strapi CMS](#123-strapi-cms)
  - [12.4. Salesforce: CSV](#124-salesforce-csv)
- [13. API](#13-api)
  - [13.1. Errors](#131-errors)
    - [13.1.1. Error Response Example](#1311-error-response-example)
- [14. Requests](#14-requests)
  - [14.1. Limits](#141-limits)
  - [14.2. Timeout](#142-timeout)
  - [14.3. Query Normalization](#143-query-normalization)
    - [14.3.1. \_id and BSON ObjectIds](#1431-_id-and-bson-objectids)
  - [14.4. /data/aggregate](#144-dataaggregate)
    - [14.4.1. /data/aggregate Schema](#1441-dataaggregate-schema)
    - [14.4.2. /data/aggregate Request Examples](#1442-dataaggregate-request-examples)
    - [14.4.3. See Also](#1443-see-also)
  - [14.5. /data/createIndex](#145-datacreateindex)
    - [14.5.1. /data/createIndex Schema](#1451-datacreateindex-schema)
    - [14.5.2. /data/createIndex Request Examples](#1452-datacreateindex-request-examples)
  - [14.6. /data/delete](#146-datadelete)
    - [14.6.1. /data/delete Schema](#1461-datadelete-schema)
    - [14.6.2. /data/delete Request Example](#1462-datadelete-request-example)
  - [14.7. /data/dropIndex](#147-datadropindex)
    - [14.7.1. /data/dropIndex Schema](#1471-datadropindex-schema)
    - [14.7.2. /data/dropIndex Request Examples](#1472-datadropindex-request-examples)
  - [14.8. /data/find](#148-datafind)
    - [14.8.1. /data/find Schema](#1481-datafind-schema)
    - [14.8.2. /data/find Request Example](#1482-datafind-request-example)
  - [14.9. /data/insert](#149-datainsert)
    - [14.9.1. /data/insert Schema](#1491-datainsert-schema)
    - [14.9.2. /data/insert Request Example](#1492-datainsert-request-example)
  - [14.10. /data/listIndexes](#1410-datalistindexes)
    - [14.10.1. /data/listIndexes Schema](#14101-datalistindexes-schema)
    - [14.10.2. /data/listIndexes Request Examples](#14102-datalistindexes-request-examples)
  - [14.11. /data/update](#1411-dataupdate)
    - [14.11.1. /data/update Schema](#14111-dataupdate-schema)
    - [14.11.2. /data/update Request Example](#14112-dataupdate-request-example)
- [15. TODO](#15-todo)
  - [15.1. Compression](#151-compression)
  - [15.2. In-Memory Cache of User Data](#152-in-memory-cache-of-user-data)
  - [15.3. /data/delete Logic Check: `_id` Property](#153-datadelete-logic-check-_id-property)
  - [15.4. Pagination](#154-pagination)
  - [15.5. Logging](#155-logging)
  - [15.6. Caching](#156-caching)
  - [15.7. CORS Support](#157-cors-support)
  - [15.8. /user/register Option: email](#158-userregister-option-email)
  - [15.9. /user/login Option: email](#159-userlogin-option-email)
  - [15.10. /user/activate Endpoint](#1510-useractivate-endpoint)
  - [15.11. Request Option: idempotence_id](#1511-request-option-idempotence_id)
  - [15.12. Request Option: format](#1512-request-option-format)
  - [15.13. Request Option: email](#1513-request-option-email)
  - [15.14. Benchmarking](#1514-benchmarking)
  - [15.15. Rate Limiting](#1515-rate-limiting)
  - [15.16. PolicyStatement Property: rateLimit](#1516-policystatement-property-ratelimit)
  - [15.17. PolicyStatement Property: ip](#1517-policystatement-property-ip)
  - [15.18. PolicyStatement Property: hostname](#1518-policystatement-property-hostname)
  - [15.19. /user/explain Endpoint](#1519-userexplain-endpoint)
  - [15.20. API Documentation Generator](#1520-api-documentation-generator)
  - [15.21. Commit Release Update](#1521-commit-release-update)
  - [15.22. Two-Factor Authentication (TOTP)](#1522-two-factor-authentication-totp)

# 1. Conventions and Terminology

The key words _MUST_, _MUST NOT_, _REQUIRED_, _SHALL_, _SHALL NOT_, _SHOULD_, _SHOULD NOT_, _RECOMMENDED_, _MAY_, and _OPTIONAL_ in this document are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119).

# 2. Development

1. Install Node modules: `yarn install`
2. Copy `config/example.json` to `environment.json`, replacing `environment` with the appropriate environment name (e.g. `development.json`). See [snippets/development.json](https://gitlab.solarixdigital.com/solarix/core/soldata/coeus/snippets/20) for functional example.
3. Update the configuration file with appropriate values.
4. Make code changes under `src/`
5. Build dist package from TypeScript with `yarn run build`
6. Launch app with `yarn run start`
7. Perform status check to ensure server is active:

```bash
$ curl -X POST "localhost:8000/status"
{"active":true,"date":"2020-08-28T22:14:56.294Z"}
```

## 2.1. Dynamic Watches

1. Execute `yarn run watch` to monitor and rebuild on source changes.
2. In a second console, execute `yarn run watch:server` to monitor build changes and restart the server.

## TLS Certificate

1. SSH to `coeus.solarix.dev`
2. Renew via AWS Route53 DNS:

```bash
acme.sh --issue --dns dns_aws -d coeus.solarix.dev
```

# 3. Testing

1. Create `*.test.ts` files under `src/`
2. Execute `yarn run test` to perform a one-off test
3. Alternatively, execute `yarn run test:watch` to watch and re-run tests on code change

**NOTE:** The `test:debug` command variants _SHOULD_ be executed while halting execution within a test (such as during debugging).

## 3.1. Coverage

`yarn run test:coverage` generates a full test coverage report using [IstanbulJS](https://github.com/istanbuljs/istanbuljs). The generated HTML report can be found in the `coverage/lcov-report`, e.g.:

| Statements        |       | Branches |       | Functions |       | Lines |       |
| ----------------- | ----- | -------- | ----- | --------- | ----- | ----- | ----- |
| src               |       |          |       |           |       |       |       |
| 100%              | 12/12 | 100%     | 0/0   | 100%      | 1/1   | 100%  | 12/12 |
| src/config        |       |          |       |           |       |       |       |
| 100%              | 8/8   | 100%     | 0/0   | 100%      | 0/0   | 100%  | 8/8   |
| src/helpers       |       |          |       |           |       |       |       |
| 100%              | 29/29 | 100%     | 5/5   | 100%      | 16/16 | 100%  | 29/29 |
| src/models        |       |          |       |           |       |       |       |
| 100%              | 66/66 | 100%     | 24/24 | 100%      | 18/18 | 100%  | 66/66 |
| src/plugins       |       |          |       |           |       |       |       |
| 100%              | 28/28 | 100%     | 2/2   | 100%      | 5/5   | 100%  | 26/26 |
| src/plugins/db    |       |          |       |           |       |       |       |
| 100%              | 15/15 | 100%     | 0/0   | 100%      | 6/6   | 100%  | 12/12 |
| src/plugins/hooks |       |          |       |           |       |       |       |
| 100%              | 7/7   | 100%     | 0/0   | 100%      | 4/4   | 100%  | 5/5   |
| src/routes        |       |          |       |           |       |       |       |
| 100%              | 7/7   | 100%     | 0/0   | 100%      | 4/4   | 100%  | 6/6   |
| src/routes/data   |       |          |       |           |       |       |       |
| 100%              | 31/31 | 100%     | 0/0   | 100%      | 12/12 | 100%  | 28/28 |
| src/routes/user   |       |          |       |           |       |       |       |
| 100%              | 19/19 | 100%     | 0/0   | 100%      | 8/8   | 100%  | 17/17 |
| src/schema        |       |          |       |           |       |       |       |
| 100%              | 1/1   | 100%     | 0/0   | 100%      | 0/0   | 100%  | 1/1   |
| src/services      |       |          |       |           |       |       |       |
| 100%              | 70/70 | 100%     | 35/35 | 100%      | 14/14 | 100%  | 70/70 |

# 4. Production

Launch production app via:

```bash
node dist/server.js | pino-cloudwatch --group coeus --stream coeus/base/production/log --aws_region us-west-2 --aws_access_key_id <key> --aws_secret_access_key <key>
```

# 5. Benchmarks

## 5.1. Status

`$ autocannon -c 100 -d 20 -p 10 -m POST localhost:8000/status`

- Purpose: Determine maximum app response rate
- Result: `19500 req/sec` average, `14500 req/sec` minimum, `5 ms` average latency

```
┌─────────┬──────┬──────┬───────┬───────┬─────────┬──────────┬────────┐
│ Stat    │ 2.5% │ 50%  │ 97.5% │ 99%   │ Avg     │ Stdev    │ Max    │
├─────────┼──────┼──────┼───────┼───────┼─────────┼──────────┼────────┤
│ Latency │ 0 ms │ 0 ms │ 53 ms │ 61 ms │ 5.05 ms │ 15.63 ms │ 232 ms │
└─────────┴──────┴──────┴───────┴───────┴─────────┴──────────┴────────┘
┌───────────┬─────────┬─────────┬─────────┬─────────┬─────────┬─────────┬─────────┐
│ Stat      │ 1%      │ 2.5%    │ 50%     │ 97.5%   │ Avg     │ Stdev   │ Min     │
├───────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤
│ Req/Sec   │ 14447   │ 14447   │ 19535   │ 21727   │ 19482   │ 1968.22 │ 14443   │
├───────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤
│ Bytes/Sec │ 2.83 MB │ 2.83 MB │ 3.83 MB │ 4.26 MB │ 3.82 MB │ 386 kB  │ 2.83 MB │
└───────────┴─────────┴─────────┴─────────┴─────────┴─────────┴─────────┴─────────┘
```

## 5.2. Unauthorized Request

`$ autocannon -c 100 -d 20 -p 10 -m POST localhost:8000/data/find`

- Purpose: Test speed for processed, unauthorized requests (i.e. invalid JWT)
- Result: `8500 req/sec` average, `7500 req/sec` minimum, `11.5 ms` average latency

```
┌─────────┬──────┬──────┬────────┬────────┬──────────┬──────────┬────────┐
│ Stat    │ 2.5% │ 50%  │ 97.5%  │ 99%    │ Avg      │ Stdev    │ Max    │
├─────────┼──────┼──────┼────────┼────────┼──────────┼──────────┼────────┤
│ Latency │ 0 ms │ 0 ms │ 119 ms │ 134 ms │ 11.65 ms │ 35.15 ms │ 252 ms │
└─────────┴──────┴──────┴────────┴────────┴──────────┴──────────┴────────┘
┌───────────┬─────────┬─────────┬─────────┬─────────┬─────────┬────────┬─────────┐
│ Stat      │ 1%      │ 2.5%    │ 50%     │ 97.5%   │ Avg     │ Stdev  │ Min     │
├───────────┼─────────┼─────────┼─────────┼─────────┼─────────┼────────┼─────────┤
│ Req/Sec   │ 7531    │ 7531    │ 8703    │ 9047    │ 8507.5  │ 481.77 │ 7530    │
├───────────┼─────────┼─────────┼─────────┼─────────┼─────────┼────────┼─────────┤
│ Bytes/Sec │ 1.93 MB │ 1.93 MB │ 2.23 MB │ 2.32 MB │ 2.18 MB │ 123 kB │ 1.93 MB │
└───────────┴─────────┴─────────┴─────────┴─────────┴─────────┴────────┴─────────┘
```

## 5.3. Basic Data Retrieval

```bash
$ autocannon -c 100 -d 20 -p 10 -m POST -H 'Authorization: Bearer <JWT>' -H 'Content-Type: application/json' -i benchmark/data/find/movie-basic.json localhost:8000/data/find
```

- Purpose: Test full authorization, database lookup, and retrieval
- Result: `760 req/sec` average, `600 req/sec` minimum, `125 ms` average latency

```
┌─────────┬──────┬──────┬─────────┬─────────┬───────────┬───────────┬─────────┐
│ Stat    │ 2.5% │ 50%  │ 97.5%   │ 99%     │ Avg       │ Stdev     │ Max     │
├─────────┼──────┼──────┼─────────┼─────────┼───────────┼───────────┼─────────┤
│ Latency │ 0 ms │ 1 ms │ 1284 ms │ 1305 ms │ 126.92 ms │ 378.76 ms │ 1538 ms │
└─────────┴──────┴──────┴─────────┴─────────┴───────────┴───────────┴─────────┘
┌───────────┬────────┬────────┬────────┬────────┬────────┬─────────┬────────┐
│ Stat      │ 1%     │ 2.5%   │ 50%    │ 97.5%  │ Avg    │ Stdev   │ Min    │
├───────────┼────────┼────────┼────────┼────────┼────────┼─────────┼────────┤
│ Req/Sec   │ 591    │ 591    │ 778    │ 788    │ 761.4  │ 45.07   │ 591    │
├───────────┼────────┼────────┼────────┼────────┼────────┼─────────┼────────┤
│ Bytes/Sec │ 596 kB │ 596 kB │ 785 kB │ 795 kB │ 768 kB │ 45.5 kB │ 596 kB │
└───────────┴────────┴────────┴────────┴────────┴────────┴─────────┴────────┘
```

## 5.4. Full Text Search

```bash
$ autocannon -c 100 -d 20 -p 10 -m POST -H 'Authorization=Bearer <JWT>' -H 'Content-Type: application/json' -i benchmark/data/find/movie-full-text-search.json localhost:8000/data/find
```

- Purpose: Test full text search with reasonable payload size (`8.5MB`)v
- Result: `660 req/sec` average, `530 req/sec` minimum, `145 ms` average latency

```
┌─────────┬──────┬──────┬─────────┬─────────┬───────────┬───────────┬─────────┐
│ Stat    │ 2.5% │ 50%  │ 97.5%   │ 99%     │ Avg       │ Stdev     │ Max     │
├─────────┼──────┼──────┼─────────┼─────────┼───────────┼───────────┼─────────┤
│ Latency │ 0 ms │ 1 ms │ 1484 ms │ 1497 ms │ 145.07 ms │ 434.79 ms │ 1690 ms │
└─────────┴──────┴──────┴─────────┴─────────┴───────────┴───────────┴─────────┘
┌───────────┬─────────┬─────────┬─────────┬─────────┬─────────┬────────┬─────────┐
│ Stat      │ 1%      │ 2.5%    │ 50%     │ 97.5%   │ Avg     │ Stdev  │ Min     │
├───────────┼─────────┼─────────┼─────────┼─────────┼─────────┼────────┼─────────┤
│ Req/Sec   │ 531     │ 531     │ 670     │ 681     │ 662.9   │ 30.89  │ 531     │
├───────────┼─────────┼─────────┼─────────┼─────────┼─────────┼────────┼─────────┤
│ Bytes/Sec │ 6.97 MB │ 6.97 MB │ 8.79 MB │ 8.94 MB │ 8.69 MB │ 405 kB │ 6.96 MB │
└───────────┴─────────┴─────────┴─────────┴─────────┴─────────┴────────┴─────────┘
```

### 5.4.1. Conclusion

Caveat: These tests are based on my own local machine.

If results are similar in production, a target maximum of `500 req/sec` across the app should be sufficient and maintainable.

# 6. Rate Limits

Rate limiting is based on a [forked version](https://github.com/GabeStah/fastify-rate-limit/tree/add-fastify-hook-method-setting) of `fastify-rate-limit`. The fork is necessary to allow rate limiting to be executed at any point in the Fastify lifecycle.

## 6.1. Global Rate Limits

Global rate limiting is defined in the `rateLimit` configuration options and behaves as expected:

- Each request is tracked in memory using a unique identifer (`User.id`, if applicable token payload exists, otherwise IP address is used).
- The `rateLimit.timeWindow` defines the period of time in which the `rateLimit.maxRequests` number of requests are accepted for each unique key.
- By default, a given request source can perform a maximum of `60 requests per second`.
- If a limit is exceeded the response is a `429` error.

## 6.2. Policy-Based Rate Limits

Each User Policy may define a `maxRequests` **Constraint**. See [Policy Statement: Constraints](#9515-policy-statement-constraints) for details.

# 7. Database: MongoDB

- Version: 4.4
- Region: AWS us-east-1 (required for version 4.4 free tier support)

## 7.1. Naming Conventions

**NOTE:** Windows users may experience trouble within the `mongo` shell when using special characters (`/\. "$*<>:|?`). Please use a Linux-based shell when executing shell commands that use such characters in relevant `namespaces`.

- **Database** names _MUST_ be a minimum of `4` characters in length.
- **Database** names _MUST NOT_ exceed a maximum of `64` characters in length.
- **Database** names _MAY NOT_ contain any of the following characters: `/\. "$*<>:|?`.
- **Database** names _MAY NOT_ begin with the string `coeus`.
- A `namespace` (the combined **database.collection** name) _MAY NOT_ exceed `255` characters in length.
- **Collection** names _MAY_ use special characters, so long as they begin with a letter.
- **Collection** names _MUST_ be a minimum of `4` characters in length.
- **Collection** names _MUST NOT_ exceed a maximum of `190` characters in length.
- **Role** names _MUST_ contain only letters, numbers, hyphens, and underscores.

## 7.2. Databases

Each **database** _MUST_:

- be globally unique and identifiable, based on a single high-level entity such as an `organization`.

### 7.2.1. Examples

- `acme`: A **database name** for the Acme `org`
- `solarix`: For Solarix projects

## 7.3. Collections

Each **collection** _MUST_:

- be self-contained.
- contain related data.
- use [Solarix Resource Name (SRN)](https://docs.solarix.tools/solarix-resource-names) conventions.

Each **collection** _SHOULD_:

- be based on a single high-level entity such as an `org`, if applicable.

A higher-order **SRN** is preferred for aggregate queries and data storage, but smaller **collections** can be created to store separate `project` or even `app` data.

### 7.3.1. Examples

An **SRN** of `srn::acme` is the highest order **SRN** for the `Acme` `org`, applying to all services and all projects within that `org`. A **collection `name`** of `srn:coeus:acme::collection` may contain any type of Acme-related data.

Alternatively, an **SRN** of `srn::acme:tracker:api:production` with related **collection `name`** of `srn:coeus:acme:tracker:api:production::collection` is expected to contain data only related to Acme's Tracker API project, in the production environment of the AWS EC2 service. This is a much smaller scope, so take care when choosing which **collection** names to create.

## 7.4. Document

Each **document** _MUST_:

- be less than `16MB`.

# 8. Identifiers

- database
- collection
- document
- srn
- service
- org
- project
- app
- environment
- resource-type
- resource-id

# 9. Protected Database / Collections

The `coeus` database is protected and used for administration purposes.

- `coeus.users` - Stores all User documents

# 10. Users, Authentication, and Authorization

Coeus authentication and authorization is performed based on the requesting User's `coeus.users` document. The schema of a `user` document can be found in the [src/routes/user/register.ts](src/routes/user/register.ts#L21) file. Below is an example `user` document:

```json
{
  "_id": "12345",
  "srn": "srn:coeus:acme::user/johndoe",
  "email": "john@acme.com",
  "org": "acme",
  "username": "johndoe",
  "password": "password",
  "verified": true,
  "active": true,
  "policy": {
    "version": "1.1.0",
    "statement": [
      {
        "action": "data:find",
        "resource": "acme.*"
      },
      {
        "action": ["data:insert", "data:update"],
        "allow": true,
        "resource": "acme.srn:coeus:acme::collection"
      },
      {
        "action": ["data:delete"],
        "allow": false,
        "resource": "acme.*"
      }
    ]
  }
}
```

## 10.1. User Registration

A new User is registered by making an appropriate request to the `/user/register` endpoint. The current schema can be found in the [src/routes/user/register.ts](src/routes/user/register.ts#L21) file.

Once a User is registered and set `active` by an admin, that user can then login to retrieve an authentication token.

### 10.1.1. /user/register Request Example

Each **User** _MUST_ contain:

- `username`: To allow for multiple users per email, `username` is the **primary unique value** for differentiating users.
- `email`
- `org`: Organization name, per the [Solarix Resource Name (SRN)](https://docs.solarix.tools/solarix-resource-names/) conventions.
- `password`

Each **User** _SHOULD_ contain:

- `policy`: An object containing `PolicyStatements` defining permissions. See [Policy](#policy) for details.

For example, a `POST` request to `/user/register` can be made with a body payload of:

```json
{
  "email": "john@acme.com",
  "org": "acme",
  "username": "johnsmith",
  "password": "password",
  "policy": {
    "version": "1.1.0",
    "statement": [
      {
        "action": "data:find",
        "resource": "acme.*"
      },
      {
        "action": ["data:insert", "data:update"],
        "allow": true,
        "resource": "acme.srn:coeus:acme::collection"
      },
      {
        "action": ["data:delete"],
        "allow": false,
        "resource": "acme.*"
      }
    ]
  }
}
```

This successfully registers the above user and responds with the created message and data object:

```json
{
  "message": "User successfully created.",
  "data": {
    "email": "john@acme.com",
    "org": "acme",
    "srn": "srn:coeus:acme::user/johnsamith",
    "username": "johnsamith",
    "policy": {
      "version": "1.1.0",
      "statement": [
        {
          "action": "data:find",
          "resource": "acme.*"
        },
        {
          "action": ["data:insert", "data:update"],
          "allow": true,
          "resource": "acme.srn:coeus:acme::collection"
        },
        {
          "action": ["data:delete"],
          "allow": false,
          "resource": "acme.*"
        }
      ]
    }
  }
}
```

## 10.2. User Email Verification

A `verificationToken` sub-document is attached to the User document upon registration. The User is emailed a verification link upon registration. Clicking this route verifies the User's email address, setting the `verified` property to `true` and removing the `verificationToken` sub-document.

The `verificationToken` is a 60-character string and expires after 24 hours.

## 10.3. User Login

After registering a User may send a request to the `/user/login` endpoint to authenticate and retrieve their unique JSON Web Token (JWT).

### 10.3.1. /user/login Request Example

A `/user/login` request payload _MUST_ contain:

- `username`
- `password`

A `/user/login` request payload _MAY_ contain:

- `email`: Boolean indicating if JWT should be emailed to user upon successful login.

For example, a `POST` request to `/user/login` can be made with a body payload of:

```json
{
  "username": "johnsmith",
  "password": "password"
}
```

If the username and password are correct and the User is `active` and `verified` then the response payload will output the User's JWT:

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY3RpdmUiOmZhbHNlLCJlbWFpbCI6ImpvaG5AYWNtZS5jb20iLCJvcmciOiJhY21lIiwicHJpdmlsZWdlcyI6W3sicmVzb3VyY2UiOnsiZGIiOiJhY21lIiwiY29sbGVjdGlvbiI6InNybjpjb2V1czphY21lOjpjb2xsZWN0aW9uIn0sImFjdGlvbnMiOiJmaW5kIn0seyJyZXNvdXJjZSI6eyJkYiI6InNvbGFyaXgiLCJjb2xsZWN0aW9uIjoic3JuOmNvZXVzOnNvbGFyaXg6OmNvbGxlY3Rpb24ifSwiYWN0aW9ucyI6ImZpbmQifV0sInNybiI6InNybjpjb2V1czphY21lOjp1c2VyL2pvaG5zbWl0aCIsInVzZXJuYW1lIjoiam9obnNtaXRoIiwiaWF0IjoxNTk5MDk0ODk5LCJpc3MiOiJjb2V1cy5zb2xhcml4LnRvb2xzIn0.73dnMmj1g2_gVS5rrlIcUT2MZgp7JjWZo9vbQyDas2c"
}
```

The generated `token` is issued by `coeus.solarix.tools` and contains encoded user data from the time of authentication, e.g.:

```json
{
  "active": true,
  "email": "john@acme.com",
  "org": "acme",
  "policy": {
    "version": "1.1.0",
    "statement": [
      {
        "action": "data:find",
        "resource": "acme.*"
      },
      {
        "action": ["data:insert", "data:update"],
        "allow": true,
        "resource": "acme.srn:coeus:acme::collection"
      },
      {
        "action": ["data:delete"],
        "allow": false,
        "resource": "acme.*"
      }
    ]
  },
  "srn": "srn:coeus:acme::user/johnsmith",
  "username": "johnsmith",
  "iat": 1599095260,
  "iss": "coeus.solarix.tools"
}
```

## 10.4. Authentication

Authentication is performed for all **protected** endpoints. Such endpoints perform a JSON Web Token `preValidation` phase before processing the request payload. For example, all `/data/*` endpoints are protected.

To authenticate a request to a protected endpoint the `Authorization` header must contain a `Bearer <jwt>` value. For example, making a request to `/data/find`:

```bash
$ curl --location --request POST 'http://localhost:8000/data/find' \
--header 'Authorization: Bearer <JWT>' \
--header 'Content-Type: application/json' \
--data-raw '{
  "collection": "srn:coeus:acme::collection",
  "db": "acme",
  "query": {
    "foo": "bar"
  }
}'
```

### 10.4.1. User Hash Map Cache

The content of the JWT payload is trustworthy and authenticates the defined User along with their appropriate Policy permissions.

**Problem**: Business requirements strongly discourage reliance on JWT expiration dates, since this would require third-party services relying on Coeus to re-authenticate and setup a new JWT `Authorization` token after every JWT expiration. Beyond that, when an Admin needs to disable a User or rotate a given JWT the app needs a method for determining the validity of an existing JWT. Making a database call on every request to verify the incoming request against the `coeus.users` collection is infeasible and costly.

**Solution**: Coeus maintains an in-memory hashmap of `coeus.users` data:

```json
{
  "5f5f35bceb2bfe1b0c5fab57": "2176dc31c298ae2a1b88409158402596d2fda788",
  "5f5f3611a897730f002fbf0f": "5f39ac60b004857da58afbd29e2ec6f8d38a65c2",
  "5f600bc48496ae4e6091f648": "0ad2768804d4d6c96d8a14b75bc12839693e28a9",
  "5f60160c42b3c361103fab6e": "1c6ce905339e03c91e15b561fdcaff23e7fbe3dd",
  "5f601764ecdb2d584868scce": "b334a2cd2f35462561ea380c9a1eed694d50606a"
}
```

Each `hash` value contains the hashed value of the matching User's relevant data, e.g.:

```js
{
  active: this.active,
  email: this.email,
  id: this.id,
  org: this.org,
  password: this.password,
  policy: this.policy.toObject(),
  username: this.username,
  srn: this.srn,
  verified: this.verified
}
```

During JWT verification within a **protected** endpoint the payload's `hash` is compared to the in-memory hashmap value. A match indicates that the passed JWT is up-to-date and can be trusted, while a mismatch indicates that the JWT is out of date and should be denied.

The local cached User hashmap is updated whenever Coeus is initialized, or anytime User db data is generated or updated. This allows Coeus to maintain real-time JWT validation without making unnecessary database calls on every request.

## 10.5. Authorization

The passed JWT is decoded and evaluated to determine the privileges assigned to the User based on the User's **Policy** object.

In general, authorization is based on a combination of **four** primary properties:

- `service`: The service that is handling the request. For example, a request to a `/data/*` endpoint uses the [DataService](src/services/DataService.ts).
- `method`: The service method that is handling the request. For example, a request to the `/data/find` endpoint is processed by the [routes/data/find](src/routes/data/find.ts) plugin, which passes the validated request to the [DataService.find()](src/services/DataService.ts#L94) method for authorization.
- `db`: The accessed database of the request.
- `collection`: The accessed collection of the request.

Coeus compares the incoming request against the **Policy** permissions granted to the User to determine if the request is authorized.

### 10.5.1. Policy

The `policy` property of a **User** document defines permissions for that user. A policy consists of one or more `statement` objects.

A **policy** _MUST_ contain:

- a `statement` property with an array of **policy statement** objects defining a related collection of privileges.

A **policy** _MAY_ contain:

- a `version` semver value that indicates what API version this policy was generated with. This value is automatically generated upon creation.

A **policy statement** _MUST_ contain:

- an `action` property.
- a `resource` property.

A **policy statement** _MAY_ contain:

- an `allow` property.

#### 10.5.1.1. Policy Statement: Property Case Sensitivity

All `PolicyStatement` property strings are **case insensitive.** Coeus normalizes all string casings during execution, so consistency and convention dictates that all property values remain lowercase.

#### 10.5.1.2. Policy Statement: action

The `action` property defines the action(s) allowed or denied by the **statement**. An `action` string _MUST_ be formatted as `<service>:<method>`. For example, an `action` of `data:find` indicates the `find` method for the `data` service.

An asterisk (`*`) may be substituted for the `<method>` as a wildcard indicator for **ALL** methods within the given `<service>`. For example, `data:*` applies actions to all methods of the `data` service.

#### 10.5.1.3. Policy Statement: resource

The `resource` property defines the resource(s) allowed or denied by the **statement**. A `resource` string _MUST_ be formatted as `<db>.<collection>`. For example, a `resource` of `acme.srn:coeus:acme::collection` indicates the `srn:coeus:acme::collection` collection of the `acme` database.

An asterisk (`*`) may be substituted for the `<collection>` as a wildcard indicator for **ALL** collections within the given `<db>`. For example, `acme:*` applies to all collections within the `acme` database.

An asterisk (`*`) may also be substituted for the entire `resource` string as a wildcard indicator for **ALL** database and collection combinations. **This provides full admin access, so use with caution.**

#### 10.5.1.4. Policy Statement: allow

The `allow` property determines if the **statement** is allowing or denying permission indicated by the related `action` and `resource`.

By default, a **statement** without an `allow` property is assumed to be `true`, allowing permission to the related `resource`. Otherwise, the default policy across the app is to deny permission unless explicitly allowed.

#### 10.5.1.5. Policy Statement: constraints

The `constraints` array defines an optional set of rules that the matching request **statement** must pass to allow the request.

An example **PolicyStatement** with **Constraints** defined:

```json
{
  "action": ["data:insert", "data:update"],
  "allow": true,
  "resource": "acme.srn:coeus:acme::collection",
  "constraints": [
    {
      "type": "maxRequests",
      "value": 60
    },
    {
      "type": "ip",
      "value": ["127.0.0.1"]
    },
    {
      "type": "hostname",
      "value": "localhost"
    }
  ]
}
```

##### 10.5.1.5.1. MaxRequestsConstraint

A **MaxRequestsConstraint** determines the maximum number of requests matching the **statement** that can be made within the `rateLimit.timeWindow` period (default: 60 seconds). This value _overrides_ the global value and is specific to the matching User and PolicyStatement.

##### 10.5.1.5.2. IpConstraint

A **IpConstraint** determines which requesting IP address(es) are allowed for requests matching the **statement**. The `value` _MUST_ be a string or array of strings, each containing a valid IP address.

If the matching **statement** contains an **IpConstraint**, the requesting IP address _MUST_ match one of the values or the request is denied.

##### 10.5.1.5.3. HostnameConstraint

A **HostnameConstraint** determines which requesting hostname(s) are allowed for requests matching the **statement**. The `value` _MUST_ be a string or array of strings, each containing a valid hostname _without_ a port (e.g. `localhost` or `example.com` are valid, `localhost:8000` or `example.com:80` are invalid).

If the matching **statement** contains an **HostnameConstraint**, the requesting hostname _MUST_ match one of the values or the request is denied.

#### 10.5.1.6. Example Policies

The following policy is intended for an Acme `org` User with moderate permissions. The policy:

- allows `data:find` access across the `acme` database
- allows `data:insert` and `data:update` access to the `srn:coeus:acme::collection` collection in the `acme` database.
- denies `data:delete` access across the `acme` database

```json
{
  "version": "1.1.0",
  "statement": [
    {
      "action": "data:find",
      "resource": "acme.*"
    },
    {
      "action": ["data:insert", "data:update"],
      "allow": true,
      "resource": "acme.srn:coeus:acme::collection"
    },
    {
      "action": ["data:delete"],
      "allow": false,
      "resource": "acme.*"
    }
  ]
}
```

The following policy is intended for a Solarix `org` User with full administrative privileges. The policy:

- allows access to all `admin` service methods across **ALL** resources
- allows access to all `data` service methods across **ALL** resources
- allows access to all `user` service methods across **ALL** resources

```json
{
  "version": "1.1.0",
  "statement": [
    {
      "action": "admin:*",
      "resource": "*"
    },
    {
      "action": "data:*",
      "resource": "*"
    },
    {
      "action": "user:*",
      "resource": "*"
    }
  ]
}
```

# 11. Routes

- /admin/authenticate - Authenticate as a User without password and receive JWT
- /admin/register - Register a User, automatically verify and activate, and receive JWT
- /data/delete - Delete documents
- /data/find - Find documents
- /data/insert - Insert documents
- /data/update - Update documents
- /user/login - Login via `username` / `password` and receive JWT
- /user/register - Register a User

# 12. Story Implementation Examples

## 12.1. WCASG Dashboard: Usage Stats

> WCASG is to start collection some very basic user stats about their widget in WCASG Dashboard issue #75 and @gabestah must store that data. The database is already built, underwent customization stresses during development and may not perfectly fit the widget/dashboard model as if it was built from scratch. However, it must still report data to SolData on how many times a widget is loaded by a web browser. Perhaps @gabestah reads the future documentation about SolData API, integrates the database storage per the specific project best fit solutions and uses a provided snippet of php to run a cronjob to POST to webhook that will be processed & stored by SolData in a timely manner. johndoe is then able to visualize the pageview & bandwidth data at the end of the month to include on the customer invoices.

- **database**: `wcasg`
- **collection**: `srn:coeus:wcasg::collection`

> Number of current Active Sites
> Page Views Per Current Month (For all active sites)
> Overall Page Views (For all active sites)
> Overall Bandwidth Used Per Month
> Average Bandwidth Per Sites (Bandwidth total / # of active sites) per month
> Voice Bandwidth Used Per Month (For all active sites)

## 12.2. Acme Logistics: Fruit

> bettyDoe at Acme Logistics asks johndoe to create a small app for her logistics company that transports fruit from regional farms. The client will require a database that stores information such as fruits, client profiles, farm asset profiles, current deliveries, sales amounts and a wide variety of other points. He refers to internal developer documentation and the basic API it offers so that the company's data can be easily submitted and eventually visualized in a report. He then writes a small script that submits some of that data to the SolData web service daily. johndoe then creates a visualization to automate weekly and sends the client an email with only the week's sales numbers and number of deliveries.

## 12.3. Strapi CMS

> charlieDoe wants Solarix to make a website for his pallet processing & storage business. His request is for a simple brochure website with an single dynamic page that displays the status of each pallet in his warehouse so that his customers know the current progress on their job. johndoe reads up on documentation requiremeents and there are a few starter databases in template repos to fork from, so he chooses to include the MongoDB starter with Strapi headless client. He writes his project, including the brochure/business content in the website attached to the CMS and giving the the customer access to the Strapi dashboard to update pallet info as needed. The MongoDB fork was already modeled to be structured in the preferred way SolData dictates and the Strapi repo included a snippet to add a POST to SolData web service whenever Strapi data is saved.

## 12.4. Salesforce: CSV

> deniseDoe has a large customer database from Salesforce exported to csv and she want's it accessible via a JSON rest API in the future. johndoe visits an internal Solarix tool that triggers a process that: imports the CSV > SolData creates an EAV model table to store that data and saves it > SolData then allows that data to be accessed via REST API to those who have access to that asset.

# 13. API

## 13.1. Errors

| Code | Type         | Message                                               | Cause                                                                                                          |
| ---- | ------------ | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| 403  | Forbidden    | Policy is invalid: No valid policy statement provided | User has no `PolicyStatement` objects within their `Policy`                                                    |
| 403  | Forbidden    | You do not have permission to perform the request     | User does not have authorization for the requested `service`, `method`, `db`, or `collection`                  |
| 403  | Forbidden    | You do not have permission to perform the request     | User has a `PolicyStatement` matching the request that is explicitly marked as denied by `allow = false`       |
| 403  | Forbidden    | Authorization token is invalid: User is inactive      | User is marked as inactive via `active = false`                                                                |
| 409  | Conflict     | Unable to create new user: <username>                 | Attempt to register a new User cannot continue, typically due to a matching `username` already in the database |
| 401  | Unauthorized | Unable to authenticate with provided credentials.     | Attempt to login failed, either because of invalid `username` or `username/password` combination               |

### 13.1.1. Error Response Example

```json
{
  "code": 403,
  "type": "Forbidden",
  "message": "You do not have permission to perform the request"
}
```

# 14. Requests

## 14.1. Limits

By default, Coeus limits the number of documents returned by a single request:

- `20` - Default maximum number of documents retrieved if no `limit` specified. Configurable via the `config.db.thresholds.limit.base` path.
- `1` - Minimum number of documents that can be retrieved. Configurable via the `config.db.thresholds.limit.minimum` path.
- `100` - Maximum number of documents that can be retrieved. Configurable via the `config.db.thresholds.limit.maximum` path.

## 14.2. Timeout

By default, Coeus restricts all requests to under `5000` milliseconds. This value is configurable via the `config.db.thresholds.timeout.maximum` path.

## 14.3. Query Normalization

MongoDB uses a JSON-like format called [BSON](https://www.mongodb.com/json-and-bson) which provides numerous advantages. However, certain value types require pre-processing before they can be used in a query.

### 14.3.1. \_id and BSON ObjectIds

When writing a Coeus API query or filter that uses a MongoDB `_id` you _MUST_ use one of two formats for related fields/values:

- **`_id` key with string value**: Any query/filter key of `_id` with a string value is automatically converted to an BSON `ObjectID` instance:

```json
{
  "collection": "users",
  "db": "sample_mflix",
  "query": {
    "_id": "59b99dbacfa9a34dcd7885c2"
  },
  "limit": 5
}
```

- **`ObjectID('value')` string value**: More complex queries that require the use of an ObjectID value _MUST_ surround the intended string value with `ObjectID('')`. Coeus will automatically parse such values and convert them into BSON `ObjectID` instances:

```json
{
  "collection": "users",
  "db": "sample_mflix",
  "query": {
    "_id": {
      "$lt": "ObjectID('59b99dbacfa9a34dcd7885c2')"
    }
  },
  "limit": 5
}
```

## 14.4. /data/aggregate

The `/data/aggregate` endpoint allows for complex, multi-stage data operations. Each [stage](https://docs.mongodb.com/manual/reference/operator/aggregation-pipeline/) passes its result document(s) to the next stage. In combination with various [built-in operators](https://docs.mongodb.com/manual/reference/operator/aggregation/) data can be analyzed and aggregated in many ways.

The body _MUST_ contain:

- `db`: The database name to query.
- `collection`: The collection name within the database to query.
- `pipeline`: MongoDB-compatible array of objects defining as series of [pipeline stages](https://docs.mongodb.com/manual/aggregation/#aggregation-pipeline).

The body _MAY_ contain:

- `limit`: Maximum number of documents to return.
- `options`: MongoDB-compatible object defining aggregate options.

See [MongoDb Collection.aggregate()](http://mongodb.github.io/node-mongodb-native/3.6/api/Collection.html#aggregate) for parameter option details.

### 14.4.1. /data/aggregate Schema

```js
const schema = {
  body: {
    type: 'object',
    required: ['collection', 'db', 'pipeline'],
    properties: {
      collection: {
        $id: 'collection',
        type: 'string',
        minLength: 4,
        maxLength: 190,
      },
      db: {
        $id: 'db',
        type: 'string',
        minLength: 4,
        maxLength: 64,
        pattern: '^(?!coeus).+',
      },
      limit: {
        type: ['number', 'null'],
        default: config.get('db.thresholds.limit.base'),
        minimum: config.get('db.thresholds.limit.minimum'),
        maximum: config.get('db.thresholds.limit.maximum'),
      },
      pipeline: {
        type: 'array',
        uniqueItems: true,
        default: [],
        items: {
          type: 'object',
        },
      },
      options: {
        type: ['object', 'null'],
        default: null,
      },
    },
  },
};
```

### 14.4.2. /data/aggregate Request Examples

**Example**: Count the number of documents which contain have a complex key of `site.subscription.user.email` equal to `gabe@solarixdigital.com`:

```json
{
  "collection": "srn:coeus:wcasg:widget:dashboard::collection",
  "db": "wcasg",
  "pipeline": [
    {
      "$match": {
        "site.subscription.user.email": "gabe@solarixdigital.com"
      }
    },
    {
      "$count": "siteCount"
    }
  ]
}
```

**Response**:

```json
[
  {
    "siteCount": 20
  }
]
```

**Example**: For the above matching documents sum the total of all `requestBytes` fields:

```json
{
  "collection": "srn:coeus:wcasg:widget:dashboard::collection",
  "db": "wcasg",
  "pipeline": [
    {
      "$match": {
        "site.subscription.user.email": "gabe@solarixdigital.com"
      }
    },
    {
      "$group": {
        "_id": 1,
        "totalBytes": {
          "$sum": "$requestBytes"
        }
      }
    }
  ]
}
```

**Response**:

```json
[
  {
    "_id": 1,
    "totalBytes": 3852448
  }
]
```

### 14.4.3. See Also

- [MongoDB Aggregation Documentation](https://docs.mongodb.com/manual/aggregation/)
- [Aggregation Pipeline Stages](https://docs.mongodb.com/manual/reference/operator/aggregation-pipeline/)
- [Aggregation Pipeline Operators](https://docs.mongodb.com/manual/reference/operator/aggregation/)

## 14.5. /data/createIndex

Create indexes used for fast `/data/find` queries and complex pagination.

The body _MUST_ contain:

- `db`: The database name to query.
- `collection`: The collection name within the database to query.
- `fieldOrSpec`: MongoDB-compatible `object` or `string` defining the field or document combination of fields to index.

The body _MAY_ contain:

- `options`: MongoDB-compatible object defining createIndex options.

See [MongoDb Collection.createIndex()](http://mongodb.github.io/node-mongodb-native/3.6/api/Collection.html#createIndex) for parameter option details.

### 14.5.1. /data/createIndex Schema

```js
const schema = {
  body: {
    type: 'object',
    required: ['collection', 'db', 'fieldOrSpec'],
    properties: {
      collection: Collection,
      db: Database,
      fieldOrSpec: {
        type: ['object', 'string'],
        default: null,
      },
      options: {
        type: ['object', 'null'],
        default: null,
      },
    },
  },
};
```

### 14.5.2. /data/createIndex Request Examples

**Example**: Create simple, single-field index by passing a string with the field name:

```json
{
  "collection": "srn:coeus:acme::collection",
  "db": "acme",
  "fieldOrSpec": "active"
}
```

**Response**:

```json
{
  "statusCode": 200,
  "message": "'active_1' index created on 'acme.srn:coeus:acme::collection'"
}
```

**Example**: Create a compound index for the `name` and `email` fields. The `email` key's direction is reversed by specifying a `-1` value:

```json
{
  "collection": "srn:coeus:acme::collection",
  "db": "acme",
  "fieldOrSpec": {
    "name": 1,
    "email": -1
  }
}
```

**Response**:

```json
{
  "statusCode": 200,
  "message": "'name_1_email_-1' index created on 'acme.srn:coeus:acme::collection'"
}
```

## 14.6. /data/delete

To delete one or more documents send a `POST` request to the `/data/delete` endpoint.

The body _MUST_ contain:

- `db`: The database name to access.
- `collection`: The collection name within the database to access.
- `filter`: MongoDB-compatible object defining the filter query on which to base deletion targets.

The body _MAY_ contain:

- `options`: MongoDB-compatible object defining method options.

See [MongoDb Collection.deleteMany()](http://mongodb.github.io/node-mongodb-native/3.6/api/Collection.html#deleteMany) for parameter option details.

### 14.6.1. /data/delete Schema

```js
const schema = {
  type: 'object',
  required: ['collection', 'db', 'filter'],
  properties: {
    collection: schema.collection,
    db: schema.db,
    filter: {
      type: 'object',
      default: {},
    },
    options: {
      type: ['object', 'null'],
      default: null,
    },
  },
};
```

### 14.6.2. /data/delete Request Example

**Example**: Delete all documents with a key `foo` value of `bar` within the `acme.srn:coeus:acme::collection` collection:

```bash
$ curl --location --request POST 'http://localhost:8000/data/delete' \
--header 'Authorization: Bearer <JWT>' \
--header 'Content-Type: application/json' \
--data-raw '{
  "collection": "srn:coeus:acme::collection",
  "db": "acme",
  "filter": {
    "foo": "bar"
  }
}'
```

**Response** indicates the number of deleted documents:

```json
{
  "statusCode": 200,
  "message": "1 document deleted"
}
```

## 14.7. /data/dropIndex

Delete an existing index within a db collection.

The body _MUST_ contain:

- `db`: The database name to query.
- `collection`: The collection name within the database to query.
- `indexName`: The exact name of the index to be deleted.

The body _MAY_ contain:

- `options`: MongoDB-compatible object defining dropIndex options.

See [MongoDb Collection.dropIndex()](http://mongodb.github.io/node-mongodb-native/3.6/api/Collection.html#dropIndex) for parameter option details.

### 14.7.1. /data/dropIndex Schema

```js
const schema = {
  body: {
    type: 'object',
    required: ['collection', 'db', 'indexName'],
    properties: {
      collection: Collection,
      db: Database,
      indexName: {
        type: 'string',
        minLength: 1,
      },
      options: {
        type: ['object', 'null'],
      },
    },
  },
};
```

### 14.7.2. /data/dropIndex Request Examples

**Example**: Remove an existing `active_1` index:

```json
{
  "collection": "srn:coeus:acme::collection",
  "db": "acme",
  "indexName": "active_1"
}
```

**Response**:

```json
{
  "statusCode": 200,
  "message": "'active_1' index dropped from 'acme.srn:coeus:acme::collection'"
}
```

**Example**: Attempting to remove an index that doesn't exist returns an error:

```json
{
  "collection": "srn:coeus:acme::collection",
  "db": "acme",
  "indexName": "active_1"
}
```

**Response**:

```json
{
  "statusCode": 500,
  "code": "27",
  "error": "Internal Server Error",
  "message": "index not found with name [active_1]"
}
```

## 14.8. /data/find

To find one or more documents send a `POST` request to the `/data/find` endpoint.

The body _MUST_ contain:

- `db`: The database name to query.
- `collection`: The collection name within the database to query.
- `query`: MongoDB-compatible object defining query parameters.

The body _MAY_ contain:

- `limit`: Maximum number of documents to return.
- `options`: MongoDB-compatible object defining query options.

See [MongoDb Collection.find()](http://mongodb.github.io/node-mongodb-native/3.6/api/Collection.html#find) for parameter option details.

### 14.8.1. /data/find Schema

```js
const schema = {
  type: 'object',
  required: ['collection', 'db'],
  properties: {
    collection: {
      $id: 'collection',
      type: 'string',
      minLength: 4,
      maxLength: 190,
    },
    db: {
      $id: 'db',
      type: 'string',
      minLength: 4,
      maxLength: 64,
      pattern: '^(?!coeus).+',
    },
    limit: {
      type: ['number', 'null'],
      default: config.get('db.thresholds.limit.base'),
      minimum: config.get('db.thresholds.limit.minimum'),
      maximum: config.get('db.thresholds.limit.maximum'),
    },
    options: {
      type: ['object', 'null'],
      default: null,
    },
    query: {
      type: ['object', 'string'],
      default: {},
    },
  },
};
```

### 14.8.2. /data/find Request Example

**Example**: Perform a full text search for the term `Superman` within the `sample_mflix.movies` collection. Limit to a maximum of `5` documents:

```bash
$ curl --location --request POST 'http://localhost:8000/data/find' \
--header 'Authorization: Bearer <JWT>' \
--header 'Content-Type: application/json' \
--data-raw '{
  "collection": "movies",
  "db": "sample_mflix",
  "query": {
    "$text": {
      "$search": "Superman"
    }
  },
  "limit": 5
}'
```

**Response** is `5` entries similar to this one:

```json
[
  {
    "_id": "573a13dff29313caabdb959c",
    "plot": "Superman and Supergirl take on the cybernetic Brainiac, who boasts that he possesses \"the knowledge and strength of 10,000 worlds.\"",
    "genres": ["Animation", "Action", "Adventure"],
    "runtime": 75,
    "rated": "PG-13",
    "cast": ["Matt Bomer", "Stana Katic", "John Noble", "Molly C. Quinn"],
    "num_mflix_comments": 3,
    "poster": "https://m.media-amazon.com/images/M/MV5BMTkzMjczODQzMV5BMl5BanBnXkFtZTcwOTIyOTQ0OQ@@._V1_SY1000_SX677_AL_.jpg",
    "title": "Superman: Unbound",
    "fullplot": "Offering herself as a hostage, Lois Lane is caught in an aerial confrontation between her terrorist captors and the unpredictable Supergirl before Superman arrives to save the day. Soon after, knowing Superman's civilian identity, Lois attempts to get Clark Kent to make their relationship public despite his fear of the consequences, but their argument is halted by a Daily Planet staff meeting before Kent leaves when they are being alerted to a meteor. Intercepting it, Superman learns the meteor to be a robot and that he promptly defeats before activating its beacon and taking it to the Fortress of Solitude. With help from a fear-filled Supergirl, Superman learns the robot is actually a drone controlled by a being named Brainiac, a cyborg who was originally a Coluan scientist who subjected himself to extensive cybernetic and genetic enhancements. As Supergirl reveals from her experience with the monster, Brainiac seized and miniaturized Krypton's capital city of Kandor prior to the planet's destruction with her father and mother attempting to track him down before they mysteriously lost contact with Krypton. Fearing more drones would come, Superman goes flying all through the galaxy in an attempt to track down Brainiac before finding his drones attacking a planet. Though he attempted to stop them, Superman witnesses Brainiac capture the planet's capital like he did with Kandor before firing a Solar Aggressor missile to have the planet be consumed by the exploding sun. The explosion knocks Superman unconscious and he is brought upon Brainiac's ship, coming to in the examination room and fighting his way through the vessel before he discovers a room full of bottled cities prior to being attacked by Brainiac. At this point, confirming that he spared Krypton because of its eventual destruction, Brainiac reveals that he has been collecting information of all the planets he visited before destroying them. Using Superman's spacecraft, Brainiac decides to chart a course to Earth while sending Superman into Kandor. Inside Kandor, his strength waning due to the artificial red sun, Superman meets his uncle Zor-El and aunt Alura. After spending time with them, Superman formulates a plan and escapes Kandor using the subjugator robots. From there, Superman disables Brainiac's ship and spirits Kandor to Earth. At that time, Lois learns from Supergirl of why Superman left and alerts the Pentagon for a possible invasion by Brainiac as he eventually repaired his ship and arrives to Metropolis. Despite everyone, including Supergirl, doing their best to fend his drones off, Metropolis is encased in a bottle and both Superman and Supergirl are captured. Having hooked Superman up to his ship, revealing that Earth offers nothing to him, Brainiac tortures Superman to obtain Kandor before destroying the planet. However, telling his captor what Earth means to him, Superman breaks free and then frees Supergirl and convinces her to stop the Solar-Aggressor from hitting the sun. Remembering Zor-El's words about Brainiac's ideals, Superman knocks him out of the ship and they crash into a swamp. As he fights Braniac, Superman forces the cyborg to experience the chaos of life itself outside of his safe, artificial environments he created. Eventually, the combined mental and physical strain reaches its toll on Brainiac as he combusts and is reduced to ash and molten machinery. After restoring Metropolis, taking Kandor to another planet to restore it to its normal size to establish a Kryptonian colony, Superman makes his love life with Lois as Kent public with a marriage proposal. However, placed in the Fortress of Solitude, Brainiac's remains are still active.",
    "languages": ["English"],
    "released": "2013-05-23T00:00:00.000Z",
    "directors": ["James Tucker"],
    "writers": [
      "Bob Goodman",
      "Geoff Johns (graphic novel: \"Superman: Brainiac\")",
      "Gary Frank (graphic novel: \"Superman: Brainiac\")",
      "Jerry Siegel (creator)",
      "Joe Shuster (creator)",
      "Jerry Ordway (creator)",
      "Tom Grummet (creator)"
    ],
    "awards": {
      "wins": 0,
      "nominations": 3,
      "text": "3 nominations."
    },
    "lastupdated": "2015-08-31 00:27:43.340000000",
    "year": 2013,
    "imdb": {
      "rating": 6.6,
      "votes": 6421,
      "id": 2617456
    },
    "countries": ["USA"],
    "type": "movie",
    "tomatoes": {
      "viewer": {
        "rating": 2.4,
        "numReviews": 5
      },
      "lastUpdated": "2015-07-04T18:27:40.000Z"
    }
  }
]
```

## 14.9. /data/insert

To insert one or more documents send a `POST` request to the `/data/insert` endpoint.

The body _MUST_ contain:

- `db`: The database name to query.
- `collection`: The collection name within the database to query.
- `document`: An array of object(s) to be inserted.

The body _MAY_ contain:

- `ordered`: If true, when an insert fails, don't execute the remaining writes. If false, continue with remaining inserts when one fails. Default: `true`

See [MongoDb Collection.insertMany()](http://mongodb.github.io/node-mongodb-native/3.6/api/Collection.html#insertMany) for parameter option details.

### 14.9.1. /data/insert Schema

```js
const schema = {
  type: 'object',
  required: ['collection', 'db', 'document'],
  properties: {
    collection: schema.collection,
    db: schema.db,
    document: {
      type: 'array',
      items: {
        type: 'object',
        default: {},
      },
      minItems: 1,
    },
    ordered: {
      type: 'boolean',
      default: true,
    },
  },
};
```

### 14.9.2. /data/insert Request Example

**Example**: Insert 3 simple documents into the `acme.srn:coeus:acme::collection` collection:

```bash
$ curl --location --request POST 'http://localhost:8000/data/insert' \
--header 'Authorization: Bearer <JWT>' \
--header 'Content-Type: application/json' \
--data-raw '{
  "collection": "srn:coeus:acme::collection",
  "db": "acme",
  "document": [
    {
      "data": "foo"
    },
    {
      "data": "bar"
    },
    {
      "data": "baz"
    }
  ]
}'
```

**Response**: Indicates the number of documents inserted and returns the unique `_id` array.

```json
{
  "statusCode": 200,
  "message": "3 documents inserted",
  "data": [
    "5f5c2046f95d0460e0856827",
    "5f5c2046f95d0460e0856828",
    "5f5c2046f95d0460e0856829"
  ]
}
```

## 14.10. /data/listIndexes

Get all existing indexes within the db collection.

The body _MUST_ contain:

- `db`: The database name to query.
- `collection`: The collection name within the database to query.

The body _MAY_ contain:

- `batchSize`: The batchSize for the returned command cursor.
- `readPreference`: The preferred read preference.

See [MongoDb Collection.listIndexes()](http://mongodb.github.io/node-mongodb-native/3.6/api/Collection.html#listIndexes) for parameter option details.

### 14.10.1. /data/listIndexes Schema

```js
const schema = {
  body: {
    type: 'object',
    required: ['collection', 'db'],
    properties: {
      batchSize: {
        type: 'number',
      },
      collection: Collection,
      db: Database,
      readPreference: {
        type: 'string',
        enum: [
          'primary',
          'primaryPreferred',
          'secondary',
          'secondaryPreferred',
          'nearest',
        ],
      },
    },
  },
};
```

### 14.10.2. /data/listIndexes Request Examples

**Example**: Retrieve all indexes in `acme.srn:coeus:acme::collection`:

```json
{
  "collection": "srn:coeus:acme::collection",
  "db": "acme"
}
```

**Response**:

```json
{
  "statusCode": 200,
  "message": "2 indexes found on 'acme.srn:coeus:acme::collection'",
  "data": [
    {
      "v": 2,
      "key": {
        "_id": 1
      },
      "name": "_id_"
    },
    {
      "v": 2,
      "key": {
        "name": 1,
        "email": -1
      },
      "name": "name_1_email_-1"
    }
  ]
}
```

## 14.11. /data/update

Update one or more documents by sending a `POST` request to the `/data/update` endpoint.

The body _MUST_ contain:

- `db`: The database name to access.
- `collection`: The collection name within the database to access.
- `filter`: MongoDB-compatible object defining the filter query on which to base update targets.
- `update`: MongoDB-compatible object defining the document shape with which to update.

The body _MAY_ contain:

- `options`: MongoDB-compatible object defining method options.

See [MongoDb Collection.updateMany()](http://mongodb.github.io/node-mongodb-native/3.6/api/Collection.html#updateMany) for parameter option details.

### 14.11.1. /data/update Schema

```js
const schema = {
  type: 'object',
  required: ['collection', 'db', 'filter', 'update'],
  properties: {
    collection: schema.collection,
    db: schema.db,
    filter: {
      type: 'object',
      minProperties: 1,
      default: null,
    },
    options: {
      type: ['object', 'null'],
      default: null,
    },
    update: {
      type: 'object',
      minProperties: 1,
      default: null,
    },
  },
};
```

### 14.11.2. /data/update Request Example

**Example**: Update all documents with a `data` key value that matches the RegEx `^bar` (begins with 'bar') within the `acme.srn:coeus:acme::collection` collection. For all matched documents, set the `data` key value to `foo`:

```bash
$ curl --location --request POST 'http://localhost:8000/data/update' \
--header 'Authorization: Bearer <JWT>' \
--header 'Content-Type: application/json' \
--data-raw '{
    "collection": "srn:coeus:acme::collection",
    "db": "acme",
    "filter": {
        "data": {
            "$regex": "^bar"
        }
    },
    "update": {
        "$set": {
            "data": "foo"
        }
    }
}'
```

**Response** indicates the number of updated documents:

```json
{
  "statusCode": 200,
  "message": "2 documents updated"
}
```

# 15. TODO

## 15.1. Compression

- [x] Integrate response payload compression.

## 15.2. In-Memory Cache of User Data

- [x] Using JWT payload to determine Policy is preferred for speed, but the only means of disabling a User for an Admin is to wait for JWT token expiration.
- [x] Cache User db permissions in-memory for validation against request. Update in-memory cache on any relevant successful `/user` endpoint request.
- [x] Use `coeus.users` `hash` property for fast validation, comparing JWT hash to in-memory hash.

Generate `hash` when:

- [x] User document inserted into db
- [x] User document updated in db

## 15.3. /data/delete Logic Check: `_id` Property

- [x] If a `/data/delete` `filter` object key is `_id`, perform backend conversion of value to `ObjectId(value)` before making MongoDB request to ensure proper parsing.

## 15.4. Pagination

- [x] see: https://docs.mongodb.com/manual/indexes/#indexes

## 15.5. Logging

- [x] Integrate into CloudWatch logs

## 15.6. Caching

- Apply request caching; in-memory or Redis-powered.

## 15.7. CORS Support

- [x] Add CORS support.

## 15.8. /user/register Option: email

- [x] Integrate email support (AWS SES)
- [x] Email verification token to user email address.
- [x] Clicking should set User `verified = true`.
- [x] Upon verification email user with confirmation and inform to await admin activation.

## 15.9. /user/login Option: email

- [x] Email generated JWT token to user email address.
- [x] User must be `active` and `verified`.
- [x] Send attachment

## 15.10. /user/activate Endpoint

- [x] Endpoint for activating specified User(s), so they can login, recieve JWT, and make requests.
- [x] Upon activation email User with JWT.
- [x] Send attachment

## 15.11. Request Option: idempotence_id

- Mutable action requests (i.e. `delete`, `insert`, `update`) should have an `idempotence_id` to ensure repeated requests are not processed multiple times.

## 15.12. Request Option: format

- json, csv, etc: Allow output format of results

## 15.13. Request Option: email

- [x] Email address to send results to. Requires background worker system.

## 15.14. Benchmarking

- [x] Determine basic route endpoint benchmarks/limitations to gauge proper rate limiting ranges

## 15.15. Rate Limiting

- [x] Set default rate limits, overridable within validated range by PolicyStatement

## 15.16. PolicyStatement Property: rateLimit

- [x] Override rate limiting for matching service/method requests

## 15.17. PolicyStatement Property: ip

- [x] Restrict requests from ip address/range

## 15.18. PolicyStatement Property: hostname

- [x] Restrict requests from hostname

## 15.19. /user/explain Endpoint

- Explains Policy rules of specified User(s), such as `db` and `collection` access, and associated `service` and `method` allowances.

## 15.20. API Documentation Generator

- Swagger or similar tool?

## 15.21. Commit Release Update

- https://github.com/conventional-changelog/conventional-changelog
- https://github.com/conventional-changelog/standard-version
- https://github.com/semantic-release/commit-analyzer#release-rules

## 15.22. Two-Factor Authentication (TOTP)

- Add two-factor auth for `/user/login` and similar endpoints
- Package: https://github.com/yeojz/otplib
