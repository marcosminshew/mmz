# Contentstack notes and Content Types documentation
For the purposes of this document "Entry" and "Entries" will have the meaning as defined by Contentstack; An Entry is a set of recorded values warehoused in a Content Type. What one might normally call a "record".

Creating large numbers of Entries in Contentstack is time-consumming. As of this writing there is no facility to act on multiple Entries simultaneously. Create, Delete, Export, and Import are one at a time with lots of mouse-clicks per. If you need manipulate a large set of Entries it will likely be faster to write a script that loops through a list of commands and fires them off to the server via curl or some such.

Beside being only one at a time the export/import will fail if your Entries have anything that requires a UID such as a "Reference" entity or an Asset. The UID(s) in the Stack you are importing into will have different UID values and the matching Entry/Asset will not be found. If there is sufficient data to warrant importing anyway edit the exported json manually and remove the offending UID value(s). You will still have to go in and manually hook up the Reference(s) and/or Asset(s) after the fact.

Some markdown conventions are used for formatting string values:
- In-line links
  To create an inline link use the markdown convention, i.e., \[this is a link\]\(to-a-location). If linking to a page within the site use named locations.
- In-line emphasized text
  Emphasized text is presented in Axiom orange. To execute emphasized text use surrounding underscores \_like this\_.

***
## List Content Types
Some Content Types are referenced by others and thus comprise a list of options to content editors in the Contentstack UI. For example, Case Studies (__case-study__) are associated with a Service Line (__service__). In order for a content editor to make the association the UI needs to display the list of __service__ Entries. This is accomplished via the Reference object. These List Content Types usually have nothing but a Title field. Any Content Type can be referenced, however, and thus be a list. For example the Home Page displays two featured Case Studies. To make the assocation the content editor selects two __case-study__ Entries in the page-home Entry. __case-study__ is not considered here, however, whereas those Entries are much more than just a list. Content Types that are used as lists are:
### service
The list of Service Lines, e.g., Responsive Resourcing, Intelligent Transactions, etc. Also presented to end users in a data entry form.
### industry
A list referenced by __case-study__ to allow filtering by Industry. One or more needed. Also presented to end users in a data entry form.
### form-field-type
A list referenced by the Content Type __form__. Displays the list of available input objects. New Entries and modifications require code changes to support.
### News Category
Referenced by __news-article__. May eventually be used to support filtering in an "All News Articles" UI.

***
## Utility Content Types
These Content Types support page elements such as the footer and main navigation or provide structure for data entry forms. Changes should only be made to the values in the Entries. Modifying the structure of these Content Types will require changes to the code. Ideally these will require very little modification.

### footer
Populates the footer with string values and links. Supports only one Entry.
|-----------------+------------+-----------------+----------------|
| Default aligned |Left aligned| Center aligned  | Right aligned  |
|-----------------|:-----------|:---------------:|---------------:|
| First body part |Second cell | Third cell      | fourth cell    |
| Second line     |foo         | **strong**      | baz            |
| Third line      |quux        | baz             | bar            |
|-----------------+------------+-----------------+----------------|
| Second body     |            |                 |                |
| 2 line          |            |                 |                |
|=================+============+=================+================|
| Footer row      |            |                 |                |
|-----------------+------------+-----------------+----------------|


| Field | Type | Notes |
|---+---+---|
| title | single-line text | used as H2 element |
| title_caption | multi-line text | paragraph shown proximal to title |
| contact_us_button_label | single-line text | label shown on Contact Us button |
| column_links | group |  |
| column_links.column_header | single-line text | shown as the column header (H4) |
| column_links.link | link | create one for each link shown in the column |
| jobs | group |  |
| jobs.jobs_header | single-line text | column header |
| jobs.jobs_caption | multi-line text | paragraph shown under the column header |
| bottom_links | group |  |
| bottom_links.bottom_link | link | create one for each link shown |
| bottom_links.log_in_to_myaxiom | link | data for the log in link |
| form | group | see __form__ discussion for use |
| sections | group | see __form__ discussion for use |
| fields | reference | reference to __form__, use sign-up-for-news |
| social_channels | group | create one for each Social Channel link |
| social_channels.title | single-line text | name of the channel |
| social_channels.link | single-line text | URL |
| social_channels.icon | single-line text | name of the icon to display |

