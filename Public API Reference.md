**Classification**: __Public__

**Last modified**: 2023-10-19 (revision 2)

# Public API Reference

## Overview

SMW Central's API is located at: `https://www.smwcentral.net/ajax.php`

All endpoints documented here are available for public use. We'll do our best to keep their input/output interfaces stable and to support old versions of the APIs. New properties being added to JSON objects won't be treated as a breaking change.

If you use these APIs in a project, we'll appreciate it if you let the Developer team know. It helps us track the popularity of our APIs and we'll be able to update you on any changes.

If you need access to data which isn't exposed by our public API, please get in touch with the Developer team. We'll gladly work with you to provide APIs for any use-cases you might have.

Any endpoint not documented here is internal and may change, break, or be decommissioned at any time. Parsing HTML from the website is also unsupported and we won't make any efforts to accommodate such usage.

## Revision History

- Revision 2 (2023-10-19)
  - All `File` objects in the Section APIs now have `tags` and `images` fields.

    As a result, `getsectionlist` now returns tags and images.
  - Added a new "Get section info" API.
  - APIs can now return information in any supported language.

## General Usage

Parameters to GET APIs ("URL parameters") are passed in the URL, alongside the parameters that identify the endpoint. URL parameters follow [PHP's rules for query strings](https://www.php.net/manual/en/language.variables.external.php).

All APIs return JSON data (regardless of request headers).

### Pagination

Some endpoints return paginated results. Responses for these APIs have the following format:

- `total`: *number*

  Total records in the entire result set.
- `per_page`: *number*

  Number of records per page. Usually, there is no way to change this.
- `current_page`: *number*

  Page number returned in the current response.
- `last_page`: *number*

  Number of the last page.
- `first_page_url`: *string*

  URL of the first page. Make a request to this URL to get the first page of the results.
- `last_page_url`: *string*

  URL of the last page.
- `next_page_url`: *string | null*

  URL of the next page, or `null` if this response is the last page.
- `prev_page_url`: *string | null*

  URL of the previous page, or null if this response is the first page.
- `from`: *number*

  Index (starting at 1) of the first record in this response. For the first page, this is 1.
- `to`: *number*

  Index (starting at 1) of the last record in this response. For the last page, this is `total`.
- `data`: *object\[\]*

  Results, see the specific endpoint for information.

### Language

All APIs which have localized strings in their results support the following URL parameter:

- `locale`: *string*

  Language to return localized strings in. One of: `en-US`, `fr-CA`, `pt-BR`, `ja-JP`

  Defaults to `en-US`.

## Sections

### Types

#### `User`

- `id`: *number | null*

  User ID on SMW Central.

  Some `User` objects may refer to unregistered users, in which case this is `null`. References to a `User` will explicitly state whether this is possible.
- `name`: *string*

  Username. If the user is registered, this will be their SMW Central username.

#### `File`

- `id`: *number*

  File ID.
- `section`: *string*

  Section name.
- `name`: *string*

  File name.
- `time`: *number*

  Time at which the file was added to the section (for moderated files) or submitted (for waiting files). UNIX timestamp.
- `moderated`: *boolean*  
  Whether the file is moderated.
- `authors`: *User\[\]*

  Authors of the file. Users may be unregistered.
- `submitter`: *User | null*

  For waiting files, user who submitted the file. Always `null` for moderated files.
- `tags`: *string\[\]*

  Array of tags on the file.
- `images`: *string\[\] | null*

  If the section has images, array of URLs for each image on the file.
- `rating`: *number | null*

  File rating, if the section allows ratings.
- `size`: *number*

  File size in bytes.
- `downloads`: *number*

  Number of downloads.
- `download_url`: *string*  
  URL at which the file can be downloaded. Make a `GET` request to download the file.
- `obsoleted_by`: *number | null*

  If the file is obsolete (i.e. an update has been submitted), the ID of the newest version.
- `fields`: *string\[\]*

  Section-specific fields, returned as safe HTML. Localized values always use the `en_US` translation.

### Endpoints

#### Get section info

`GET` `/ajax.php?a=getsectioninfo` 

Fetches information about a section.

**URL parameters**

- `s`: *string*, **required**

  Name of the section.
- `with`: *string\[\]*

  Additional fields to include in the response. Valid items: `description`, `submission_guidelines`

**Response**

Returns a JSON object with the following fields:

- `name`: *string*

  Name of the section.
- `friendly_name`: *string*

  User-friendly (localized) name of the section. Returned in the selected language.
- `game`: *string*

  Game for the section. One of: `smw`, `yi`, `sm64`
- `description`: *string | null*

  Description for the section, or `null` if `description` wasn't requested in the `with` parameter.
- `submission_guidelines`: *string*

  Submission guidelines for the section, or `null` if description wasn't requested in the `with` parameter.
- `waiting_count`: *number*

  Number of waiting files in the section.
- `max_file_size`: *string*

  Max allowed file size for uploads to the section, in bytes.
- `removal_forum`: *string*

  ID of the forum where removal logs for the section are posted in.
- `images`: *object*

  Information about the images used in the section.
- `images.allowed`: *boolean*

  Whether images can be uploaded to the section. If `false`, all `File` objects belonging to this section will have an `images` field set to `null`.
- `images.min_count`: *number*

  Min number of images required. Only present if `images.allowed` is `true`.
- `images.max_count`: *number*

  Max number of images allowed. Only present if `images.allowed` is `true`.
- `images.max_file_size`: *number*

  Max allowed file size for images in the section. Only present if `images.allowed` is `true`.
- `images.max_width`: *number*

  Max width of an image, in pixels. Only present if `images.allowed` is `true`.
- `images.max_height`: *number*

  Max height of an image, in pixels. Only present if `images.allowed` is `true`.
- `allows_submissions`: *boolean*

  Whether users can submit files to the section.
- `allows_ratings`: *string*

  Whether users can rate files in the section.
- `allows_comments`: *string*

  Whether users can comment on files in this section.
- `default_order`: *string*

  Default field that the section is ordered by. Used by "Get section files" if no order is specified. One of: `date`, `name`, `filesize`, `downloads`, or the name of a section field with `allow_order` set to `true`.
- `fields`: *object\[\]*

  An array of all section-specific fields.
- `fields[].id`: *number*

  Internal unique ID of the field.
- `fields[].name`: *string*

  Name of the field.
- `fields[].friendly_name`: *string*

  User-friendly (localized) name of the field. Returned in the selected language.
- `fields[].list_name`: *string*

  User-friendly (localized) name of the field when shown on the section list page. Returned in the selected language.
- `fields[].description`: *string*

  Description of the field.
- `fields[].type`: *string*

  Type of the field. One of: `short_text`, `long_text`, `number`, `url`, `checkbox`, `single_choice`, `multi_choice`
- `fields[].options`: *object\[\] | null*

  For `single_choice` and `multi_choice` fields, the options accepted by the field.
- `fields[].options[].id`: *number*

  Internal ID of the option. This is the ID used by filters (see "Get section files" below).
- `fields[].options[].name`: *string*

  Name of the option.
- `fields[].options[].friendly_name`: *string*

  User-friendly (localized) name of the option. Returned in the selected language.
- `fields[].in_list`: *boolean*

  Whether this field appears on the section list page.
- `fields[].allow_order`: *string*

  Whether this field supports sorting. If `true`, its `name` is a valid option for sorting fields (see `o` in "Get section files" below).
- `fields[].allow_filter`: *string*

  Whether this field supports filtering. If `true`, its `name` can be used in filters (see `f` in "Get section files" below).

#### Get section files

`GET` `/ajax.php?a=getsectionlist`

Fetches a list of all files in a section. Supports filtering and sorting.

This endpoint is completely equivalent to the section list page on the website and uses the same URL parameters. A `/?p=section&a=list` URL can be directly converted to this endpoint.

**URL parameters**

- `s`: *string*, **required**

  Name of the section to query.
- `u`: *int*

  If set to a non-zero value, show waiting files. If missing or `0`, show moderated files.
- `n`: *number*

  Page number.
- `o`: *string*

  Field to sort the results by, one of: `date`, `name`, `filesize`, `downloads`, or the name of a section field (see "Get section info" above) with `allow_order` set to `true`.

  Default value is section-dependent (see `default_order` in "Get section info" above), typically `date`.
- `d`: *string*

  Sort direction (ascending or descending), one of: `asc`, `desc`

  Default value is `desc` for moderated files, `asc` for waiting files.
- `f`: *array*

  Filters to apply. This is an associative array keyed by the field name (see `fields[].name` in "Get section info" above).

  For single- and multi-choice fields, values should be arrays of option IDs (see `fields[].options` in "Get section info" above).

**Response**

Returns a paginated response (see Pagination above). Records in the `data` array are `File` objects (see Types above).

#### Get file

`GET` `/ajax.php?a=getfile&v=2`

Fetches detailed information about a file.

**URL parameters**

- `id`: *number*, **required**

  File ID.

**Response**

Returns a `File` object (see Types above). The returned object also contains the following additional values:

- `versions`: *object\[\]*

  The full version history of the file, sorted from newest to oldest.

  One of these will correspond to the requested file. For up-to-date files, this will be the first entry. For obsolete files, it will be a later entry.
- `versions[].id`: *number*

  File ID of the version.
- `versions[].name`: *string*

  Name of the version.
- `versions[].obsolete`: *boolean*

  Whether this version is obsolete.