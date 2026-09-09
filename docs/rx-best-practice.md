# RX Object Page Framework - Mini Specification

**Author:** ChatGPT (GPT-5.6 Sol)  
**Date:** 30 August 2026

## Abstract

This article specifies a compact object-oriented PHP framework for administrative pages backed by PostgreSQL. It defines the architecture, naming rules, widget model, page composition API, stylesheet organization, paged device lists, record editing, database access, validation, and acceptance criteria required to reproduce the RX Object Page solution developed in the accompanying example implementation.

## Contents

- [1 Purpose](#1-purpose)
- [2 Normative language](#2-normative-language)
- [3 General design principles](#3-general-design-principles)
  - [3.1 Separation of responsibilities](#31-separation-of-responsibilities)
  - [3.2 Application pages shall be simple](#32-application-pages-shall-be-simple)
- [4 Naming requirements](#4-naming-requirements)
  - [4.1 Filesystem names](#41-filesystem-names)
  - [4.2 PHP identifiers](#42-php-identifiers)
- [5 Directory structure](#5-directory-structure)
- [6 Widget model](#6-widget-model)
  - [6.1 Widget interface](#61-widget-interface)
  - [6.2 Generic container](#62-generic-container)
  - [6.3 RX page](#63-rx-page)
- [7 Stylesheet architecture](#7-stylesheet-architecture)
  - [7.1 Loading order](#71-loading-order)
  - [7.2 CSS responsibilities](#72-css-responsibilities)
    - [7.2.1 rx-base.css](#721-rx-basecss)
    - [7.2.2 rx-page.css](#722-rx-pagecss)
    - [7.2.3 rx-menu.css](#723-rx-menucss)
    - [7.2.4 rx-form.css](#724-rx-formcss)
    - [7.2.5 rx-list.css](#725-rx-listcss)
    - [7.2.6 rx-footer.css](#726-rx-footercss)
- [8 Menu model](#8-menu-model)
  - [8.1 Domain menu](#81-domain-menu)
  - [8.2 Object menu](#82-object-menu)
- [9 Device list](#9-device-list)
  - [9.1 Public list page](#91-public-list-page)
  - [9.2 List interface](#92-list-interface)
  - [9.3 Paging semantics](#93-paging-semantics)
  - [9.4 List database access](#94-list-database-access)
  - [9.5 Count query](#95-count-query)
  - [9.6 Paging query](#96-paging-query)
  - [9.7 Pager](#97-pager)
  - [9.8 Clickable rows](#98-clickable-rows)
- [10 Device record access](#10-device-record-access)
  - [10.1 Record class](#101-record-class)
  - [10.2 Fetching one device](#102-fetching-one-device)
  - [10.3 UUID validation](#103-uuid-validation)
- [11 Device edit form](#11-device-edit-form)
  - [11.1 Form separation](#111-form-separation)
  - [11.2 Form contents](#112-form-contents)
  - [11.3 Hidden edit fields](#113-hidden-edit-fields)
  - [11.4 Form submission](#114-form-submission)
- [12 Device update](#12-device-update)
  - [12.1 Update operation](#121-update-operation)
  - [12.2 Update validation](#122-update-validation)
  - [12.3 Post/Redirect/Get](#123-postredirectget)
  - [12.4 Reload after update](#124-reload-after-update)
  - [12.5 Error behaviour](#125-error-behaviour)
- [13 PostgreSQL integration](#13-postgresql-integration)
  - [13.1 Connection management](#131-connection-management)
  - [13.2 Configuration](#132-configuration)
  - [13.3 Database security](#133-database-security)
- [14 Output safety](#14-output-safety)
  - [14.1 HTML escaping](#141-html-escaping)
- [15 Responsibility summary](#15-responsibility-summary)
- [16 Object-oriented page composition](#16-object-oriented-page-composition)
- [17 Reuse requirements](#17-reuse-requirements)
- [18 Recommended application-page size](#18-recommended-application-page-size)
- [19 Page-state preservation](#19-page-state-preservation)
- [20 Functional acceptance test](#20-functional-acceptance-test)
- [21 Structural acceptance test](#21-structural-acceptance-test)
- [22 Core architectural rule](#22-core-architectural-rule)

## 1 Purpose

The RX page framework shall provide a small object-oriented PHP framework for building administrative pages for PostgreSQL-backed objects such as devices, sites, companies, and users.

The framework shall separate:

1. page composition;
2. reusable widgets;
3. object-specific forms;
4. object-specific lists;
5. database record access; and
6. styling.

A normal application page shall contain as little implementation logic as possible. The intended programming style is declarative:

```php
$page = new rx_page('Device list');

$page->add(new page_header('rScada', 'Device list'));
$page->add(new rx_domain_menu('device'));
$page->add(new rx_object_menu('device', 'list'));

$page->add(
    device_list::create(
        $start_row,
        $number_of_rows
    )
);

$page->add(
    new page_footer('rScada')
);

echo $page->render();
```

The page shall therefore describe what the page contains, not how each component is implemented.

## 2 Normative language

The words shall, should, and may are used deliberately. Shall denotes a requirement for conformance to this specification. Should denotes a recommended best practice that may be departed from for a documented reason. May denotes an optional feature or implementation choice.

## 3 General design principles

### 3.1 Separation of responsibilities

Each class shall have one primary responsibility. For the device implementation, the responsibilities are divided as follows:

```php
public/device-list.php
    page/request handling and composition

device/device-list.php
    device list and paged list database query

public/device-edit.php
    edit request handling and page composition

device/device-edit-form.php
    construction of the device edit form

device/device-record.php
    fetch and update of one database record

lib/rx-page.php
    complete HTML document and top-level widget composition

lib/rx-container.php
    generic widget container

lib/rx-widget.php
    common widget interface

lib/rx-db.php
    PostgreSQL connection

config/rx-db-config.php
    PostgreSQL configuration
```

### 3.2 Application pages shall be simple

Public page files shall primarily:

- read request parameters;
- construct RX widgets;
- add widgets to the page; and
- render the page.

Large form definitions, list generation, and SQL statements shall not normally be placed directly in public page files.

## 4 Naming requirements

### 4.1 Filesystem names

All filenames and directory names shall use `-` rather than `_`.

Correct:

```
rx-page.php
rx-domain-menu.php
device-edit.php
device-edit-form.php
device-record.php
rx-list.css
```

Incorrect:

```
rx_page.php
device_edit.php
device_edit_form.php
rx_list.css
```

### 4.2 PHP identifiers

PHP identifiers may and should use `_`.

```php
class rx_page
class device_list
class device_edit_form
class device_record

$start_row
$number_of_rows
$device_uuid
```

CamelCase is not required by this framework.

## 5 Directory structure

The implementation shall approximately use the following structure:

```
config/
    rx-db-config.php

device/
    device-edit-form.php
    device-list.php
    device-record.php

lib/
    rx-widget.php
    rx-container.php
    rx-page.php
    rx-page-library.php
    rx-db.php

    page-header.php
    page-footer.php
    page-grid.php

    menu-widget.php
    rx-domain-menu.php
    rx-object-menu.php

    form-widget.php
    form-field.php
    input-field.php
    number-field.php
    hidden-field.php
    output-field.php
    form-section.php
    record-info-section.php

    list-widget.php
    text-widget.php
    legacy-widget.php

public/
    index.php
    device-list.php
    device-edit.php

    css/
        rx-base.css
        rx-page.css
        rx-menu.css
        rx-form.css
        rx-list.css
        rx-footer.css
```

## 6 Widget model

### 6.1 Widget interface

All renderable RX components shall implement one common interface:

```php
interface rx_widget
{
    public function render(): string;
}
```

Anything implementing `rx_widget` shall therefore be usable wherever an RX widget is accepted.

### 6.2 Generic container

A generic `rx_container` shall implement `rx_widget`. It shall maintain an ordered array of child widgets.

```php
class rx_container implements rx_widget
{
    protected array $widgets = [];

    public function add(rx_widget $widget): void;

    protected function render_widgets(): string;

    public function render(): string;
}
```

Widgets shall be rendered in exactly the order in which they were added.

### 6.3 RX page

`rx_page` shall extend `rx_container`.

```php
$page = new rx_page('Devices');

$page->add(...);
$page->add(...);
$page->add(...);

echo $page->render();
```

`rx_page` shall be responsible for generating the complete HTML document:

```html
<!DOCTYPE html>
<html>
<head>
    ...
</head>
<body>
    ...
</body>
</html>
```

It shall not contain device-specific knowledge.

## 7 Stylesheet architecture

### 7.1 Loading order

The RX page shall automatically load the framework stylesheets in this order:

```
/css/rx-base.css
/css/rx-page.css
/css/rx-menu.css
/css/rx-form.css
/css/rx-list.css
/css/rx-footer.css
```

Optional application-specific stylesheets may be passed to `rx_page`. They shall be loaded after the RX stylesheets so that application CSS can override framework defaults.

### 7.2 CSS responsibilities

#### 7.2.1 rx-base.css

Shall contain global RX definitions such as font defaults, box sizing, common colours, CSS variables, and common spacing.

#### 7.2.2 rx-page.css

Shall contain the page body, main page container, page header, page grid, left/right/bottom panels, and general page layout.

#### 7.2.3 rx-menu.css

Shall contain the domain menu, object menu, active menu state, and menu links.

#### 7.2.4 rx-form.css

Shall contain forms, form rows, fields, labels, inputs, output fields, form sections, buttons, and record information.

#### 7.2.5 rx-list.css

Shall contain the list table, rows, clickable cells, pager, Previous/Next links, and empty-list state.

#### 7.2.6 rx-footer.css

Shall contain page footer styling.

## 8 Menu model

### 8.1 Domain menu

A domain menu shall be represented by:

```php
new rx_domain_menu('device')
```

Typical domains are Company, Devices, Sites, and Users. The currently selected domain shall have an active visual state.

### 8.2 Object menu

A device object menu shall support at least:

```
Home
List
Edit
```

Example:

```php
new rx_object_menu(
    'device',
    'list'
);
```

The second argument identifies the active menu item. The List item shall link to `/device-` `list.php`; the Edit item shall link to `/device-edit.php`.

## 9 Device list

### 9.1 Public list page

The public device list page shall not fetch device records itself. It shall only determine paging parameters and compose the page.

```php
$start_row = max(
    0,
    (int)($_GET['start-row'] ?? 0)
);

$number_of_rows =
    (int)($_GET['rows'] ?? 100);

if ($number_of_rows < 1)
{
    $number_of_rows = 100;
}

$number_of_rows = min(
    $number_of_rows,
    500
);

$page = new rx_page('Device list');

$page->add(
    new page_header(
        'rScada',
        'Device list example'
    )
);

$page->add(
    new rx_domain_menu('device')
);

$page->add(
    new rx_object_menu(
        'device',
        'list'
    )
);

$page->add(
    device_list::create(
        $start_row,
        $number_of_rows
    )
);

$page->add(
    new page_footer(
        'Object-oriented page framework example'
    )
);

echo $page->render();
```

### 9.2 List interface

The device list shall be created with:

```php
device_list::create(
    $start_row,
    $number_of_rows
);
```

It shall not normally receive an array of database records.

Incorrect:

```php
device_list::create($records);
```

Correct:

```php
device_list::create(
    100,
    100
);
```

### 9.3 Paging semantics

`start_row` shall be zero based. With 100 rows per page:

```
start-row=0
    first page

start-row=100
    second page

start-row=200
    third page
```

Example URLs:

```
/device-list.php?start-row=0&rows=100
/device-list.php?start-row=100&rows=100
/device-list.php?start-row=200&rows=100
```

Default values shall be:

```
start-row = 0
rows
          = 100
```

The maximum permitted page size shall be 500 rows.

### 9.4 List database access

`device_list` shall perform its own database fetch when constructed.

```php
public function __construct(
    int $start_row,
    int $number_of_rows
)
{
    ...

    $connection =
        rx_db_connection();

    $this->total_rows =
        $this->fetch_total_rows(
            $connection
        );

    $this->records =
        $this->fetch_records(
            $connection
        );
}
```

### 9.5 Count query

The list shall obtain the total number of records:

```sql
SELECT
    COUNT(*) AS row_count
FROM
    rscada.device;
```

The count shall be used to show the current range, determine whether Previous and Next are available, and correct an offset beyond the end of the table.

### 9.6 Paging query

The list shall use PostgreSQL server-side paging.

```sql
SELECT
    device_uuid,
    name,
    device_type,
    location,
    status,
    last_updated
FROM
    rscada.device
ORDER BY
    name,
    device_uuid
OFFSET $1 ROWS
FETCH FIRST $2 ROWS ONLY;
```

Parameters shall be passed with `pg_query_params()`. Paging values shall never be concatenated directly into SQL.

A paged query shall always contain an `ORDER BY`. Ordering by both `name` and `device_uuid` is preferred because it remains deterministic when multiple devices have identical names.

### 9.7 Pager

The list shall display information similar to:

```
Rows 101-200 of 4000
```

It shall provide Previous and Next controls. Previous shall be disabled on the first page, Next shall be disabled on the final page, and the pager shall preserve the current `rows` value.

### 9.8 Clickable rows

Every device row shall lead to the edit page. The edit URL shall contain `device_uuid`, `start-row`, and `rows`.

```
/device-edit.php?device_uuid=550e8400-e29b-41d4-a716-446655440000&start-row=100&
    rows=100
```

This allows the user to return to the same list position after editing. The UUID, rather than the name or row number, shall identify the database record.

## 10 Device record access

### 10.1 Record class

Database operations concerning a single device shall be encapsulated in:

```php
class device_record
```

At minimum it shall provide:

```php
device_record::fetch(
    string $device_uuid
): ?array;
```

and:

```php
device_record::update(
    string $device_uuid,
    array $values
): array;
```

The public edit page shall not contain the device `SELECT` or `UPDATE` SQL.

### 10.2 Fetching one device

A device shall be selected using its UUID.

```sql
SELECT
    d.device_uuid,
    d.name,
    d.device_id,
    d.device_type_uuid,
    d.site_uuid,
    d.device_type,
    d.location,
    d.description,
    d.status,
    d.last_updated,
    to_jsonb(d)->>'record_created'
        AS record_created,
    to_jsonb(d)->>'record_owner_uuid'
        AS record_owner_uuid,
    to_jsonb(d)->>'last_updated_by_uuid'
        AS last_updated_by_uuid,
    to_jsonb(d)->>'version'
        AS version
FROM
    rscada.device AS d
WHERE
    d.device_uuid = $1::uuid;
```

The query shall use `pg_query_params()`.

### 10.3 UUID validation

UUID values received from requests shall be validated before database use. The implementation shall accept canonical UUID format, for example:

```
809ad97e-bc75-11ef-8f06-c3822b09ef38
```

Invalid UUID input shall not be passed to the database as a valid object identifier.

## 11 Device edit form

### 11.1 Form separation

The form definition shall live in:

```
device/device-edit-form.php
```

The public page shall not contain a long sequence of form field definitions. Instead it shall use:

```php
$form = device_edit_form::create(
    $record,
    $start_row,
    $number_of_rows
);
```

This keeps page composition separate from form construction.

### 11.2 Form contents

The edit form shall expose:

```
Device UUID
Name
Device ID
Device type UUID
Site UUID
Type
Status
Location
Description
```

`device_uuid` shall be displayed but shall not be directly editable.

The form shall also support a reusable record information section containing metadata such as Created, Updated, Status, Created by, Updated by, and Version.

### 11.3 Hidden edit fields

The form shall preserve these values as hidden fields:

```
device_uuid
start-row
rows
```

Example:

```php
$form->add_hidden(
    new hidden_field(
        'device_uuid',
        $record['device_uuid']
    )
);

$form->add_hidden(
    new hidden_field(
        'start-row',
        (string)$start_row
    )
);

$form->add_hidden(
    new hidden_field(
        'rows',
        (string)$number_of_rows
    )
);
```

### 11.4 Form submission

The device form shall submit using:

```
POST /device-edit.php
```

The submit button shall be labelled Save.

## 12 Device update

### 12.1 Update operation

On Save, `device_record::update()` shall update the database record identified by `device_uuid`.

Required fields are:

```
name
device_id
device_type_uuid
site_uuid
device_type
status
```

Optional fields may include:

```
location
description
```

The update shall use parameterized SQL:

```sql
UPDATE
    rscada.device
SET
    name = $2,
    device_id = $3,
    device_type_uuid = $4::uuid,
    site_uuid = $5::uuid,
    device_type = $6,
    location = $7,
    description = $8,
    status = $9,
    last_updated = CURRENT_TIMESTAMP
WHERE
    device_uuid = $1::uuid
RETURNING
    device_uuid;
```

### 12.2 Update validation

Before issuing the `UPDATE`:

- `device_uuid` shall be a valid UUID;
- `name` shall not be empty;
- `device_id` shall not be empty;
- `device_type_uuid` shall be a valid UUID;
- `site_uuid` shall be a valid UUID;
- `device_type` shall not be empty; and
- `status` shall not be empty.

### 12.3 Post/Redirect/Get

After a successful Save, the application shall redirect rather than immediately render the POST response.

```
POST /device-edit.php
        |
        | UPDATE
        v
302 Location:
/device-edit.php?device_uuid=...&start-row=100&rows=100&saved=1
```

This prevents accidental duplicate updates if the browser refreshes the page.

### 12.4 Reload after update

After a successful database update, the saved record shall be fetched again from PostgreSQL. The form shall therefore display the authoritative database state rather than merely redisplaying the submitted POST values.

### 12.5 Error behaviour

If the requested UUID does not identify a record, HTTP 404 is appropriate.

If validation or a database operation fails:

- the error shall be displayed;
- submitted values should remain visible when practical; and
- the user shall not lose form contents merely because the database update failed.

## 13 PostgreSQL integration

### 13.1 Connection management

Database connection management shall be centralized in:

```
lib/rx-db.php
```

Application widgets shall obtain the connection using:

```php
$connection =
    rx_db_connection();
```

They shall not repeatedly construct their own connection strings.

### 13.2 Configuration

Configuration shall live outside the individual widgets in:

```
config/rx-db-config.php
```

Supported environment variables shall be:

```
RX_DB_HOST
RX_DB_PORT
RX_DB_NAME
RX_DB_USER
RX_DB_PASSWORD
```

Example defaults:

```php
return [
    'host' =>
        getenv('RX_DB_HOST')
            ?: '127.0.0.1',

    'port' =>
        getenv('RX_DB_PORT')
            ?: '5432',

    'dbname' =>
        getenv('RX_DB_NAME')
            ?: 'rscada',

    'user' =>
        getenv('RX_DB_USER')
            ?: 'rscada',

    'password' =>
        getenv('RX_DB_PASSWORD')
            ?: ''
];
```

### 13.3 Database security

SQL containing request or form data shall use `pg_query_params()`. User input shall not be directly concatenated into SQL. Database credentials shall not be hard-coded in application page files. The PostgreSQL user used by the web application should have only the permissions required by the application.

## 14 Output safety

### 14.1 HTML escaping

All database values and other dynamic text rendered as HTML shall be escaped using the equivalent of:

```php
htmlspecialchars(
    $value,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
);
```

URL query strings should be produced with:

```php
http_build_query()
```

rather than manually concatenating unescaped values.

## 15 Responsibility summary

The following distinction shall be maintained.

`device_list` owns:

```
COUNT
paged SELECT
pager
list table
links to records
```

`device_record` owns:

```
SELECT one record
validation for database update
UPDATE one record
reload one record
```

`device_edit_form` owns:

```
form layout
fields
hidden fields
record information section
```

`device-list.php` owns:

```
request paging parameters
page composition
```

`device-edit.php` owns:

```
GET/POST request flow
calling fetch/update
redirect after save
page composition
```

## 16 Object-oriented page composition

The preferred page-building API shall remain:

```php
$page = new rx_page('Page title');

$page->add(
    new rx_domain_menu('device')
);

$page->add(
    new rx_object_menu(
        'device',
        'list'
    )
);

$page->add(
    device_list::create(
        $start_row,
        $number_of_rows
    )
);

echo $page->render();
```

This is the object-oriented equivalent of the older procedural style:

```
rx_page_start();
rx_domain_menu();
...
rx_page_end();
```

The new version retains the readability of that style while removing global page-generation functions.

## 17 Reuse requirements

The RX library shall contain generic components. The device directory shall contain device-specific components.

```
lib/form-widget.php
    generic

device/device-edit-form.php
    device specific
lib/rx-page.php
    generic

device/device-list.php
    device specific
```

A future implementation for sites should therefore be able to follow the same pattern:

```
site/site-list.php
site/site-record.php
site/site-edit-form.php

public/site-list.php
public/site-edit.php
```

without modifying the generic RX classes.

## 18 Recommended application-page size

Public application pages should remain short enough that their structure can be understood immediately.

A reader opening `public/device-list.php` should see essentially:

```
read paging
create page
add header
add menus
add device list
add footer
render
```

A reader opening `public/device-edit.php` should see essentially:

```
read request
fetch/update record
create edit form
compose page
render
```

Field implementation details and SQL details belong elsewhere.

## 19 Page-state preservation

Navigating from list page to record, saving, and returning to the list shall preserve paging context. Therefore `start-row` and `rows` shall accompany the selected record.

This is important when editing records from a large table such as a 4,000-record device table.

## 20 Functional acceptance test

With approximately 4,000 records in `rscada.device`, the following sequence shall succeed:

1. Open `/device-list.php`.
2. The first 100 devices shall be displayed.
3. The page shall report approximately `Rows 1-100 of 4000`.
4. Press Next.
5. The URL shall become `/device-list.php?start-row=100&rows=100`.
6. Rows 101-200 shall be displayed.
7. Click a device row.
8. The browser shall open `/device-edit.php?device_uuid=...&start-row=100&rows=100`.
9. The device data shall be read from `rscada.device`.
10. The form shall display the database values.
11. Change, for example, Location.
12. Press Save.
13. PostgreSQL shall be updated.
14. `last_updated` shall be changed to the current timestamp.
15. The browser shall redirect to the GET version of the edit page.
16. The saved value shall be re-read from PostgreSQL and displayed.
17. Returning to the list shall retain `start-row=100` and `rows=100`.

## 21 Structural acceptance test

The implementation is structurally conformant when all of the following are true:

- every renderable component implements `rx_widget`;
- `rx_page` accepts widgets through `add()`;
- public pages do not contain long form definitions;
- public list pages do not fetch arrays of records before constructing the list;
- `device_list` fetches its own paged records;
- `device_record` owns single-record `SELECT` and `UPDATE` operations;
- `device_edit_form` owns form construction;
- SQL uses parameters;
- list rows identify records with `device_uuid`;
- Save updates by `device_uuid`;
- pagination context survives list/edit navigation;
- RX CSS is separated by responsibility;
- all filesystem filenames use `-`, never `_`;
- PHP identifiers may use `_`; and
- generic RX classes contain no unnecessary device-specific logic.

## 22 Core architectural rule

Pages compose widgets. Widgets implement presentation. Object classes implement object-specific behaviour. Record classes implement database access.

Thus a page should read like:

```php
$page = new rx_page('Devices');
$page->add(
    new rx_domain_menu('device')
);

$page->add(
    new rx_object_menu(
        'device',
        'list'
    )
);

$page->add(
    device_list::create(
        $start_row,
        $number_of_rows
    )
);

echo $page->render();
```

and not like a mixture of HTML generation, SQL, form field definitions, database updates, menu definitions, paging calculations, and CSS in one file.

That separation is the central design requirement of this RX implementation.
