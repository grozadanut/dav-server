# WARNING!!! Status of this project is WORK IN PROGRESS

## Description

The purpose of this project is to provide a CalDav and CardDav server implementation 
in order to connect calendars and contacts to the Moqui backend.

For information about the implementation status see the [Capabilities](#capabilities) section below.

## Testing

For testing the compliance with the RFC specification the following methods are used:
1. [Caldav Tester](https://github.com/CalConnect/caldavtester) is used for automatic testing.
2. [DAVx5](https://play.google.com/store/apps/details?id=at.bitfire.davdroid) and [Fossify Calendar](https://play.google.com/store/apps/details?id=org.fossify.calendar) are used as apps for manual testing on Android.

## Capabilities

### [WebDAV Class 1](https://datatracker.ietf.org/doc/html/rfc4918) ⏳🇼🇮🇵

Class 1 compliant resources MUST return, at minimum, the value "1" in
the DAV header on all responses to the OPTIONS method.

A class 1 compliant resource MUST meet all "MUST" requirements in all
sections of this document:
- **MUST** preserve dead properties(page 11) ❌
- not well-formed XML **MUST** return 400 Bad Request ❌
- return 415 Unsupported Media Type if body was passed to a method not expecting a body ❌
- **MUST** implement PROPFIND depth 0 and 1 ❌
  - prop ❌
  - allprop(include) ❌
  - propname ❌
- **SHOULD** implement PROPFIND depth infinity ❌

### [CalDAV RFC 4791](https://datatracker.ietf.org/doc/html/rfc4791) ⏳🇼🇮🇵

**Requirements Overview**

This section lists what functionality is required of a CalDAV server.
To advertise support for CalDAV, a server:

- **MUST** support iCalendar [RFC2445, Obsoleted by RFC 5545] as a media type for the calendar
object resource format; ❌
- **MUST** support WebDAV Class 1 [RFC2518, Obsoleted by RFC 4918] (note that [rfc2518bis]
describes clarifications to [RFC2518] that aid interoperability); ⏳🇼🇮🇵
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

### CardDav ❌

### References

- [WebDAV RFC 4918](https://datatracker.ietf.org/doc/html/rfc4918)
- [WebDAV ACL RFC 3744](https://datatracker.ietf.org/doc/html/rfc3744)
- [CalDAV RFC 4791](https://datatracker.ietf.org/doc/html/rfc4791)
- [iCalendar RFC 5545](https://datatracker.ietf.org/doc/html/rfc5545)
- [CardDAV RFC 6352](https://datatracker.ietf.org/doc/html/rfc6352)
- [vCard v3 RFC 2426](https://datatracker.ietf.org/doc/html/rfc2426)
- [vCard v4 RFC 6350](https://datatracker.ietf.org/doc/html/rfc6350)
- [iCal4j](https://www.ical4j.org/)
- [Cosmo](https://github.com/mam-dev/cosmo): CalDAV server implementation in Spring Boot