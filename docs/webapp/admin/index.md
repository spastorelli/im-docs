# Website Admin

The SuperUser is the administrator of your InterMine webapp. The SuperUser can use tagging to configure the appearance and functionality of the webapp.

The SuperUser account is created when the UserProfile database is built using the properties specified in the [Intermine properties](../properties/intermine-properties.md) file.

## Templates

All logged in users can create template queries, but the SuperUser can make them available to all users by tagging them as public templates. Making a template query is an easy way to get users of your webapp to the data they want very quickly.

## Tagging

### Template queries and lists

The SuperUser can change where templates and lists appear by adding tags via the templates and lists pages in the MyMine section of the webapp. Only the administrator can apply/view/edit tags starting at `im:` . The tag data is stored in the user-profile database.

| tag | purpose |
| :--- | :--- |
| [im:public](im:public) \[1\] | make list/template viewable by all users |
| [im:frontpage](im:frontpage) | put list on home page |
| [im:converter](im:converter) | template used in generating links in the ‘Convert’ section on the list analysis page |
| [im:aspect:CategoryName](im:aspect:CategoryName) | template appears underneath specified category. For instance, template with [im:aspect:Genomics](im:aspect:Genomics) tag will be displayed in Genomics category on the report page and on the home page |
| [im:report](im:report) | allows template to be displayed on report or list analysis page |
| [im:order:n](im:order:n) | specify the order lists should go in \(on homepage only currently\). If two lists have the same Integer “n” value, natural ordering on the list name will be applied as a decisive criterion |

> \[1\] Editable by all admins

### Fields and collections

The SuperUser can change how fields are displayed by adding tags via the report page.

| tag | purpose |
| :--- | :--- |
| [im:hidden](im:hidden) | hides the field/collection |
| [im:summary](im:summary) | adds collection to ‘Summary’ section of report page |
| [im:aspect:CategoryName](im:aspect:CategoryName) | collection appears underneath category |

### Classes

The SuperUser can change how classes are displayed by adding tags via the model browser.

| tag | purpose |
| :--- | :--- |
| [im:aspect:CategoryName](im:aspect:CategoryName) | class appears on aspect page |
| [im:preferredBagType](im:preferredBagType) | class appears first in the class selection |

### tag

If a template is tagged with `im:converter`, it is:

1. Used by the list analysis page, in the "Convert" section.
2. Used by the list upload page to convert between types.

> * E.g., the user pastes in a protein identifier, but chooses "Gene" from the type dropdown menu. A converter template can be used to look up the `Gene` corresponding to the given `Protein`.

To work as a converter, the template must follow the following pattern:

* the top-level class in the query must be the class we wish to convert _from_ \(eg. `Gene`\)
* there must be exactly one editable constraint - the `id` field of the top level class \(eg. `Gene.id`\)
* the fields selected for output must be `Gene.id` and the id field of the class to convert _to_

Normally the `id` field isn't shown in the query builder and probably isn't useful in other queries. Only the administrator user can create queries using the `id` field. Here is an example converter template:

```markup
<template name="Gene_To_Protein_Type_Converter" title="Gene to protein type converter" longDescription="" comment="">
    <query name="Gene_To_Protein_Type_Converter" model="genomic" view="Gene.id Gene.proteins.id" longDescription="" sortOrder="Gene.id asc">
        <node path="Gene" type="Gene"></node>
        <node path="Gene.id" type="Integer">
            <constraint op="=" value="0" description="Gene.id" identifier="Gene.id" editable="true" code="A"></constraint>
        </node>
    </query>
</template>
```

