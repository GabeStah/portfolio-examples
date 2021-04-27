**Note: This README is extracted from the original source project that I created. Please see [WARNING & COPYRIGHT NOTICE](../../../README.md#warning--copyright-notice) for more info.**

- [1. Import Pipeline](#1-import-pipeline)
  - [1.1. Pipeline Overview](#11-pipeline-overview)
    - [Import Command Options](#import-command-options)
  - [1.2. `import:category`](#12-importcategory)
    - [1.2.1. ClientName-v1 Data Export](#121-clientname-v1-data-export)
    - [1.2.2. Normalization and Mapping](#122-normalization-and-mapping)
    - [1.2.3. Filtered Data](#123-filtered-data)
    - [1.2.4. Extra Data](#124-extra-data)
    - [1.2.5. Finalized Database Imports](#125-finalized-database-imports)
    - [1.2.6. JSON Dump](#126-json-dump)
  - [1.3. `import:subcategory`](#13-importsubcategory)
    - [1.3.1. ClientName-v1 Data Export](#131-clientname-v1-data-export)
    - [1.3.2. Normalization and Mapping](#132-normalization-and-mapping)
      - [1.3.2.1. Parent Determination](#1321-parent-determination)
    - [1.3.3. Filtered Data](#133-filtered-data)
    - [1.3.4. Extra Data](#134-extra-data)
    - [1.3.5. Finalized Database Imports](#135-finalized-database-imports)
    - [1.3.6. JSON Dump](#136-json-dump)
  - [1.4. `import:product`](#14-importproduct)
    - [1.4.1. ClientName-v1 Data Export](#141-clientname-v1-data-export)
    - [1.4.2. Normalization and Mapping](#142-normalization-and-mapping)
      - [1.4.2.1. Attributes](#1421-attributes)
      - [1.4.2.2. Categorization](#1422-categorization)
      - [1.4.2.3. Images](#1423-images)
      - [1.4.2.4. Taxons](#1424-taxons)
      - [1.4.2.5. Descriptions](#1425-descriptions)
    - [1.4.3. Filtered Data](#143-filtered-data)
    - [1.4.4. Extra Data](#144-extra-data)
    - [1.4.5. Finalized Database Imports](#145-finalized-database-imports)
    - [1.4.6. JSON Dump](#146-json-dump)
  - [1.5. `import:product-variant`](#15-importproduct-variant)
    - [1.5.1. ClientName-v1 Data Export](#151-clientname-v1-data-export)
    - [1.5.2. Normalization and Mapping](#152-normalization-and-mapping)
      - [1.5.2.1. Options](#1521-options)
      - [1.5.2.2. Pricing](#1522-pricing)
      - [1.5.2.3. Product: Water Heater](#1523-product-water-heater)
    - [1.5.3. Filtered Data](#153-filtered-data)
    - [1.5.4. Extra Data](#154-extra-data)
    - [1.5.5. Finalized Database Imports](#155-finalized-database-imports)
    - [1.5.6. JSON Dump](#156-json-dump)
  - [1.6. `import:dealer`](#16-importdealer)
    - [1.6.1. Prerequisite](#161-prerequisite)
    - [1.6.2. Entity Fields](#162-entity-fields)
    - [1.6.3. ClientName-v1 Data Export](#163-clientname-v1-data-export)
    - [1.6.4. Normalization and Mapping](#164-normalization-and-mapping)
    - [1.6.5. Filtered Data](#165-filtered-data)
    - [1.6.6. Extra Data](#166-extra-data)
    - [1.6.7. Finalized Database Imports](#167-finalized-database-imports)
    - [1.6.8. JSON Dump](#168-json-dump)
  - [1.7. `import:zone`](#17-importzone)
    - [1.7.1. Data Source](#171-data-source)
    - [1.7.2. Normalization and Mapping](#172-normalization-and-mapping)
    - [1.7.3. Filtered Data](#173-filtered-data)
    - [1.7.4. Extra Data](#174-extra-data)
    - [1.7.5. Enabling/Modifying Tax Rates](#175-enablingmodifying-tax-rates)
- [2. Shipping](#2-shipping)
  - [2.1. Shipping Methods](#21-shipping-methods)
  - [2.2. Shipping Calculator](#22-shipping-calculator)
  - [2.3. Issue: Handling Multiple Rates](#23-issue-handling-multiple-rates)
  - [2.4. Issue: Provider API Request Throttling](#24-issue-provider-api-request-throttling)
  - [2.5. Issue: Orders w/ Invalid Shipping Methods](#25-issue-orders-w-invalid-shipping-methods)
  - [Issue: Expanded International Shipping](#issue-expanded-international-shipping)
- [3. Static Page Imports](#3-static-page-imports)
- [4. Dealers](#4-dealers)
  - [4.1. Nearby Dealers (Geolocation)](#41-nearby-dealers-geolocation)
  - [4.2. Frontend Dealers Pages](#42-frontend-dealers-pages)
  - [4.3. Administration](#43-administration)
- [5. Files](#5-files)

## 1. Import Pipeline

The pipeline that imports ClientName-v1 data into ClientName-v2/Sylius is executed via a series of custom CLI commands. All
pipeline commands are fully idempotent, allowing a given command to run multiple times but always producing the same
imported data result.

### 1.1. Pipeline Overview

Each import pipeline command executes the following stages, adjusting as appropriate to the ClientName-v1 data and the
targeted entities.

1. A `php bin/console import:<command>` is executed.
2. An appropriate [ImportCommand](src/Command/Import) class catches the CLI execution.
3. The `ImportCommand` injects the matching [Importer](src/Service/Importer) class.
4. A [Provider](src/Service/Importer/Provider) is assigned to the `Importer` instance. This `Provider` determines how the importer retrieves data. At present, data providers can be via SQL (i.e. ClientName V1) or JSON.
5. A [Normalizer](src/Service/Importer/Normalizer) is also assigned to the `Importer` instance. This `Normalizer` determines how data is normalized between the original `Provider` and the expected types within the target `Importer`. For example, the [v1/ProductVariantNormalizer](src/Service/Importer/Normalizer/v1/ProductVariantNormalizer.php) normalizes all the fields passed from the ClientName V1 MySQL database, while the [v2/ProductVariantNormalizer](src/Service/Importer/Normalizer/v2/ProductVariantNormalizer.php) only handles the minimal fields provided in a JSON file.

- Normalized data includes attributes, categories (taxons), dimensions, images, meta data, product options, product variants, pricing, and so forth.

6. The normalized dataset is mapped and merged with optional extra data.
7. Mapped data is filtered to remove any exceptional records (i.e. duplicate unique key types).
8. Then each mapped record is passed through a series of Entity creation steps necessary to generate the appropriate
   Sylius Entity with relevant data, associated Entities, and relationships.

- New database records are generated while existing records are updated.

9. Lastly the new/updated dataset is serialized and exported to the [/exports](/exports) directory.

#### Import Command Options

Import commands can accept extra arguments, some optional and some required, depending on the desired data `Provider` and `Normalizer`. All arguments are specified using double-dash flag syntax, followed by the parameter value. For example: `--flag value` or `--flag=value`.

- `provider`: Specifies the `Provider` type to use. Current values: `json` or `sql`
- `path`: Path to the source file. For an SQL `Provider` this is an SQL file path containing the query to execute within the `Provider`. For JSON `Providers` this is the path to the source JSON file.

### 1.2. `import:category`

Transfers all ClientName-v1 `productcategory` records into top-level Sylius `Taxonomies`.

#### 1.2.1. ClientName-v1 Data Export

All `enabled` product categories that are _NOT_ `trudesign`-related are exported:

```sql
SELECT
    *
FROM
    productcategory
WHERE
    pcategoryurl NOT LIKE "trudesign%"
AND
    catstatus = 1
```

#### 1.2.2. Normalization and Mapping

A new `Taxon` entity is created for each incoming record with a few important mappings:

- `code`: Snakified v1 `pcategoryurl`, prefixed with `category_`
- `slug`: Slugified v1 `pcategoryurl`
- `name`: Titlized v1 `pcategoryname`
- `parent`: All top-level `Taxons` are assigned a parent `Taxon` of `MENU_CATEGORY`. This allows UI components to render
  navigation menus from the single top-level `Taxon`.
- `category_id`: v1 `productcategory.catid` value for future pipeline reference
- `category_type`: Static `category` or `subcategory` designation for future pipeline reference

#### 1.2.3. Filtered Data

No filters are applied beyond the initial SQL statement.

#### 1.2.4. Extra Data

A new `miscellaneous` `Taxon` is added to the import data and is used later in the pipeline as a last-resort catchall
for otherwise non-categorized `Products`.

```php
[
  'category_id' => 100,
  'category_type' => 'category',
  'code' => StringNormalizer::toSnake('category-' . 'miscellaneous'),
  'description' => null,
  'enabled' => boolval(true),
  'name' => StringNormalizer::toTitle('Miscellaneous'),
  'parent' => [
    'category_id' => 999,
    'category_type' => 'menu',
  ],
  'slug' => StringNormalizer::toSlug('miscellaneous'),
  'timestamp' => time(),
]
```

#### 1.2.5. Finalized Database Imports

Generated `sylius_taxon` data:

| id  | parent_id | code                               | tree_left | tree_right | tree_level | position | enabled | category_id | category_type |
| --- | --------- | ---------------------------------- | --------- | ---------- | ---------- | -------- | ------- | ----------- | ------------- |
| 1   |           | MENU_CATEGORY                      | 1         | 130        | 0          | 0        | 1       | 999         | menu          |
| 494 | 1         | category_alarms                    | 112       | 115        | 1          | 1        | 1       | 27          | category      |
| 495 | 1         | category_holding_tanks             | 104       | 111        | 1          | 3        | 1       | 21          | category      |
| 496 | 1         | category_legacy_products           | 88        | 103        | 1          | 6        | 1       | 26          | category      |
| 497 | 1         | category_marine_ice_makers         | 84        | 87         | 1          | 8        | 1       | 19          | category      |
| 498 | 1         | category_marine_sanitation_devices | 74        | 83         | 1          | 9        | 1       | 12          | category      |
| 499 | 1         | category_marine_toilets            | 46        | 73         | 1          | 10       | 1       | 14          | category      |
| 500 | 1         | category_marine_water_heater       | 42        | 45         | 1          | 11       | 1       | 24          | category      |
| 501 | 1         | category_miscellaneous             | 2         | 3          | 1          | 4        | 1       | 100         | category      |
| 502 | 1         | category_odor_control              | 30        | 41         | 1          | 12       | 1       | 20          | category      |
| 503 | 1         | category_ruddermaster              | 24        | 29         | 1          | 13       | 1       | 25          | category      |
| 504 | 1         | category_sanitation_accessories    | 12        | 23         | 1          | 14       | 1       | 22          | category      |
| 505 | 1         | category_strainers                 | 4         | 11         | 1          | 15       | 1       | 23          | category      |

#### 1.2.6. JSON Dump

All V2 category `Taxon` data can be found in [exports/category.json](exports/category.json).

### 1.3. `import:subcategory`

Transfers all ClientName-v1 `productsubcategory` records into second-level Sylius `Taxonomies`.

#### 1.3.1. ClientName-v1 Data Export

All product subcategories that are _NOT_ direct children of a `trudesign` `productcategory` are exported:

```sql
SELECT
    *
  FROM
    productsubcategory
  WHERE
    categoryid !=
    (
      SELECT
        catid
      FROM
        productcategory
      WHERE
        pcategoryurl LIKE "trudesign%"
    )
  AND
    subcaturl != ''
```

#### 1.3.2. Normalization and Mapping

A new `Taxon` entity is created for each incoming record with a few important mappings:

- `code`: Snakified v1 `subcaturl`, prefixed with `subcategory_`
- `slug`: Slugified v1 `subcaturl`
- `name`: Titlized v1 `subcatname`
- `parent`: See below
- `category_id`: v1 `productcategory.catid` value for future pipeline reference
- `category_type`: Static `category` or `subcategory` designation for future pipeline reference

##### 1.3.2.1. Parent Determination

ClientName-v1 data is generally _non-relational_, instead duplicating and flattening data within singular records.

V1 `productsubcategory` records store their child-parent relationships in a series of `parent#` columns, as seen in the
exported snippet below:

| subcatid | parent2 | parent3 | parent4 | categoryid | subcaturl              |
| -------- | ------- | ------- | ------- | ---------- | ---------------------- |
| 1        | 0       | 0       | 0       | 14         | electric-marine-toilet |
| 2        | 0       | 0       | 0       | 14         | manual-marine-toilets  |
| 3        | 0       | 0       | 0       | 18         | connectors             |
| 4        | 0       | 0       | 0       | 18         | reducers               |
| 5        | 0       | 0       | 0       | 18         | threaded-connectors    |
| 6        | 0       | 0       | 0       | 18         | seacocks               |
| 7        | 0       | 0       | 0       | 18         | valves                 |
| 8        | 0       | 0       | 0       | 18         | y-valves-manual        |
| 9        | 0       | 0       | 0       | 18         | thru-hull-fittings     |
| 10       | 0       | 0       | 0       | 18         | vented-loop            |
| 11       | 1       | 0       | 0       | 14         | atlantes-freedom       |
| 12       | 1       | 0       | 0       | 14         | marine-elegance        |
| 13       | 1       | 0       | 0       | 14         | seaera                 |
| 14       | 1       | 11      | 0       | 14         | parts                  |
| 17       | 2       | 0       | 0       | 14         | ph-superflush          |
| 18       | 2       | 17      | 0       | 14         | parts                  |
| 19       | 2       | 0       | 0       | 14         | fresh-head             |
| 20       | 2       | 19      | 0       | 14         | parts                  |
| 21       | 0       | 0       | 0       | 10         | icerette               |
| 22       | 21      | 0       | 0       | 10         | parts                  |
| 23       | 0       | 0       | 0       | 19         | icerette               |
| 24       | 23      | 0       | 0       | 19         | parts                  |
| 25       | 1       | 0       | 0       | 14         | phiie                  |
| 26       | 2       | 0       | 0       | 14         | phii                   |
| 27       | 2       | 26      | 0       | 14         | parts                  |
| 28       | 0       | 0       | 0       | 12         | electroscan            |
| 29       | 28      | 0       | 0       | 12         | parts                  |
| 30       | 28      | 0       | 0       | 12         |                        |

To reconcile this the `SubcategoryImporter` determines the _most nested_ subcategory relationship of a given record
during import. For example, `productsubcategory.parent4` entry indicates a higher priority child-parent relationship
than all other `parent#` or even `categoryid` entries.

Since Sylius maintains proper data relationships each subcategory `Taxon` is assigned to one and only one parent `Taxon`
.

#### 1.3.3. Filtered Data

V1 `productsubcategory` contains (4) duplicate records that match the url (`slug`) of another record:

| subcatid | subcaturl              |
| -------- | ---------------------- |
| 21       | icerette               |
| 63       | reverse-polarity-alarm |
| 114      | electroscan            |
| 152      | phii                   |

These four records were filtered out during V2 import.

In addition, dozens of duplicate `parts` and `accessories` subcategories exist but serve only to complicate the
relational structure of categories. Their existence resembles "tags" rather than singular categories, so all `parts`
and `accessories` subcategories are removed during V2 import.

**NOTE**: To retain the intended purpose of `parts` categorizations within V2 the `ProductImporter`
/ `ProductVariantImporter` services use V1 categorizations to apply a ` part``ProductAttributes ` flag where
appropriate.

#### 1.3.4. Extra Data

No extra data was added to subcategory imports.

#### 1.3.5. Finalized Database Imports

Generated `sylius_taxon` data:

| id  | parent_id | code                                             | tree_left | tree_right | tree_level | position | enabled | category_id | category_type |
| --- | --------- | ------------------------------------------------ | --------- | ---------- | ---------- | -------- | ------- | ----------- | ------------- |
| 506 | 499       | subcategory_electric_marine_toilet               | 47        | 62         | 2          | 0        | 1       | 1           | subcategory   |
| 507 | 499       | subcategory_manual_marine_toilets                | 63        | 72         | 2          | 1        | 1       | 2           | subcategory   |
| 508 | 506       | subcategory_atlantes_freedom                     | 48        | 49         | 3          | 0        | 1       | 11          | subcategory   |
| 509 | 506       | subcategory_marine_elegance                      | 50        | 51         | 3          | 1        | 1       | 12          | subcategory   |
| 510 | 506       | subcategory_seaera                               | 52        | 53         | 3          | 2        | 1       | 13          | subcategory   |
| 511 | 507       | subcategory_ph_superflush                        | 64        | 65         | 3          | 0        | 1       | 17          | subcategory   |
| 512 | 507       | subcategory_fresh_head                           | 66        | 67         | 3          | 1        | 1       | 19          | subcategory   |
| 513 | 497       | subcategory_icerette                             | 85        | 86         | 2          | 0        | 1       | 23          | subcategory   |
| 514 | 506       | subcategory_phiie                                | 54        | 55         | 3          | 3        | 1       | 25          | subcategory   |
| 515 | 507       | subcategory_phii                                 | 68        | 69         | 3          | 2        | 1       | 26          | subcategory   |
| 516 | 498       | subcategory_electroscan                          | 75        | 76         | 2          | 0        | 1       | 28          | subcategory   |
| 517 | 506       | subcategory_seaera_qc                            | 56        | 57         | 3          | 4        | 1       | 33          | subcategory   |
| 518 | 506       | subcategory_ph_powerflush                        | 58        | 59         | 3          | 5        | 1       | 34          | subcategory   |
| 519 | 498       | subcategory_purasan                              | 77        | 78         | 2          | 1        | 1       | 35          | subcategory   |
| 520 | 498       | subcategory_hold_n_treat_system                  | 79        | 80         | 2          | 2        | 1       | 36          | subcategory   |
| 521 | 498       | subcategory_managerm                             | 81        | 82         | 2          | 3        | 1       | 37          | subcategory   |
| 522 | 502       | subcategory_sani_flex_odor_shield_hose           | 31        | 32         | 2          | 0        | 1       | 38          | subcategory   |
| 523 | 502       | subcategory_ko_kills_odors                       | 33        | 34         | 2          | 1        | 1       | 39          | subcategory   |
| 524 | 502       | subcategory_cp_cleans_potties                    | 35        | 36         | 2          | 2        | 1       | 40          | subcategory   |
| 525 | 502       | subcategory_ch_cleans_hoses                      | 37        | 38         | 2          | 3        | 1       | 41          | subcategory   |
| 526 | 502       | subcategory_ko_kills_odors_kit                   | 39        | 40         | 2          | 4        | 1       | 42          | subcategory   |
| 527 | 495       | subcategory_semi_custom_holding_tanks            | 105       | 106        | 2          | 0        | 1       | 43          | subcategory   |
| 528 | 504       | subcategory_macerator_pump                       | 13        | 14         | 2          | 0        | 1       | 44          | subcategory   |
| 529 | 504       | subcategory_vented_loops                         | 15        | 16         | 2          | 1        | 1       | 45          | subcategory   |
| 530 | 504       | subcategory_cleans_hoses                         | 17        | 18         | 2          | 2        | 1       | 46          | subcategory   |
| 531 | 504       | subcategory_tank_monitor                         | 19        | 20         | 2          | 3        | 1       | 47          | subcategory   |
| 532 | 505       | subcategory_raw_water_strainer                   | 5         | 6          | 2          | 0        | 1       | 59          | subcategory   |
| 533 | 505       | subcategory_fresh_water_strainer                 | 7         | 8          | 2          | 1        | 1       | 60          | subcategory   |
| 534 | 505       | subcategory_in_line_strainer                     | 9         | 10         | 2          | 2        | 1       | 61          | subcategory   |
| 535 | 500       | subcategory1700_series                           | 43        | 44         | 2          | 0        | 1       | 62          | subcategory   |
| 536 | 503       | subcategory_mk2_mk3                              | 25        | 26         | 2          | 0        | 1       | 64          | subcategory   |
| 537 | 496       | subcategory_crown_head                           | 89        | 90         | 2          | 0        | 1       | 65          | subcategory   |
| 538 | 496       | subcategory_crown_head_cd_series                 | 91        | 92         | 2          | 1        | 1       | 66          | subcategory   |
| 539 | 496       | subcategory_crown_head_ii                        | 93        | 94         | 2          | 2        | 1       | 67          | subcategory   |
| 540 | 496       | subcategory_compact_ii                           | 95        | 96         | 2          | 3        | 1       | 68          | subcategory   |
| 541 | 496       | subcategory_ice_maker8283                        | 97        | 98         | 2          | 4        | 1       | 69          | subcategory   |
| 542 | 496       | subcategory_ice_maker_model8485                  | 99        | 100        | 2          | 5        | 1       | 70          | subcategory   |
| 543 | 503       | subcategory_mk5_mk6                              | 27        | 28         | 2          | 1        | 1       | 112         | subcategory   |
| 544 | 504       | subcategory_electric_toilet_controls             | 21        | 22         | 2          | 4        | 1       | 113         | subcategory   |
| 545 | 506       | subcategory_electric_marine_toilet_control_parts | 60        | 61         | 3          | 6        | 1       | 119         | subcategory   |
| 546 | 507       | subcategory_multiple_products_use_parts          | 70        | 71         | 3          | 3        | 1       | 120         | subcategory   |
| 547 | 494       | subcategory_reverse_polarity_alarm               | 113       | 114        | 2          | 0        | 1       | 136         | subcategory   |
| 548 | 496       | subcategory_lectra_san                           | 101       | 102        | 2          | 6        | 1       | 142         | subcategory   |
| 549 | 495       | subcategory_holding_tank_monitor                 | 107       | 108        | 2          | 1        | 1       | 150         | subcategory   |
| 550 | 495       | subcategory_holding_tanks_drawings               | 109       | 110        | 2          | 2        | 1       | 154         | subcategory   |

#### 1.3.6. JSON Dump

All V2 subcategory `Taxon` data can be found in [exports/subcategory.json](exports/subcategory.json).

### 1.4. `import:product`

Transfers all top-level ClientName-v1 `products` records into Sylius `Products` along with all necessary related entities:

- `ProductAttribute`
- `ProductAttributeValue`
- `ProductImage`
- `ProductTaxon`

#### 1.4.1. ClientName-v1 Data Export

All `enabled` products with a unique `productnumber` that are _NOT_ `trudesign`-related are exported:

```sql
SELECT
  p.*,
  s1.subcatid subcatid1,
  s1.subcatname subcatname1,
  s1.subcaturl subcaturl1,
  s2.subcatid subcatid2,
  s2.subcatname subcatname2,
  s2.subcaturl subcaturl2,
  s3.subcatid subcatid3,
  s3.subcatname subcatname3,
  s3.subcaturl subcaturl3,
  s4.subcatid subcatid4,
  s4.subcatname subcatname4,
  s4.subcaturl subcaturl4
FROM
  (
    SELECT
      p.productnumber,
      MAX(p.prid) AS prid
    FROM
      products p
    GROUP BY
      p.productnumber
  ) p_max
INNER JOIN
  products p
  ON
    p_max.productnumber = p.productnumber
  AND
    p_max.prid = p.prid
LEFT JOIN
  productsubcategory s1
  ON
    p.psubcategory = s1.subcatid
LEFT JOIN
  productsubcategory s2
  ON
    p.psubcategory3 = s2.subcatid
LEFT JOIN
  productsubcategory s3
  ON
    p.psubcategory4 = s3.subcatid
LEFT JOIN
  productsubcategory s4
  ON
    p.psubcategory5 = s4.subcatid
WHERE
  seourl NOT LIKE '%trudesign%'
AND
  prstatus = 1
ORDER BY
  productnumber ASC
```

#### 1.4.2. Normalization and Mapping

A new `Product` entity is created for each incoming record with a few important mappings:

- `code`: V1 `productnumber`. V1 `products` data is all flattened and makes no relational distinction between parent
  products and child product variants. Put another way, ALL v1 `products` records can be thought of as product variants,
  with the only distinct parent product field being `productnumber`. This is why the SQL export only selects a single
  record for each unique `productnumber`.
- `name`: Titlized v1 `productname`
- `price`: V1 `punitprice`, converted from dollars to whole cents
- `shortdescription`: v1 `metadesc`, if exists
- `slug`: Slugified v1 `seourl`

##### 1.4.2.1. Attributes

`ProductAttributes` are generated via an [AttributeObject](src/Object/Attribute.php) instance, based on the existence of
mapped v1 `products` field values. For example, a non-empty v1 `pdualvoltage` value is converted to a
v2 `dual_voltage` `ProductAttribute` boolean entity with a `true` value.

In addition to explicit attribute conversions the `ProductImporter` also determines if the incoming record was assigned
to a v1 `parts` or `accessories` subcategory:

```php
public function getCategoryAttribute($value)
{
  if ($value == 'part' || $value == 'parts') {
    return new AttributeObject(
      'part',
      'Part',
      true,
      CheckboxAttributeType::TYPE
    );
  }
  if ($value == 'accessory' || $value == 'accessories') {
    return new AttributeObject(
      'accessory',
      'Accessory',
      true,
      CheckboxAttributeType::TYPE
    );
  }
  return null;
}
```

The `Product` is assigned the appropriate `part` or `accessory` `ProductAttribute` if required.

##### 1.4.2.2. Categorization

Similar to v1 `productsubcategory` the v1 `products` subcategories are listed flat within a given record. To address
this, each imported `Product` determines its primary parent category `Taxon` based on `subcategory_id`
and `subcategory_type` within previously-imported `Taxon` entities. The most-nested `subcatid` of a v1 `products` record
indicates the main `Taxon` of the `Product`. If _no_ parent subcategory can be determined then the `Product` is assigned
to the `miscellaneous` top-level category.

##### 1.4.2.3. Images

ClientName-v1 product images are all stored in the local filesystem and indicated by the existence of a relative file path
value within a handful of v1 `products` fields (i.e. `pmainimage`, `pmobileimage1`, `p2mainimage`, etc).

The `ProductImporter` creates normalized and maps existing images to [ProductImageObjects](src/Object/ProductImage.php)
then generates the `ProductImage` entity associated with the `Product`. Depending on the original v1 field name
the `ProductImage.type` is set to `main` or `mobile`.

##### 1.4.2.4. Taxons

`ProductTaxons` are relational entities that assign the `Product` to a given `Taxon`, also referred to as its **
main** `Taxon`.

##### 1.4.2.5. Descriptions

Many ClientName-v1 product decriptions contain static, hard-coded, and/or relative URLs, e.g.:

```html
The pump draws water from the
<a
  title="ClientName Semi-Custom Holding Tanks"
  href="../../en/product-categories/holding-tanks"
  >storage tank(s)</a
>
and fills the water heater tank. Inside the water heater is an electrical
<a
  title="Water Heater Heating Element - PART NUMBER: WH1A"
  href="../../en/shop/products/water-heater-heating-element"
  >heating element</a
>
and usually a coiled tube called a heat exchanger. When AC power is available,
the electrical element (<a
  title="Water Heater Thermostat Part Number: WH16"
  href="../../en/shop/products/water-heater-thermostat"
  >controlled by a thermostat</a
>) heats the water.
```

The import process normalizes descriptions by searching and replacing hard-coded URLs with relative, v2 URLs, e.g.:

```html
The pump draws water from the
<a
  title="ClientName Semi-Custom Holding Tanks"
  href="/en_US/taxons/holding-tanks"
  >storage tank(s)</a
>
and fills the water heater tank. Inside the water heater is an electrical
<a
  title="Water Heater Heating Element - PART NUMBER: WH1A"
  href="/en_US/products/water-heater-heating-element"
  >heating element</a
>
and usually a coiled tube called a heat exchanger. When AC power is available,
the electrical element (<a
  title="Water Heater Thermostat Part Number: WH16"
  href="/en_US/products/water-heater-thermostat"
  >controlled by a thermostat</a
>) heats the water.
```

v1 URLs take two forms, as found in the below queries:

- 444 rows

```sql
SELECT * FROM products where gendescription LIKE '%href="../../%' or pdescription LIKE '%href="../../%'
```

- 255 rows

```sql
SELECT * FROM products where gendescription LIKE '%http://ClientNameeng.com/en/pages%' or pdescription LIKE '%http://ClientNameeng.com/en/pages%'
```

#### 1.4.3. Filtered Data

No data was filtered beyond the base SQL query.

#### 1.4.4. Extra Data

No extra data was added.

#### 1.4.5. Finalized Database Imports

`ProductImporter` creates and touches too many Sylius database tables / records to show here. However, records are added
to the following tables throughout the process:

- `sylius_product`
- `sylius_product_attribute`
- `sylius_product_attribute_translation`
- `sylius_product_attribute_value`
- `sylius_product_channels`
- `sylius_product_image`
- `sylius_product_taxon`
- `sylius_product_translation`

#### 1.4.6. JSON Dump

All generated V2 `Product` data can be found in [exports/product.json](exports/product.json).

### 1.5. `import:product-variant`

Transfers child ClientName-v1 `products` records into Sylius `ProductVariants` along with all necessary related entities:

- `ProductImage`
- `ProductOption`
- `ProductOptionValue`

#### 1.5.1. ClientName-v1 Data Export

All `enabled` products that are _NOT_ `trudesign`-related are exported:

```sql
SELECT
  p.*,
  s1.subcatid subcatid1,
  s1.subcatname subcatname1,
  s1.subcaturl subcaturl1,
  s2.subcatid subcatid2,
  s2.subcatname subcatname2,
  s2.subcaturl subcaturl2,
  s3.subcatid subcatid3,
  s3.subcatname subcatname3,
  s3.subcaturl subcaturl3,
  s4.subcatid subcatid4,
  s4.subcatname subcatname4,
  s4.subcaturl subcaturl4
FROM
  products p
LEFT JOIN
  productsubcategory s1
  ON
    p.psubcategory = s1.subcatid
LEFT JOIN
  productsubcategory s2
  ON
    p.psubcategory3 = s2.subcatid
LEFT JOIN
  productsubcategory s3
  ON
    p.psubcategory4 = s3.subcatid
LEFT JOIN
  productsubcategory s4
  ON
    p.psubcategory5 = s4.subcatid
WHERE
  seourl NOT LIKE '%trudesign%'
AND
  prstatus = 1
ORDER BY
  productnumber ASC
```

#### 1.5.2. Normalization and Mapping

A new `Product` entity is created for each incoming record with a few important mappings:

- `code`: V1 `partnumber` (`sku`)
- `name`: Titlized v1 `productname`
- `price`: V1 `punitprice`, converted from dollars to whole cents
- `slug`: Slugified v1 `seourl`, with prefixed `product_id` _if_ base `seourl` slug already exists

##### 1.5.2.1. Options

`ProductOptions` are generated via an [Option](src/Object/Option.php) instance, based on the existence of mapped
v1 `products` field values. For example, a non-empty v1 `volt` value is converted to a v2 `voltage` `ProductOption` text
entity.

Once the base `ProductOption` is created then `ProductOptionValues` can be created via
an [OptionValue](src/Object/OptionValue.php) instance. There's a subtle but important difference
between `ProductOptions` and `ProductOptionValues`. Each incoming v1 `products` record may indicate a wide range of
possible values within a given option field. Each value is imported as a `ProductOptionValue` with a unique `code`.
Here's a collection of `sylius_product_option_value` records for the `bowl_size` `ProductOption`:

| id  | option_id | code                  |
| --- | --------- | --------------------- |
| 396 | 9         | bowl_size_household   |
| 404 | 9         | bowl_size_elongated   |
| 411 | 9         | bowl_size_marine      |
| 441 | 9         | bowl_size_bidet       |
| 444 | 9         | bowl_size_tall_bowl   |
| 446 | 9         | bowl_size_short_bowl  |
| 467 | 9         | bowl_size_lower_base  |
| 475 | 9         | bowl_size6_gallon     |
| 477 | 9         | bowl_size12_gallon    |
| 478 | 9         | bowl_size20_gallon    |
| 490 | 9         | bowl_size_marine_size |
| 493 | 9         | bowl_size412          |
| 495 | 9         | bowl_size312          |
| 498 | 9         | bowl_size2            |
| 500 | 9         | bowl_size338          |

##### 1.5.2.2. Pricing

Sylius manages the price of a given `ProductVariant` through an assigned `Channel`. At present a
single `DEFAULT` `Channel` exists and contains all imported `ProductVariants`.

##### 1.5.2.3. Product: Water Heater

- V1 `products` table data:

```sql
SELECT
  `partnumber`, `pdescription`, `volt`, `pfitting`, `watersource`, `control`, `prid`
FROM
  `products`
WHERE
  `productnumber` = 343
```

| partnumber | pdescription                                                     | volt | pfitting | watersource       | control         | BowlSize  |
| ---------- | ---------------------------------------------------------------- | ---- | -------- | ----------------- | --------------- | --------- |
| 170601     | <p>WATER HEATER: 6 GAL., W/O HEAT EXHANGER, 120VOLT</p>          | 115v |          | no heat exchanger |                 | 6 gallon  |
| 170602     | <p>WATER HEATER: 6 GAL., W/O HEAT EXCHANGER, 240VOLTS</p>        | 220v |          | no heat exchanger |                 | 6 gallon  |
| 170611     | WATER HEATER: 6 GAL., W/HEAT EXCHANGER, 120VOLTS                 | 115v |          | heat exchanger    |                 | 6 gallon  |
| 170612     | WATER HEATER: 6 GAL., W/HEAT EXCHANGER, 240VOLTS                 | 220v |          | heat exchanger    |                 | 6 gallon  |
| 171201     | WATER HEATER: 12 GAL., W/O HEAT EXCHANGER, 120VOLT               | 115v |          | no heat exchanger |                 | 12 gallon |
| 171202     | WATER HEATER: 12 GAL., W/O HEAT EXCHANGER, 240VOLT               | 220v |          | no heat exchanger |                 | 12 gallon |
| 171211     | WATER HEATER: 12 GAL., W/HEAT EXCHANGER, 120VOLTS                | 115v |          | heat exchanger    |                 | 12 gallon |
| 171212     | WATER HEATER: 12 GAL., W/HEAT EXCHANGER, 240VOLTS                | 220v |          | heat exchanger    |                 | 12 gallon |
| 172002     | WATER HEATER: 20 GAL., W/O HEAT EXCHANGER, 240VOLT               | 220v |          | no heat exchanger |                 | 20 gallon |
| 172012     | WATER HEATER: 20 GAL., W/HEAT EXCHANGER, 240VOLTS                | 220v |          | heat exchanger    |                 | 20 gallon |
| 17061201   | WATER HEATER:_CUSTOM_ 170612 W/PL2A-75GW (GAZ SET)               | 115v | GAZ      | heat exchanger    | GAZ fittings    | 6 gallon  |
| 17061202   | WATER HEATER: _CUSTOM_ 170612 W/PL2A75MW (SET)                   | 220v | GAZ      | heat exchanger    | GAZ fittings    | 6 gallon  |
| 17120203   | WATER HEATER: 12 GALLON, W/OUT HEAT EXCHANGER, 4500 WATT/240VOLT | 220v |          | no heat exchanger |                 | 12 gallon |
| 17121101   | HEATER: 171211 WITH PL2A75GW ( GAZ SET) 120 VOLT                 | 115v | GAZ      | heat exchanger    | GAZ fittings    | 12 gallon |
| 17121102   | WATER HEATER: CUSTOM, 171211 W/PL2A75MW (METRIC SET)             | 115v | METRIC   | heat exchanger    | METRIC fittings | 12 gallon |
| 17121201   | WATER HEATER: _CUSTOM_ 171212 W/PL2A75GW (GAZ SET)               | 220v | GAZ      | heat exchanger    | GAZ fittings    | 12 gallon |
| 17121202   | WATER HEATER:_CUSTOM_ 171212 W/PL2A75MW (SET)                    | 220v | METRIC   | heat exchanger    | METRIC fittings | 12 gallon |
| 17121203   | WATER HEATER: 12 GAL.,4500 WATT,W/HEAT EXCHANGER, 240 VOLT       | 220v |          | heat exchanger    |                 |           |
| 17200104   | WATER HEATER: 20 GAL , W/O HEAT EXCHANGES, 120VOLT               | 115v |          | no heat exchanger |                 |           |
| 17200203   | WATER HEATER: _CUSTOM_ 20 GAL.,W/O H.E., 45OOWATTS               | 4500 |          | no heat exchanger |                 |           |
| 17201104   | WATER HEATER 20 GAL. W/ HEAT EXCHANGER 120V                      | 115v |          | heat exchanger    |                 |           |
| 17201201   | WATER HEATER:_CUSTOM_ 172012 W/PL2A-75GW (GAZ SET)               |      |          | heat exchanger    |                 |           |
| 17201202   | WATER HEATER: _CUSTOM_ 172012 W/PL2A75MW (SET)                   |      |          |                   |                 |           |
| 17201203   | WATER HEATER: 20 GAL. 4500 WATT W/ HEAT EXCHANGER 240 VOLT       | 220v |          | heat exchanger    |                 |           |

- Partnumber/SKU mappings:

| partnumber | code_size | code_exchanger | code_power | code_fittings | actual_size | actual_exchanger    | actual_power | actual_fittings        |
| ---------- | --------- | -------------- | ---------- | ------------- | ----------- | ------------------- | ------------ | ---------------------- |
| 170601     | 06        | 0              | 1          |               | 6 gallon    | no heat exchanger   | 120V AC      | N/A                    |
| 170602     | 06        | 0              | 2          |               | 6 gallon    | no heat exchanger   | 220V AC      | N/A                    |
| 170611     | 06        | 1              | 1          |               | 6 gallon    | with heat exchanger | 120V AC      | N/A                    |
| 170612     | 06        | 1              | 2          |               | 6 gallon    | with heat exchanger | 220V AC      | N/A                    |
| 171201     | 12        | 0              | 1          |               | 12 gallon   | no heat exchanger   | 120V AC      | N/A                    |
| 171202     | 12        | 0              | 2          |               | 12 gallon   | no heat exchanger   | 220V AC      | N/A                    |
| 171211     | 12        | 1              | 1          |               | 12 gallon   | with heat exchanger | 120V AC      | N/A                    |
| 171212     | 12        | 1              | 2          |               | 12 gallon   | with heat exchanger | 220V AC      | N/A                    |
| 172002     | 20        | 0              | 2          |               | 20 gallon   | no heat exchanger   | 220V AC      | N/A                    |
| 172012     | 20        | 1              | 2          |               | 20 gallon   | with heat exchanger | 220V AC      | N/A                    |
| 17061201   | 06        | 1              | 2          | 01            | 6 gallon    | with heat exchanger | 220V AC      | GAZ                    |
| 17061202   | 06        | 1              | 2          | 02            | 6 gallon    | with heat exchanger | 220V AC      | Metric                 |
| 17120203   | 12        | 0              | 2          | 03            | 12 gallon   | no heat exchanger   | 220V AC      | 4500 W Heating Element |
| 17121101   | 12        | 1              | 1          | 01            | 12 gallon   | with heat exchanger | 120V AC      | GAZ                    |
| 17121102   | 12        | 1              | 1          | 02            | 12 gallon   | with heat exchanger | 120V AC      | Metric                 |
| 17121201   | 12        | 1              | 2          | 01            | 12 gallon   | with heat exchanger | 220V AC      | GAZ                    |
| 17121202   | 12        | 1              | 2          | 02            | 12 gallon   | with heat exchanger | 220V AC      | Metric                 |
| 17121203   | 12        | 1              | 2          | 03            | 12 gallon   | with heat exchanger | 220V AC      | 4500 W Heating Element |
| 17200104   | 20        | 0              | 1          | 04            | 20 gallon   | no heat exchanger   | 120V AC      | ???                    |
| 17200203   | 20        | 0              | 2          | 03            | 20 gallon   | no heat exchanger   | 220V AC      | 4500 W Heating Element |
| 17201104   | 20        | 1              | 1          | 04            | 20 gallon   | with heat exchanger | 120V AC      | ???                    |
| 17201201   | 20        | 1              | 2          | 01            | 20 gallon   | with heat exchanger | 220V AC      | GAZ                    |
| 17201202   | 20        | 1              | 2          | 02            | 20 gallon   | with heat exchanger | 220V AC      | Metric                 |
| 17201203   | 20        | 1              | 2          | 03            | 20 gallon   | with heat exchanger | 220V AC      | 4500 W Heating Element |

- Actual based on promo sheet SKU codes:

| partnumber | code_size | code_exchanger | code_power | code_fittings | actual_size | actual_exchanger    | actual_power | actual_fittings        | pdescription                                                     | volt | pfitting | watersource       | control         | BowlSize  |
| ---------- | --------- | -------------- | ---------- | ------------- | ----------- | ------------------- | ------------ | ---------------------- | ---------------------------------------------------------------- | ---- | -------- | ----------------- | --------------- | --------- |
| 170601     | 06        | 0              | 1          |               | 6 gallon    | no heat exchanger   | 120V AC      | N/A                    | WATER HEATER: 6 GAL., W/O HEAT EXHANGER, 120VOLT                 | 115v |          | no heat exchanger |                 | 6 gallon  |
| 170602     | 06        | 0              | 2          |               | 6 gallon    | no heat exchanger   | 220V AC      | N/A                    | WATER HEATER: 6 GAL., W/O HEAT EXCHANGER, 240VOLTS               | 220v |          | no heat exchanger |                 | 6 gallon  |
| 170611     | 06        | 1              | 1          |               | 6 gallon    | with heat exchanger | 120V AC      | N/A                    | WATER HEATER: 6 GAL., W/HEAT EXCHANGER, 120VOLTS                 | 115v |          | heat exchanger    |                 | 6 gallon  |
| 170612     | 06        | 1              | 2          |               | 6 gallon    | with heat exchanger | 220V AC      | N/A                    | WATER HEATER: 6 GAL., W/HEAT EXCHANGER, 240VOLTS                 | 220v |          | heat exchanger    |                 | 6 gallon  |
| 171201     | 12        | 0              | 1          |               | 12 gallon   | no heat exchanger   | 120V AC      | N/A                    | WATER HEATER: 12 GAL., W/O HEAT EXCHANGER, 120VOLT               | 115v |          | no heat exchanger |                 | 12 gallon |
| 171202     | 12        | 0              | 2          |               | 12 gallon   | no heat exchanger   | 220V AC      | N/A                    | WATER HEATER: 12 GAL., W/O HEAT EXCHANGER, 240VOLT               | 220v |          | no heat exchanger |                 | 12 gallon |
| 171211     | 12        | 1              | 1          |               | 12 gallon   | with heat exchanger | 120V AC      | N/A                    | WATER HEATER: 12 GAL., W/HEAT EXCHANGER, 120VOLTS                | 115v |          | heat exchanger    |                 | 12 gallon |
| 171212     | 12        | 1              | 2          |               | 12 gallon   | with heat exchanger | 220V AC      | N/A                    | WATER HEATER: 12 GAL., W/HEAT EXCHANGER, 240VOLTS                | 220v |          | heat exchanger    |                 | 12 gallon |
| 172002     | 20        | 0              | 2          |               | 20 gallon   | no heat exchanger   | 220V AC      | N/A                    | WATER HEATER: 20 GAL., W/O HEAT EXCHANGER, 240VOLT               | 220v |          | no heat exchanger |                 | 20 gallon |
| 172012     | 20        | 1              | 2          |               | 20 gallon   | with heat exchanger | 220V AC      | N/A                    | WATER HEATER: 20 GAL., W/HEAT EXCHANGER, 240VOLTS                | 220v |          | heat exchanger    |                 | 20 gallon |
| 17061201   | 06        | 1              | 2          | 01            | 6 gallon    | with heat exchanger | 220V AC      | GAZ                    | WATER HEATER:CUSTOM 170612 W/PL2A-75GW (GAZ SET)                 | 115v | GAZ      | heat exchanger    | GAZ fittings    | 6 gallon  |
| 17061202   | 06        | 1              | 2          | 02            | 6 gallon    | with heat exchanger | 220V AC      | Metric                 | WATER HEATER: CUSTOM 170612 W/PL2A75MW (SET)                     | 220v | GAZ      | heat exchanger    | GAZ fittings    | 6 gallon  |
| 17120203   | 12        | 0              | 2          | 03            | 12 gallon   | no heat exchanger   | 220V AC      | 4500 W Heating Element | WATER HEATER: 12 GALLON, W/OUT HEAT EXCHANGER, 4500 WATT/240VOLT | 220v |          | no heat exchanger |                 | 12 gallon |
| 17121101   | 12        | 1              | 1          | 01            | 12 gallon   | with heat exchanger | 120V AC      | GAZ                    | HEATER: 171211 WITH PL2A75GW ( GAZ SET) 120 VOLT                 | 115v | GAZ      | heat exchanger    | GAZ fittings    | 12 gallon |
| 17121102   | 12        | 1              | 1          | 02            | 12 gallon   | with heat exchanger | 120V AC      | Metric                 | WATER HEATER: CUSTOM, 171211 W/PL2A75MW (METRIC SET)             | 115v | METRIC   | heat exchanger    | METRIC fittings | 12 gallon |
| 17121201   | 12        | 1              | 2          | 01            | 12 gallon   | with heat exchanger | 220V AC      | GAZ                    | WATER HEATER: CUSTOM 171212 W/PL2A75GW (GAZ SET)                 | 220v | GAZ      | heat exchanger    | GAZ fittings    | 12 gallon |
| 17121202   | 12        | 1              | 2          | 02            | 12 gallon   | with heat exchanger | 220V AC      | Metric                 | WATER HEATER:CUSTOM 171212 W/PL2A75MW (SET)                      | 220v | METRIC   | heat exchanger    | METRIC fittings | 12 gallon |
| 17121203   | 12        | 1              | 2          | 03            | 12 gallon   | with heat exchanger | 220V AC      | 4500 W Heating Element | WATER HEATER: 12 GAL.,4500 WATT,W/HEAT EXCHANGER, 240 VOLT       | 220v |          | heat exchanger    |                 |           |
| 17200104   | 20        | 0              | 1          | 04            | 20 gallon   | no heat exchanger   | 120V AC      | ???                    | WATER HEATER: 20 GAL , W/O HEAT EXCHANGES, 120VOLT               | 115v |          | no heat exchanger |                 |           |
| 17200203   | 20        | 0              | 2          | 03            | 20 gallon   | no heat exchanger   | 220V AC      | 4500 W Heating Element | WATER HEATER: CUSTOM 20 GAL.,W/O H.E., 45OOWATTS                 | 4500 |          | no heat exchanger |                 |           |
| 17201104   | 20        | 1              | 1          | 04            | 20 gallon   | with heat exchanger | 120V AC      | ???                    | WATER HEATER 20 GAL. W/ HEAT EXCHANGER 120V                      | 115v |          | heat exchanger    |                 |           |
| 17201201   | 20        | 1              | 2          | 01            | 20 gallon   | with heat exchanger | 220V AC      | GAZ                    | WATER HEATER:CUSTOM 172012 W/PL2A-75GW (GAZ SET)                 |      |          | heat exchanger    |                 |           |
| 17201202   | 20        | 1              | 2          | 02            | 20 gallon   | with heat exchanger | 220V AC      | Metric                 | WATER HEATER: CUSTOM 172012 W/PL2A75MW (SET)                     |      |          |                   |                 |           |
| 17201203   | 20        | 1              | 2          | 03            | 20 gallon   | with heat exchanger | 220V AC      | 4500 W Heating Element | WATER HEATER: 20 GAL. 4500 WATT W/ HEAT EXCHANGER 240 VOLT       | 220v |          | heat exchanger    |                 |           |

#### 1.5.3. Filtered Data

The ClientName-v1 `products` data set contains a handful of duplicate `sku` (`partnumber`) records:

```sql
WITH T AS
  (
    SELECT *, COUNT(*)
    OVER (
      PARTITION BY partnumber
    ) AS C
    FROM
      products
  )
SELECT
  prid,
  productnumber,
  partnumber,
  productname,
  prstatus
FROM
  T
WHERE
  C > 1
ORDER BY partnumber ASC
```

| prid | productnumber | partnumber | productname                                      | prstatus | Status  |
| ---- | ------------- | ---------- | ------------------------------------------------ | -------- | ------- |
| 1287 | 424           | 221105     | O-Ring                                           | 1        | Ignored |
| 913  | 425           | 221105     | MARINE ELEGANCE: DISCHARGE O-RING                | 1        |         |
| 915  | 427           | 221115     | MARINE ELEGANCE DISCHARGE PUMP MACERATOR HOUSING | 1        |         |
| 1288 | 386           | 221115     | Discharge Pump                                   | 0        | Ignored |
| 1298 | 431           | 221351     | Water Solenoid                                   | 1        | Ignored |
| 918  | 430           | 221351     | Marine Elegance Water Solenoid 12v               | 1        |         |
| 919  | 433           | 221352     | Marine Elegance Water Solenoid 24v               | 1        |         |
| 1299 | 432           | 221352     | Water Solenoid                                   | 1        | Ignored |
| 1300 | 434           | 221353     | Water Solenoid                                   | 1        | Ignored |
| 920  | 435           | 221353     | Marine Elegance WATER SOLENOID, D90 12 VDC       | 0        |         |
| 1113 | 123           | I34B       | Icerette Water Valve                             | 1        |         |
| 1308 | 397           | I34B       | Water Valve                                      | 1        | Ignored |
| 1116 | 115           | I34M220    | Icerette Module VAC                              | 1        |         |
| 1309 | 398           | I34M220    | Icemaker Module                                  | 1        | Ignored |
| 823  | 217           | SFH10      | Sani/Flex Odor Shield Marine Hose                | 1        |         |
| 1318 | 414           | SFH10      | Sani/Flex Odor Shield Marine Hose Length         | 1        | Ignored |
| 1316 | 414           | SFH150     | Sani/Flex Odor Shield Marine Hose Length         | 1        | Ignored |
| 824  | 217           | SFH150     | Sani/Flex Odor Shield Marine Hose                | 1        |         |
| 1317 | 414           | SFH1C      | Sani/Flex Odor Shield Marine Hose Length         | 1        | Ignored |
| 825  | 217           | SFH1C      | Sani/Flex Odor Shield Marine Hose                | 1        |         |
| 826  | 217           | SFH50      | Sani/Flex Odor Shield Marine Hose                | 1        |         |
| 1319 | 414           | SFH50      | Sani/Flex Odor Shield Marine Hose Length         | 1        | Ignored |
| 1320 | 414           | SFHC       | Sani/Flex Odor Shield Marine Hose Length         | 1        | Ignored |
| 827  | 217           | SFHC       | Sani/Flex Odor Shield Marine Hose                | 1        |         |

To resolve these duplicates all `Status: Ignored` records listed above were _NOT_ imported into the ClientName-v2 data.

#### 1.5.4. Extra Data

No extra data was added.

#### 1.5.5. Finalized Database Imports

Similar to `ProductImporter` the `ProductVariantImporter` adds records to the following tables:

- `sylius_product_channels`
- `sylius_product_option`
- `sylius_product_option_translation`
- `sylius_product_option_value`
- `sylius_product_option_value_translation`
- `sylius_product_options`
- `sylius_product_taxon`
- `sylius_product_variant`
- `sylius_product_variant_option_value`
- `sylius_product_variant_translation`

#### 1.5.6. JSON Dump

All generated V2 `ProductVariant` data can be found in [exports/product-variant.json](exports/product-variant.json).

### 1.6. `import:dealer`

Transfers all top-level ClientName-v1 `dealers` records into Sylius `Dealers`.

#### 1.6.1. Prerequisite

- Enabled patching for Composer: `composer require cweagans/composer-patches`
- Add `force-annotation` option to entity
  maker: https://github.com/vklux/maker-bundle-force-annotation/blob/master/maker-force-annotation-flag.patch
- Ensure `DealerRepository` is auto-wired in `services.yaml`

#### 1.6.2. Entity Fields

| name          | type                    | new_name    |
| ------------- | ----------------------- | ----------- |
| dealername    | varchar(200)            | name        |
| dealerurl     | varchar(500)            | slug        |
| daddress1     | varchar(100) NULL       | address1    |
| daddress2     | varchar(100) NULL       | address2    |
| dcity         | varchar(100) NULL       | city        |
| dzipcode      | varchar(20) NULL        | postal_code |
| dstate        | varchar(100) NULL       | state       |
| dcountry      | varchar(2) NULL [US]    | country     |
| dphone        | varchar(50) NULL        | phone       |
| dfax          | varchar(50) NULL        | fax         |
| demail        | varchar(200) NULL       | email       |
| dwebsite      | varchar(500) NULL       | url         |
| dstatus       | int [0]                 | enabled     |
| dlogo         | varchar(500) NULL       |             |
| dealertype    | int NULL                | type        |
| wholesaleonly | int NULL [0]            | wholesale   |
| dcomment      | text NULL               | comment     |
| gmaplat       | varchar(20) NULL [0.00] | latitude    |
| gmaplong      | varchar(20) NULL [0.00] | longitude   |

#### 1.6.3. ClientName-v1 Data Export

All dealers are exported:

```sql
SELECT
  *
FROM
  dealers
```

#### 1.6.4. Normalization and Mapping

A new `Dealer` entity is created for each incoming record with a few important mappings:

- Most V1 columns are prefixed with the letter `d`, which has been removed. Names have also been normalized where
  appropriate.
- `slug`: V1 `dealerurl`. The V1 data is actually a partial URL slug, so mapping adjusted accordingly
- `url`: V1 `dwebsite`. This is the real URL of the dealer's website, if applicable.
- `country`: Uppercased.
- `enabled`: V1 `dstatus`. Booleanized.
- `wholesale`: V1 `wholesaleonly`. Booleanized.
- `type`: V1 `dealertype`. Integer value, which is mapped to appropriate string in `DealerEntity` and `DealerController`
  .

#### 1.6.5. Filtered Data

No data was filtered beyond the base SQL query.

#### 1.6.6. Extra Data

No extra data was added.

#### 1.6.7. Finalized Database Imports

Other than data normalization and mapping the V2 `Dealers` table identically matches the V1 `dealers` table.

#### 1.6.8. JSON Dump

All generated V2 `Dealer` data can be found in [exports/dealer.json](exports/dealer.json).

### 1.7. `import:zone`

Imports all taxable zones and tax rates for US states.

#### 1.7.1. Data Source

By default, province data is imported from [seed/province-tax-nj-only.json](seed/province-tax-nj-only.json), which
contains New Jersey tax rate data **only**.

Alternatively, a new import data source can be specified as the first argument to the `import:zone` command.

For example, import **all** US state tax rate data via:

```bash
$ php bin/console import:zone seed/province.json
```

#### 1.7.2. Normalization and Mapping

A new `Province` entity is created for each incoming record:

- `code`: Format of `US-XX` where `XX` is the state abbreviation.

Each `Province` entity is used to generate a new `Zone`:

- `code`: Format of `ZONE-US-XX`.
- `name`: Format of `US - <State Name>`
- `scope`: Set to `tax` to only apply to tax rates.
- `type`: Set to `province`.
- `member`: Associated with the appropriate `Province` entity so checkout tax calculator can identify when an address
  applies to a given zone.

Each `Zone` entity is used to generate a new `TaxRate`:

- `code`: Format of `US-XX`.
- `amount`: Obtained from original `Province` JSON data object.

#### 1.7.3. Filtered Data

No data was filtered.

#### 1.7.4. Extra Data

No extra data was added.

#### 1.7.5. Enabling/Modifying Tax Rates

Sylius does not have a admin-editable way to disable a given State for tax calculation purposes. Thus, the next best
option was implemented:

1. Each state has an existing `Zone` and applicable `TaxRate` entity.
2. By default per ClientName-v1 business logic, all `TaxRate.amount` values are set to `0%` with the exception of New
   Jersey.
3. Thus, checkout for addresses outside of NJ effectively have no tax applied.

## 2. Shipping

Most shipping logic including FedEx rate requests are handled via
the [Solarix Shipping - PHP](https://gitlab.solarixdigital.com/solarix/core/tooling/solarix-shipping-php) library. This
section covers integration into Sylius including pitfalls and implemented workarounds.

### 2.1. Shipping Methods

Sylius handles shipping logic via one or more **Shipping Methods**. A Shipping Method contains a few fundemental
properties:

- `code`: A unique string identifier, e.g `FEDEX_EXPRESS_SAVER`
- `configuration`: An object containing specific configuration,
  e.g. `a:2:{s:7:"minimum";i:1000;s:4:"code";s:19:"FEDEX_EXPRESS_SAVER";}`
- `calculator`: The **Shipping Calculator** class used to calculate shipping rates for this method.

### 2.2. Shipping Calculator

The custom [FedExRateCalculator](src/Shipping/Calculator/FedExRateCalculator.php) handles all shipping rate retrieval
from the FedEx provider API using
the [Solarix Shipping - PHP](https://gitlab.solarixdigital.com/solarix/core/tooling/solarix-shipping-php) package. A
handful of Sylius design choices and limitations required some workarounds.

### 2.3. Issue: Handling Multiple Rates

Sylius does not support associated multiple rates with a single **Shipping Method**. Instead, Sylius assumes rate data
will be obtained solely from the defined `configuration` property set within the Admin panel.

The result is that all shipping related UIs including the checkout shipment method page cannot display more than one
shipping method and therefore only a single rate value. Our use case requires multiple rates depending on what the FedEx
API returns.

The implemented solution was to add a new shipping method for each of the potential types returned by the FedEx rates
request API, e.g.:

- FEDEX_EXPRESS_SAVER
- FEDEX_FIRST_OVERNIGHT
- FEDEX_PRIORITY_OVERNIGHT
- ... etc.

### 2.4. Issue: Provider API Request Throttling

Each active **Shipping Method** invokes the underlying rate calculator logic during processing, which means a potential
for half a dozen or more unnecessary provider API requests.

To resolve this a simple local cache was added to keep track of recently obtained rates data for a given object. The
new `Order->rates` property stores a serialized collection
of [Rates](https://gitlab.solarixdigital.com/solarix/core/tooling/solarix-shipping-php/-/blob/master/src/Solarix/Shipping/Provider/FedEx/Model/Rate/Rate.php)
. When the rates calculator is invoked it checks the Order's rates cache for last updated timestamp. If updated within a
short period of time the cache is returned rather than making a new FedEx provider API request.

### 2.5. Issue: Orders w/ Invalid Shipping Methods

Checkout should not proceed past "Select Shipping Method" if the selected shipping method has a zero-value base charge. This can be caused by two different scenarios:

- The shipping service API is down/fails to respond
- Some of the available `ShippingMethods` provided via the Sylius admin dashboard **are not** valid for the requested `Shipment`, which causes the `Rate` to remain at the default `0` base charge value.

The first issue is resolved by adding an event listener that compares the selected `ShippingMethod` value against the API-provided rates data. If the API failed to respond or provided a zero-value base charge for the selected `ShippingMethod`, the listener halts execution, returns to the select shipping page, and displays an error to the user.

The second issue is resolved by adding rates data inline in the DOM when the `selectShipping.html.twig` template is rendered. Any shipping method div with an invalid `data-fee` attribute is hidden from view. Further, if there are **no** valid shipping methods available then the proceed button is disabled and an error message is displayed to the user informing them to contact ClientName for further assistance.

Coupons that reduce shipping charges to zero (i.e. 100% off) are unaffected by these adjustments and still function normally.

### Issue: Expanded International Shipping

The following countries are valid shipping destinations:

| Name                     | Code |
| ------------------------ | ---- |
| Australia                | AU   |
| Canada                   | CA   |
| Denmark                  | DK   |
| France                   | FR   |
| Japan                    | JP   |
| Mexico                   | MX   |
| Netherlands              | NL   |
| Panama                   | PA   |
| Puerto Rico              | PR   |
| Singapore                | SG   |
| South Korea              | KR   |
| Sweden                   | SE   |
| Turkey                   | TR   |
| United Arab Emirates     | UA   |
| United Kingdom           | GB   |
| United States            | US   |
| Virgin Islands (British) | VG   |
| Virgin Islands (U.S.)    | VI   |

FedEx Serviced Countries: https://www.fedex.com/content/dam/fedex/us-united-states/services/CountriesServed_IF_IP_IE.pdf

---

- [x] Add new `FEDEX_INTERNATIONAL_FIRST` shipping method
- [x] Disable unused countries in **Admin > Countries**
- [x] Add additional countries to **Admin > Countries**
- [x] Add `Canada` and `Mexico` to **Admin > Zones > `North America`** `Zone`
- [x] Add other countries to **Admin > Zones > `World`** `Zone`
- [x] Test address from each country:

| First Name          | Last Name         | Street Address                                         | State    | City             | Zipcode  |
| ------------------- | ----------------- | ------------------------------------------------------ | -------- | ---------------- | -------- |
| Red                 | House             | 37272 Dawson Rd                                        | BC       | Abbotsford       | V3G 2K9  |
| Copenhagen          | OperaHouse        | Ekvipagemestervej 10                                   |          | København        | 1438     |
| Musée               | du Louvre         | Rue de Rivoli                                          |          | Paris            | 75001    |
| Tokyo Medical       | Dental University | 1 Chome-5-45 Yushima                                   | Tokyo    | Bunkyo City      | 113-8510 |
| Palacio             | de Bellas Artes   | Av. Juárez S/N, Centro Histórico de la Cdad. de México |          | Ciudad de México | 06050    |
| Informatica         | Nederland B.V.    | Edisonbaan, 14A                                        |          | NIEUWEGEIN       | 3439 MN  |
| Banco Nacional      | de Panama         | Av 17 de Abril                                         |          | Changuinola      | 0101     |
| Puertorican         | Apartment         | 1234 Ave Ashford, Apt 1A                               | PR       | San Juan         | 00907    |
| Singapore Post      | Pte Ltd           | 10 Eunos Road 8                                        |          | Singapore        | 408600   |
| 홍길동              | 귀하              | Sajik-ro-3-gil 23, Bldg. 102 Unit 304                  | Seoul    | Jongno-gu        | 30174    |
| Royal               | Palace            | Kungliga slottet                                       |          | Stockholm        | 107 70   |
| MEHMET              | DEMIR             | AKŞEMSETTİN MAH., GÖKHAN SOK. NO. 1                    | ANKARA   | SİNCAN           | 06934    |
| Burj                | Khalifa           | 1 Sheikh Mohammed bin Rashid Blvd                      |          | Dubai            | 00000    |
| Rosewood            | London            | 252 High Holborn                                       |          | London           | WC1V 7EN |
| BritishVirginIsland | Person            | UPS Cineplex BVI                                       |          | Road Town        | VG1110   |
| Supersave           | Market            | 39-40 Concordia Rd                                     | St Croix | Frederiksted     | 00840    |

## 3. Static Page Imports

| ID  | Title                                           | Category          | Status   | Note                                                   |
| --- | ----------------------------------------------- | ----------------- | -------- | ------------------------------------------------------ |
| 1   | About Us                                        | Information       | Imported |                                                        |
| 2   | Terms of Service                                | Legal             | Imported |                                                        |
| 3   | Privacy Policy                                  | Legal             | Imported |                                                        |
| 4   | Digital Millennium Copyright Act Notice         | Legal             | Imported |                                                        |
| 5   | Anti Spam Policy                                | Legal             | Imported |                                                        |
| 6   | Refund Policy                                   | Legal             | Imported | Placeholder content requires client update             |
| 7   | Earnings Disclaimer                             | Legal             | IGNORED  | Unused in ClientName-v1                                |
| 10  | Affiliate Disclosure                            | Legal             | IGNORED  | Unused in ClientName-v1                                |
| 11  | Repair Shop                                     | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 12  | Lectra\|San                                     | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 13  | What the flashing lights mean                   | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 14  | Troubleshooting the “Red light shut down”       | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 15  | Do I need a salt feed unit?                     | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 16  | Control board replacement                       | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 17  | Atlantes A5, A6                                 | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 18  | Atlastes Intake                                 | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 19  | Atlantes Discharge                              | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 20  | Atlantes Control Board                          | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 21  | Fuse Relay                                      | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 22  | Crown Head (CD Series)                          | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 23  | Replacing the discharge pump wall (part #CH120) | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 24  | Servicing the siphon check valve (part #CH37E)  | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 25  | Servicing the intake pump assembly (part G13W)  | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 26  | PHII and PHEII                                  | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 27  | Icerette                                        | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 28  | Water Heater (1700 Series)                      | Repair Shop       | Imported | Fixed links, images, and pdfs                          |
| 29  | Need A Part?                                    | Technical Support | Imported | Fixed links                                            |
| 30  | LIMITED WARRANTY                                | Technical Support | Imported |                                                        |
| 31  | Identify Your Head                              | Technical Support | Imported | Added images, fixed links                              |
| 32  | Owner’s Manual / Exploded Parts View / Promos   | Technical Support | Imported | Added images, pdfs, and fixed links                    |
| 33  | FAQ                                             | Technical Support | Imported | Fixed formatting and links                             |
| 34  | Shop                                            |                   | IGNORED  | Unused in ClientName-v1                                |
| 35  | Ruddermaster                                    | Landing Pages     | IGNORED  | Unused in ClientName-v1                                |
| 36  | SeaFresh                                        |                   | IGNORED  | Unused in ClientName-v1                                |
| 37  | Not Available At The Moment                     | Information       | IGNORED  | Unused in ClientName-v1                                |
| 38  | How To & Informational Videos                   | Technical Support | Imported | Fixed video embeds, added collapse transcript elements |

## 4. Dealers

Dealer data is managed via the Administration dashboard

### 4.1. Nearby Dealers (Geolocation)

The V1 nearby dealers list indicated it was displaying nearby dealers, but instead merely showed a list of dealers
within the visitor's state lines:

```php
foreach($Dealers as $ds) {
  if(strtoupper($VisitingLocation->geoplugin_regionCode)== strtoupper($ds->dstate)) {
    // ... output local dealer list
  }
}
```

The V2 dealers list resolves this by accurately calculating the distance from the user's IP address geolocation. Dealers
in this nearby list are within `250` miles by default, though this value can be easily adjusted.

This ensures the list is _actually_ showing nearby dealers and the dealers are sorted by distance to the user.

### 4.2. Frontend Dealers Pages

The new V2 Dealers frontend pages closely mimic the V1 look and behavior.

- `/dealers/` shows the same global dealers google map, with local dealer listings immediately below, then the full list
  at the bottom.
- Each individual dealer page is located at `/dealers/{slug}/{id}` to use existing slugs while also allowing for proper
  SEO in the few cases of duplication.
  - Individual dealer pages contain a pinned Google map followed by a card listing dealer details.
  - The layout improves on V1 by hiding unpopulated fields.

### 4.3. Administration

- Visit `/admin/dealers/` section under `Miscellaneous > Dealers` admin dashboard

## 5. Files

- id
- owner_id
- enabled
- type - File type
- path - path to file
- created_by - user_id of uploader
- size - Size, in bytes
- created_at
- updated_at
- checksum - SHA256 hash of file. Checked against upload replacement file to alter `updated_at`.
  See [hash_file](https://www.php.net/manual/en/function.hash-file.php).
- version - Auto-incremented on new upload