![footer map][footer]

***
### list
Use when the web site needs to display a list of choices to the end user. Entries provide a list of values for the data entry objects select-list, select-list-multi, and radio. Each Entry is populated with two or more Groups with each Group representing a list item. Note that importing a large list of values is easy as the json can be generated programmatically. [An example payload here](http://minshew.com/misc/files/list-practice-area.json)
| Field | Type | Notes |
|---|---|---|
| title | single-line text |  |
| list_description | multi-line text | description of how/where the list is used, not displayed |
| list_item | group | create a new group for each list item |
| list_item.display_value | single-line text | the value to be displayed in the list |
| list_item.data_value | single-line text | the value to be stored if different from the display_value. if empty the display_value is used. |

***
### form
Constitutes meta-data for data entry forms. Changes to the Content Type's structure require code updates.

The design uses a pattern of sectioning long data entry forms into smaller chunks. As such to use a __form__ on a page first create a group for the entire data entry portion of the page, then create a nested group that will contain the section(s). Place the Reference object to __form__ inside the nested group. If the design calls for multiple sections set the nested group to allow multiple. The screen cap below if from the Content Type named page-job-app and is an example of a multi-section entry form.
![form UI][form-ui]
Entries in __form__ can be thought of as a collection of one or more fields that comprise data entry modules. Each field in the form is described in a form_fields group. The order of the form_fields groups is the order the entry fields will appear.

The field types select-list, select-list-multi, and radio all require a list of options be provided. These options come from one of two places: The values in a __list__ Entry or the Entries in a Content Type. Use form_fields.list_or_options or form_fields.content_type, respectively. The list_or_options is a Reference object that allows selection of a __list__ Entry. content_type is a text field that takes the UID of a Content Type, e.g., __industry__ or __service__. Specify one or the other but not both.
| Field | Type | Notes |
|---|---|---|
| title | single-line text | unused (except when establishing the reference) |
| form_description | multi-line text | optional description of form's use/purpose, not displayed |
| form_fields | group | create a group for each field in the form |
| form_fields.form_field_id | single-line text | a unique name (for the Entry) for the object |
| form_fields.field_type | reference | references __form_field_type__ |
| form_fields.list_or_options | reference | references __list__, use when the type is a list or radio button |
| form_fields.content_type | single-line text | the name of a Content Type from which to load list options |
| form_fields.is_required | boolean | if data entry is required |
| form_fields.field_label | single-line text | label attached to the field |
| form_fields.field_contents | single-line text | placeholder/prompt text, checkbox strings, etc. |
| form_fields.is_inline | boolean | field should be shown side-by-side with next/previous is_inline field |

![form fields map][form-fields]

***
### main-navigation
Drives the structure and content of the primary navigation elements which consists of a number of 'Main Nav' links (that connect to L2 Landing pages) each with zero or more 'Sub Nav' links (that lead to L3 pages). A single Entry is used. Adding Main Nav and/or Sub Nav elements will require routing updates.
| Field | Type | Notes |
|---|---|---|
| title | single-line text | unused |
| main_nav | group | create one for each main navigation element, contains a sub-group |
| main_nav.link | link | the name and target of the main navigation element |
| main_nav.icon_class | single-line text | currently unused |
| sub_nav | group | create one for each sub navigation element |
| sub_nav.link | link | the name and target of the sub navigation element |

![main navigation map][main-nav]

***
## Data Content Types
These Content Types provide content to the site and can be displayed as page elements, summarized in lists, or shown on detail pages where the full Entry is displayed. They are referenced by Page Content Types (see next section) to build out sections of the page.

### case-study
Case Studies have their own landing and detail pages and are referenced by __page-home__.  At least two Entries are needed are needed for the reference. __case-study__ references __industry__ and __service__ to allow contextual filtering. The current model accepts an image. In future video may be supported.
| Field | Type | Notes |
|---|---|---|
| title | single-line text | case study title |
| image | file, image only |  |
| client | single-line text |  |
| industry | reference | reference to __industry__ |
| body | rich text |  |
| services | reference | reference to __service__ |
| cta | single-line text | wording to show in the CTA |

***
### image-row
This Content Type supports elements of the design where one or more images are shown on a page full width in a single row. A group is used to define each image. The UI will divide the horiztonal space based on the images selected. In reponsive mode the images may stack vertically.

The image supports an optional Title and Copy that is displayed on top of the image.\* If values are supplied for the CTA (title & link) the image will support a hit area act as a link to the location given.\*

\* These features are supported in the Content Type but as of this writing they are not supported in the code.
| Field | Type | Notes |
|---|---|---|
| image | file, image only |  |
| title | Single-line text | text that appears over the image as a title |
| copy | Single-line text | additional text that appears over the image |
| CTA | link | if supplied the image is a hit area that navigates to the link |

***
### news-article
News Articles are shown as summaries on __page-home__ and a detail page is planned but not built as of the time of this writing. 3 or more are necessary to make the Home Page look balanced. More is better. Each article supports an image and a thumbnail of that image.
| Field | Type | Notes |
|---|---|---|
| title | single-line text | displayed as the article's title |
| category | reference | News Category |
| images | group |  |
| images.full_size_image | file, image only |  |
| images.thumbnail_image | file, image only | end group |
| body | rich text |  |
| cta | single-line text | wording displayed in the CTA |

***
### text-divider
This Content Type supports elements of the design where a row of text is shown full width. A CTA can optionally be shown. There are two types of text-dividers: text-only and heading divider. Only one should be entered but text-only will take priority if not. A heading divider allows a heading and sub-heading. See /join-us for examples of both on one page.
| Field | Type | Notes |
|---|---|---|
| title | single-line text | should describe the Entry's location and use |
| text_only-divider | group |  |
| content | rich-text | enter value to show as text-only, end group |
| heading_divider | group |  |
| heading | single-line text |  |
| sub_heading | single-line text | end group |
| cta | link | optional link |

***
### service-toolkit
Service Toolkits are shown on the Case Study Landing page. Each Entry presents a downloadable file to the user.
| Field | Type | Notes |
|---|---|---|
| title | single-line text | service toolkit title |
| image | file, image only |  |
| service | reference | reference to __service__ |
| description | multi-line text |  |
| download_file | file |  |

![service toolkit][service-toolkit]

***
## Page Content Types
These Content Types support individual pages on the site. They may reference other Utility and Data Content Types to build the page structure. Each supports a single Entry. When the content structure of a page is redesigned related changes will need to be made to the associated Content Type and possibly the code. Page Content Type names are prefixed with "page-" by convention.

The design calls for horizontal bands of text and images. Page Content Types may reference one or more __text-divider__ and __image-row__ Entries, respectively.

The content structure of these Content Types is subject to change as modifications are made to the page design. As such, the structure is not documented here. If you are having difficulty identifying the Content Type element that is responsible for the content on the page, edit the page's record and replace all the values with the field's name. Publish to a local environment and reload the page.

[footer]: http://minshew.com/misc/files/final_footer-map.png "final footer map"
[form-ui]: http://minshew.com/misc/files/form-ui.png "form UI"
[form-fields]: http://minshew.com/misc/files/form-fields.png "form fields map"
[main-nav]: http://minshew.com/misc/files/main-nav.png "main navigation map"
[service-toolkit]: http://minshew.com/misc/files/service-toolkit.png "service toolkit"