# LDAP Filtering Explained

* You will notice in the queries
  * using strings such as `userAccountControl:1.2.840.113556.1.4.803:=8192`.
* These strings are common LDAP queries that can be used with several different tools too, including AD PowerShell, ldapsearch, and many others.
* `userAccountControl:1.2.840.113556.1.4.803:`&#x20;
  * Specifies that we are looking at the [User Account Control (UAC) attributes](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/useraccountcontrol-manipulate-account-properties) for an object.
  * This portion can change to include three different values we will explain below when searching for information in AD (also known as [Object Identifiers (OIDs)](https://ldap.com/ldap-oid-reference-guide/).
  * `=8192` represents the decimal bitmask we want to match in this search.
    * This decimal number corresponds to a corresponding UAC Attribute flag that determines if an attribute like `password is not required` or `account is locked` is set.
    * These values can compound and make multiple different bit entries. Below is a quick list of potential values.

### UAC Values

!\[\[Pasted image 20240907115535.png]]

### OID match strings

* OIDs are rules used to match bit values with attributes
* For LDAP and AD, there are three main matching rules

`1.2.840.113556.1.4.803`

* When using this rule as we did in the example above, we are saying the bit value must match completely to meet the search requirements.

`1.2.840.113556.1.4.804`

* When using this rule, we are saying that we want our results to show any attribute match if any bit in the chain matches.
* This works in the case of an object having multiple attributes set.

`1.2.840.113556.1.4.1941`

* This rule is used to match filters that apply to the Distinguished Name of an object and will search through all ownership and membership entries.

### Logical Operators

* When building out search strings, we can utilize logical operators to combine values for the search.
* The operators `&` `|` and `!` are used for this purpose. For example we can combine multiple [search criteria](https://learn.microsoft.com/en-us/windows/win32/adsi/search-filter-syntax) with the `& (and)` operator like so:

```
(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=64))
```

* The above example sets the first criteria that the object must be a user and combines it with searching for a UAC bit value of 64 (Password Can't Change).
* A user with that attribute set would match the filter.
* You can take this even further and combine multiple attributes like `(&(1) (2) (3))`.
* The `!` (not) and `|` (or) operators can work similarly.
* For example, our filter above can be modified as follows:

```
(&(objectClass=user)(!userAccountControl:1.2.840.113556.1.4.803:=64))
```

* This would search for any user object that does `NOT` have the Password Can't Change attribute set.
* When thinking about users, groups, and other objects in AD, our ability to search with LDAP queries is pretty extensive.

