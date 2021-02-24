# List upgrade

When you update an InterMine production database, user lists have to be updated as well. This document aims to describe this process.

## Why a list "upgrade" is needed

Lists are saved in the userprofile `savedbag`, `bagvalues` tables and in the production database `osbag_int` table.

### Production Database

**obsbag\_int table**

| **column** | **notes** |
| :--- | :--- |
| bagid | unique bag id |
| value | intermine object id |

{% hint style="info" %}
The InterMine ID is only valid per database. If the database is rebuilt, the IDs change and the information in this table becomes incorrect. The lists require an _upgrade_ for them to be updated with the new, correct InterMine object IDs.
{% endhint %}

### Userprofile Database

**savedbag table**

| **column** | **notes** |
| :--- | :--- |
| osbid | bag id |
| type | type of object, eg. Gene |
| id | id |
| name | name of list, provided by user |
| datecreated | timestamp |
| description | description, provided by user |
| userprofileid | user id |
| intermine\_state | CURRENT, NOT\_CURRENT or TO\_UPGRADE |

**bagvalues table**

| **column** | **notes** |
| :--- | :--- |
| savedbagid | bag id |
| value | identifier originally typed in by user |
| extra | organism short name |

Lists are saved along with the user information in the `savedbag` table. The identifiers used to create a list are also stored in the `bagvalues` table in the userprofile database. These identifiers are used to upgrade the list to internal object ids in the new production database.

To make queries fast, the list contents are stored in the production database as internal object ids. When a new production database is used, the object ids are no longer valid and need to be "upgraded".

## Process

* Upgrade lists only when users log in - so we won't waste time upgrading dormant user accounts and old lists.
* Superuser lists are upgraded when the webapp is first deployed.
* The webapp knows when the lists need to be upgraded. For this purpose a `serialNumber` identifying the production database is generated when we build a new production db and stored in the userprofile database when we release the webapp. If the two serialNumberbs don't match, the system should upgrade the lists.

## Upgrading to a new release

* When a new production db is built, all the lists have to be upgraded. Their state is set to NOT\_CURRENT.
* When a user logs in, a thread will begin upgrading their saved lists to the new release - finding and writing the corresponding object ids to the production database. If there are no issues \(all identifiers are resolved automatically\) the state of the list is set to CURRENT.
* The user can verify the state of their saved bags in MyMine-&gt;Lists page.
* If there are any issues, the state of the list is set to TO\_UPGRADE. These lists are shown in MyMine-&gt;List page in a separate table. The user can click on the Upgrade List link and browse in the bagUploadConfirm page where all conflicts will be displayed.
* Once the user has resolved any issues, the list can be saved clicking the button 'Upgrade a list of ...' and used for queries, etc. The state is set to CURRENT.
* If a user never logs in to a particular release, the list will not be upgraded, but can still be upgraded as normal if they log in to a later release.

## Lists not current

If a list is not current:

* the user can't use it in the query/template to add list constraints
* the list is not displayed in the List-&gt;View page
* the list is displayed in MyMine-&gt;Lists page, but the column `Current` is set `Not Current`. Selecting the link, the user can resolve any issue.
* the list is not displayed in the Lists section on the report pages

## bagvalues table

The list upgrade system, needs a bagvalues table in the userprofile database, with savedbagid and value columns. This table should be generated manually, running the `load-bagvalues-table` ant task in the webapp directory. The `load-bagvalues-table` task, should create the table and load the contents of the list using the former production db, that is the same db used to create the saved lists. Every time you re-create the userprofile database, you have to re-generate the 'bagvalues' table. In theory, you should never re-create the userprofile db, so you should run the `load-bagvalues-table` task only once.

## Userprofile database

The table should be populated with one row corresponding to each row in production db osbag\_int table. Each row should contain the `IntermineBag` id and the first value not empty of the primary identifier field, defined in the `class_keys` properties file.

The `bagvalues` table is updated when the user is logged in and:

* creates a new list from the result page or starting from some identifiers
* creates a new list from union, copy, intersection, subtraction operations
* adds or deletes some rows to/from the list
* deletes a list

When a user logs in, any lists he has created in his session become saved bags in the userprofile database, and the `bagvalues` table should be updated as well. The contents of `bagvalues` are only needed when upgrading to a new release. The thread upgrading the lists, uses the contents of bagvalues as input and, if the list upgrades with no issues:

* writes values to osbag\_int table
* sets in the savedbag table the intermine-current to true
* updates osbid.

The `intermine-current` in the table `savedbag` marks whether the bag has been upgraded. The column is generated when you create the userprofile database or when `load-bagvalues-table` has been executed.

## Serial Number Overview

The list upgrade functionality uses a serialNumber that identifies the production database. The serialNumber is re-generated each time we build a new production db. On startup of the webapp, the webapp compares the production serialNumber with its own serialNumber \(before stored using the production serialNumber\). If the two serialNumbers match, the lists will not be upgraded; if they don't, the lists are set as 'not current' and will be upgraded only when the user logs in.

There are four cases:

1. production serialNumber and userprofile serialNumber are both null ==&gt; we don't need to upgrade the list.

   > Scenario: I have released the webapp but I haven't rebuilt the production db.

2. production serialNumber is not null but userprofile serialNumber is null ==&gt; we need to upgrade the lists.

   > Scenario: I have run `build-db` in the production db and it's the first time that I release the webapp. On startup, the webapp sets `intermine_current` to false and the userprofile serialNumber value with the production serialNumber value.

3. production serialNumber = userprofile serialNumber ==&gt; we don't need to upgrade the lists.

   > Scenario: we have released the webapp but we haven't changed the production db.

4. production serialNumber != userprofile serialNumber ==&gt; we need to upgrade the lists.

   > Scenario: we have run `build-db` in the production and a new serialNumber has been generated.

The following diagram shows the possible states. With the green, we identify the states that don't need a list upgrade, with the red those need a list upgrade.

![](../../../.gitbook/assets/SerialNumber.png)

