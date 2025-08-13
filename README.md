# WARNING!!! Status of this project is WORK IN PROGRESS

# Description

The purpose of this project is to provide a CalDav and CardDav server implementation 
in order to connect calendars and contacts to the Moqui backend.

For information about the implementation status see the [Capabilities](#capabilities) section below.

# Features

- Personal Calendars for each user ⏳🇼🇮🇵
- Group Calendars(eg: for meeting rooms or fixed assets) ❌
- Public Calendars(eg: for holidays) ❌

# Testing

For testing the compliance with the RFC specification the following methods are used:
1. [DAVx5](https://play.google.com/store/apps/details?id=at.bitfire.davdroid) and [Fossify Calendar](https://play.google.com/store/apps/details?id=org.fossify.calendar) are used as apps for manual testing on Android.
2. Moqui Spock tests written based on the specification are used for automated testing.

# Architecture
## WebDAV

WebDav collections are similar with directories in a file system, and resources are similar with files.

### Data mapping

WebDav collections are stored in `moqui.resource.DbResource`, using the following fields:
- **resourceId:** autoincrement
- **parentResourceId:** parent collection resourceId
- **filename:** last part of the collection URL
- **isFile:** false

Collection dead properties are stored as:
- `moqui.resource.DbResource`:
  - **resourceId:** parentResourceId+"_dprop"
  - **parentResourceId:** resourceId of collection
  - **filename:** "dproperties.xml"
  - **isFile:** true
- `moqui.resource.DbResourceFile`:
  - **resourceId:** DbResource.resourceId
  - **mimeType:** "text/xml"
  - **versionName:** null
  - **rootVersionName:** null
  - **fileData:** dead properties xml

### Data statements



## CalDAV

To ensure preservation of unknown VCOMPONENT properties, the original .ics calendar resource is stored in the
`mantle.work.effort.WorkEffortContent` entity as a resource. When requesting a calendar resource, the dav server uses 
the iCal4j library to parse the .ics file and then overrides the known properties using WorkEffort fields.
When storing an .ics calendar resource the server stores the .ics data as-is in the `WorkEffortContent` entity, and then 
stores the known properties in the `WorkEffort` fields.

### Data mapping



### Data statements



# Capabilities

## [WebDAV Class 1 RFC 4918](https://datatracker.ietf.org/doc/html/rfc4918) ❌

Class 1 compliant resources MUST return, at minimum, the value "1" in
the DAV header on all responses to the OPTIONS method.

A class 1 compliant resource MUST meet all "MUST" requirements in all
sections of this document:
- **MUST** preserve dead properties(page 11) ❌
- not well-formed XML **MUST** return 400 Bad Request ❌
- return 415 Unsupported Media Type if body was passed to a method not expecting a body ❌
- **MUST** implement **PROPFIND** depth 0 and 1 ❌
  - prop ❌
  - allprop(include) ❌
  - propname ❌
- **SHOULD** implement **PROPFIND** depth infinity ❌
- **PROPPATCH** ❌
  - propertyupdate ❌
  - set ❌
  - remove ❌
- **MKCOL**(optional) ❌
- **GET, HEAD, DELETE** ❌
- **PUT**(optional on resources, not needed on collections) ❌
- **COPY** ❌
  - depth 0, infinity ❌
- **MOVE** ❌
- **OPTIONS** ❌

**Headers:**
- DAV ❌
- Depth ❌
- Destination ❌
- If(optional) ❌
- Overwrite ❌
- Timeout(optional) ❌

**Live properties:**
- getcontentlanguage ❌
- getcontentlength ❌
- getcontenttype ❌
- getetag ❌
- getlastmodified ❌
- resourcetype ❌
- creationdate(optional) ❌
- displayname(optional) ❌
- lockdiscovery(optional) ❌
- supportedlock(optional) ❌

## [WebDAV ACL RFC 3744](https://datatracker.ietf.org/doc/html/rfc3744) ❌

**Privileges**
- DAV:read ❌
  - GET
  - PROPFIND
  - OPTIONS
- DAV:write ❌
  - PUT
  - PROPPATCH
- DAV:write-properties ❌
  - PROPPATCH
- DAV:write-content ❌
  - PUT
- DAV:unlock ❌
  - UNLOCK by a principal other than the lock owner
- DAV:read-acl ❌
  - PROPFIND to retrieve the DAV:acl property of a resource
- DAV:read-current-user-privilege-set ❌
  - PROPFIND to retrieve the DAV:current-user-privilege-set property of a resource
- DAV:write-acl ❌
  - ACL to modify DAV:acl property of a resource
- DAV:bind ❌
  - collections only: add a member via PUT, MKCOL
- DAV:unbind ❌
  - collections only: remove a member via DELETE, MOVE
- DAV:all ❌

**Principal properties**
- DAV:displayname ❌
- DAV:resourcetype ❌
  - should return DAV:principal
- DAV:alternate-URI-set(protected) ❌
- DAV:principal-URL(protected) ❌
- DAV:group-member-set ❌
- DAV:group-membership ❌

**Access control properties**
- DAV:owner ❌
- DAV:group ❌
- DAV:supported-privilege-set ❌
- DAV:current-user-privilege-set ❌
- DAV:acl ❌
- DAV:acl-restrictions(protected) ❌
- DAV:inherited-acl-set(protected) ❌
- DAV:principal-collection-set(protected) ❌

**Existing methods modifications**
- all HTTP methods: 403 Forbidden MUST contain <DAV:error>, which contains <DAV:need-privileges> ❌
- OPTIONS: return `access-control` in DAV header ❌
- MOVE: preserve non-inherited non-protected ACEs in the DAV:acl property ❌
- COPY: DO NOT preserve DAV:acl ❌

**Methods**
- ACL ❌
- REPORT [RFC 3253 section 3.6](https://datatracker.ietf.org/doc/html/rfc3253#section-3.6)
  - DAV:expand-property [RFC 3253 section 3.8](https://datatracker.ietf.org/doc/html/rfc3253#section-3.8) ❌
  - DAV:acl-principal-prop-set ❌
  - DAV:principal-match ❌
  - DAV:principal-property-search ❌
  - DAV:principal-search-property-set ❌

## [CalDAV RFC 4791](https://datatracker.ietf.org/doc/html/rfc4791) ⏳🇼🇮🇵

**Requirements Overview**

This section lists what functionality is required of a CalDAV server.
To advertise support for CalDAV, a server:

- **MUST** support iCalendar [RFC2445, Obsoleted by RFC 5545] as a media type for the calendar
object resource format; ❌
- **MUST** support WebDAV Class 1 [RFC2518, Obsoleted by RFC 4918] (note that [rfc2518bis]
describes clarifications to [RFC2518] that aid interoperability); ❌
- **MUST** support WebDAV ACL [RFC3744] with the additional privilege
defined in Section 6.1 of this document; ❌
- **MUST** support transport over TLS [RFC2246] as defined in [RFC2818]
(note that [RFC2246] has been obsoleted by [RFC4346]); ❌
- **MUST** support ETags [RFC2616] with additional requirements
specified in Section 5.3.4 of this document; ❌
- **MUST** support all calendaring reports defined in Section 7 of this
document; ❌
- **MUST** advertise support on all calendar collections and calendar
object resources for the calendaring reports in the DAV:supported-
report-set property, as defined in Versioning Extensions to WebDAV
[RFC3253]. ❌

In addition, a server:

- **SHOULD** support the MKCALENDAR method defined in Section 5.3.1 of
this document. ❌

- property DAV:resourcetype = ❌
  - DAV:collection
  - CALDAV:calendar
- OPTIONS
  - DAV:calendar-access ❌

**Calendar collection properties**
- CALDAV:calendar-description (MAY) ❌
- CALDAV:calendar-timezone (SHOULD) ❌
- CALDAV:supported-calendar-component-set (MAY)(protected) ❌
- CALDAV:supported-calendar-data (MAY)(protected) ❌
- CALDAV:max-resource-size (MAY)(protected) ❌
- CALDAV:min-date-time (MAY)(protected) ❌
- CALDAV:max-date-time (MAY)(protected) ❌
- CALDAV:max-instances (MAY)(protected) ❌
- CALDAV:max-attendees-per-instance (MAY)(protected) ❌

**Creating calendar resources**
- PUT
  - If-None-Match: * ❌

**PUT, COPY, MOVE preconditions**
- CALDAV:supported-calendar-data ❌
- CALDAV:valid-calendar-data ❌
- CALDAV:valid-calendar-object-resource ❌
- CALDAV:supported-calendar-component ❌
- CALDAV:no-uid-conflict ❌
- CALDAV:calendar-collection-location-ok ❌
- CALDAV:max-resource-size ❌
- CALDAV:min-date-time ❌
- CALDAV:max-date-time ❌
- CALDAV:max-instances ❌
- CALDAV:max-attendees-per-instance ❌

**Privileges**
- CALDAV:read-free-busy ❌

**Principal property**
- CALDAV:calendar-home-set ❌

**Additional property**
- CALDAV:supported-collation-set (protected) ❌
  - i;ascii-casemap [RFC 4790 section 9.2](https://datatracker.ietf.org/doc/html/rfc4790#section-9.2)
  - i;octet [RFC 4790 section 9.3](https://datatracker.ietf.org/doc/html/rfc4790#section-9.3)

**Reports**
- REPORT
  - CALDAV:calendar-query ❌
  - CALDAV:calendar-multiget ❌
  - CALDAV:free-busy-query ❌

## CardDav ❌

## References

- [WebDAV RFC 4918](https://datatracker.ietf.org/doc/html/rfc4918)
- [WebDAV ACL RFC 3744](https://datatracker.ietf.org/doc/html/rfc3744)
- [CalDAV RFC 4791](https://datatracker.ietf.org/doc/html/rfc4791)
- [iCalendar RFC 5545](https://datatracker.ietf.org/doc/html/rfc5545)
- [CardDAV RFC 6352](https://datatracker.ietf.org/doc/html/rfc6352)
- [vCard v3 RFC 2426](https://datatracker.ietf.org/doc/html/rfc2426)
- [vCard v4 RFC 6350](https://datatracker.ietf.org/doc/html/rfc6350)
- [iCal4j](https://www.ical4j.org/)
- [Cosmo](https://github.com/mam-dev/cosmo): CalDAV server implementation in Spring Boot