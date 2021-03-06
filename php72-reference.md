 - [PHP 7.1 Reference](https://github.com/tpunt/PHP7-Reference/blob/master/php71-reference.md)
 - [PHP 7.0 Reference](https://github.com/tpunt/PHP7-Reference/blob/master/README.md)

---

# PHP 7.2

PHP 7.2 will be released towards the end of 2017. It brings a number of new features and changes that are outlined below.

[Features](#features)
 - [New `object` Type](#new-object-type)
 - [Extension Loading by Name](#extension-loading-by-name)
 - [Allow Abstract Method Overriding](#allow-abstract-method-overriding)
 - [Password Hashing with Argon2](#password-hashing-with-argon2)
 - [Extended String Types for ext/PDO](#extended-string-types-for-extpdo)
 - [Additional Emulated Prepares Debugging Information for ext/PDO](#additional-emulated-prepares-debugging-information-for-extpdo)
 - [Add Support for Extended Operations in ext/LDAP](#add-support-for-extended-operations-in-extldap)
 - [Address Information Additions to ext/sockets](#address-information-additions-to-extsockets)

[Changes](#changes)
 - [Prevent `number_format()` from Returning Negative Zero](#prevent-number_format-from-returning-negative-zero)
 - [Parameter Type Widening](#parameter-type-widening)
 - [Convert Numeric Keys in Object/Array Casts](#convert-numeric-keys-in-objectarray-casts)
 - [Deprecate and Remove Unquoted Strings](#deprecate-and-remove-unquoted-strings)
 - [Disallow Passing `null` to `get_class()`](#disallow-passing-null-to-get_class)
 - [Warn When Counting Non-Countable Types](#warn-when-counting-non-countable-types)
 - [Allow a Trailing Comma for Grouped Namespaces](#allow-a-trailing-comma-for-grouped-namespaces)
 - [Deprecate `png2wbmp()` and `jpeg2wbmp()`](#deprecate-png2wbmp-and-jpeg2wbmp)
 - [Move ext/hash from Resources to Objects](#move-exthash-from-resources-to-objects)
 - [Deprecate and Remove `INTL_IDNA_VARIANT_2003`](#deprecate-and-remove-intl_idna_variant_2003)
 - [Improve SSL/TLS Defaults](#improve-ssltls-defaults)

## Features

### New `object` Type

A new type, `object`, has been introduced that can be used for (contravariant) parameter typing and (covariant) return typing of any objects.

```php
function test(object $obj) : object
{
    return new SplQueue();
}

test(new StdClass());
```

RFC: [Object typehint](https://wiki.php.net/rfc/object-typehint)

### Extension Loading by Name

Extensions no longer require a file extension (.so for UNIX, .dll for Windows) to be specified. This is enabled in the php.ini file, as well as in the [`dl`](http://php.net/dl) function.

```php
; ini file
extension=php-ast
zend_extension=opcache
```

RFC: [Allow loading extensions by name](https://wiki.php.net/rfc/load-ext-by-name)

### Allow Abstract Method Overriding

Abstract methods can now be overridden when an abstract class extends another abstract class.

```php
abstract class A
{
    abstract function test(string $s);
}
abstract class B extends A
{
    // overridden
    abstract function test($s) : int; // still maintaining contravariance for parameters and covariance for return
}
```

RFC: [Allow abstract function override](https://wiki.php.net/rfc/allow-abstract-function-override)

### Password Hashing with Argon2

Argon2 has been added to the password hashing API (the `password_` functions), where the following constants have been exposed:
 - `PASSWORD_ARGON2I`
 - `PASSWORD_ARGON2_DEFAULT_MEMORY_COST`
 - `PASSWORD_ARGON2_DEFAULT_TIME_COST`
 - `PASSWORD_ARGON2_DEFAULT_THREADS`

RFC: [Argon2 Password Hash](https://wiki.php.net/rfc/argon2_password_hash)

### Extended String Types For ext/PDO

PDO's string type has been extended to support the national character type when emulating prepares. This has been done with the following constants:
 - `PDO::PARAM_STR_NATL`
 - `PDO::PARAM_STR_CHAR`
 - `PDO::ATTR_DEFAULT_STR_PARAM`

These constants are utilised by bitwise OR'ing them with `PDO::PARAM_STR`:
```php
$db->quote('über', PDO::PARAM_STR | PDO::PARAM_STR_NATL);
```

RFC: [Extended String Types For PDO](https://wiki.php.net/rfc/extended-string-types-for-pdo)

### Additional Emulated Prepares Debugging Information for ext/PDO

The [`PDOStatement::debugDumpParams()`](http://php.net/manual/en/pdostatement.debugdumpparams.php) method has been updated to include the SQL being sent to the DB, where the full, raw query (including the replaced placeholders with their bounded values) will be shown. This has been added to aid with debugging emulated prepares (and so it will only be available when emulated prepares are turned on).

RFC: [Debugging PDO Prepared Statement Emulation v2](https://wiki.php.net/rfc/debugging_pdo_prepared_statement_emulation_v2)

### Add Support for Extended Operations in ext/LDAP

Support for EXOP has been added to the LDAP extension. This has been done by exposing the following functions and constants:
 - `ldap_parse_exop(resource link, resource result [, string retdata [, string retoid]]) : bool`
 - `ldap_exop(resource link, string reqoid [, string reqdata [, string retdata [, string retoid]]]) : mixed`
 - `ldap_exop_passwd(resource link [, string user [, string oldpw [, string newpw ]]]) : mixed`
 - `ldap_exop_whoami(resource link) : mixed`
 - `LDAP_EXOP_START_TLS`
 - `LDAP_EXOP_MODIFY_PASSWD`
 - `LDAP_EXOP_REFRESH`
 - `LDAP_EXOP_WHO_AM_I`
 - `LDAP_EXOP_TURN`

RFC: [LDAP EXOP](https://wiki.php.net/rfc/ldap_exop)

### Address Information Additions to ext/sockets

The sockets extension now has the ability to lookup address information, as well as connect to it, bind to it, and explain it. The following four functions have been added for this:
 - `socket_addrinfo_lookup(string node[, mixed service, array hints]) : array`
 - `socket_addrinfo_connect(resource $addrinfo) : resource`
 - `socket_addrinfo_bind(resource $addrinfo) : resource`
 - `socket_addrinfo_explain(resource $addrinfo) : array`

RFC: [Implement socket_getaddrinfo()](https://wiki.php.net/rfc/socket_getaddrinfo)


## Changes

### Prevent `number_format()` from Returning Negative Zero

Previously, it was possible for the [`number_format`](http://php.net/number_format) function to return `-0`. Whilst this is perfectly valid according to the IEEE 754 floating point specification, this oddity was not desirable for displaying formatted numbers in a human-readable form.

```php
var_dump(number_format(-0.01)); // now outputs string(1) "0" instead of string(2) "-0"
```

RFC: [Prevent number_format() from returning negative zero](https://wiki.php.net/rfc/number_format_negative_zero)

### Parameter Type Widening

Parameter types from overridden methods and from interface implementations may now be omitted. This is still in compliance with LSP, since parameters types are contravariant.

```php
interface A
{
    public function Test(array $input);
}

class B implements A
{
    public function Test($input){} // type elided for $input
}
```

RFC: [Parameter Type Widening](https://wiki.php.net/rfc/parameter-no-type-variance)

### Convert Numeric Keys in Object/Array Casts

Numeric keys are now better handled when casting arrays to object and objects to arrays (either from explicit casting or by [`settype`](http://php.net/settype).

This means that integer (or stringy integer) keys from arrays being casted to objects are now accessible:
```php
// array to object
$arr = [0 => 1];
$obj = (object)$arr;
var_dump(
    $obj,
    $obj->{'0'}, // now accessible
    $obj->{0} // now accessible
);
/* Output:
object(stdClass)#1 (1) {
  ["0"]=>    // string key now, rather than integer key
  int(1)
}
int(1)
int(1)
*/
```

And integer (or stringy integer) keys from objects being casted to arrays are now accessible:
```php
// object to array
<?php

$obj = new class {
    public function __construct()
    {
        $this->{0} = 1;
    }
};
$arr = (array)$obj;
var_dump(
    $arr,
    $arr[0], // now accessible
    $arr['0'] // now accessible
);
/* Output:
array(1) {
  [0]=>    // integer key now, rather than string key
  int(1)
}
int(1)
int(1)
*/
```

RFC: [Convert numeric keys in object/array casts](https://wiki.php.net/rfc/convert_numeric_keys_in_object_array_casts)

### Deprecate and Remove Unquoted Strings

Unquoted strings that are non-existent global constants are taken to be strings of themselves. This behaviour used to emit an `E_NOTICE`, but will not emit an `E_WARNING`. The manual will be updated to reflect that this behaviour has been deprecated, and in the next major version of PHP, an exception will be thrown instead.

```
var_dump(NONEXISTENT);

/* Output:
Warning: Use of undefined constant NONEXISTENT - assumed 'NONEXISTENT' (this will throw an Error in a future version of PHP) in %s on line %d
string(11) "NONEXISTENT"
*/
```

RFC: [Deprecate and Remove Bareword (Unquoted) Strings](https://wiki.php.net/rfc/deprecate-bareword-strings)

### Disallow Passing `null` to `get_class()`

Previously, passing `null` to the [`get_class()`](http://php.net/get_class) function would output the name of the enclosing class. This behaviour has now been removed, where an `E_WARNING` will be output instead. To achieve the same behaviour as before, the argument should simply be elided instead.

RFC: [get_class() disallow null parameter](https://wiki.php.net/rfc/get_class_disallow_null_parameter)

### Warn When Counting Non-Countable Types

An `E_WARNING` will now be emitted when attempting to [`count()`](http://php.net/count) non-countable types (this includes the [`sizeof()`](http://php.net/sizeof) alias function).

```php
var_dump(
    count(1), // integers are not countable
    count('abc'), // strings are not countable
    count(new stdclass), // objects that do not implement the Countable interface are not countable
    count([1,2]) // arrays are countable
);

/* Output:
Warning: count(): Parameter must be an array or an object that implements Countable in %s on line %d

Warning: count(): Parameter must be an array or an object that implements Countable in %s on line %d

Warning: count(): Parameter must be an array or an object that implements Countable in %s on line %d
int(1)
int(1)
int(1)
int(2)
*/
```

RFC: [Counting of non-countable objects](https://wiki.php.net/rfc/counting_non_countables)

### Allow a Trailing Comma for Grouped Namespaces

A trailing comma can now be added to the group-use syntax introduced in PHP 7.0.

```php
use Foo\Bar\{
    Foo,
    Bar,
    Baz,
};
```

RFC: [Trailing Commas In List Syntax](https://wiki.php.net/rfc/list-syntax-trailing-commas)

### Deprecate `png2wbmp()` and `jpeg2wbmp()`

The functions [`png2wbmp()`](http://php.net/png2wbmp) and [`jpeg2wbmp()`](http://php.net/jpeg2wbmp) from ext/gd have now been deprecated and will be removed in PHP 8. This is because they did not really belong in ext/gd, since they were the only conversion functions from the extension and did not follow the naming conventions of the extension. A simple implementation can be made [in PHP instead](http://news.php.net/php.internals/96366).

RFC: [Deprecate png2wbmp() and jpeg2wbmp()](https://wiki.php.net/rfc/deprecate-png-jpeg-2wbmp)

### Move ext/hash from Resources to Objects

As part of the long-term migration away from resources, the Hash extension has been updated to use objects instead of resources. The change should be seamless for PHP developers, except for where `is_resource()` checks have been made (which will need updating to `is_object()` instead).

RFC: [Migration Hash Context from Resource to Object](https://wiki.php.net/rfc/hash-context.as-resource)

### Deprecate and remove INTL_IDNA_VARIANT_2003

The INTL extension has deprecated the `INTL_IDNA_VARIANT_2003`, which is the default for [`idn_to_ascii()`](http://php.net/idn_to_ascii) and [`idn_to_utf8()`](http://php.net/idn_to_utf8). PHP 7.4 will see these defaults changed to `INTL_IDNA_VARIANT_UTS46`, and the next major version of PHP will remove `INTL_IDNA_VARIANT_2003` altogether.

RFC: [Deprecate and remove INTL_IDNA_VARIANT_2003](https://wiki.php.net/rfc/deprecate-and-remove-intl_idna_variant_2003)

### Improve SSL/TLS Defaults

The following changes to the defaults have been made:
 - `tls://` now defaults to TLSv1.0 or TLSv1.1 or TLSv1.2
 - `ssl://` an alias of `tls://`
 - `STREAM_CRYPTO_METHOD_TLS_*` constants default to TLSv1.0 or TLSv1.1 + TLSv1.2, instead of TLSv1.0 only

RFC: [Improved SSL / TLS constants](https://wiki.php.net/rfc/improved-tls-constants)
