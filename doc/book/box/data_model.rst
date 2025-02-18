.. _box_data_model:

================================================================================
Data model
================================================================================

This section describes how Tarantool stores values and what operations with data
it supports.

If you tried to create a database as suggested in our
:ref:`"Getting started" exercises <getting_started_db>`,
then your test database now looks like this:

.. image:: data_model.png

.. _index-box_space:

--------------------------------------------------------------------------------
Spaces
--------------------------------------------------------------------------------

A **space** -- 'tester' in our example -- is a container.

When Tarantool is being used to store data, there is always at least one space.
Each space has a unique **name** specified by the user.
Besides, each space has a unique **numeric identifier** which can be specified by
the user, but usually is assigned automatically by Tarantool.
Finally, a space always has an **engine**: *memtx* (default) -- in-memory engine,
fast but limited in size, or *vinyl* -- on-disk engine for huge data sets.

A space is a container for :ref:`tuples <index-box_tuple>`.
To be functional, it needs to have a :ref:`primary index <index-box_index>`.
It can also have secondary indexes.

.. _index-box_tuple:

--------------------------------------------------------------------------------
Tuples
--------------------------------------------------------------------------------

A **tuple** plays the same role as a “row” or a “record”, and the components of
a tuple (which we call “fields”) play the same role as a “row column” or
“record field”, except that:

* fields can be composite structures, such as arrays or maps, and
* fields don't need to have names.

Any given tuple may have any number of fields, and the fields may be of
different :ref:`types <index-box_data-types>`.
The identifier of a field is the field's number, base 1
(in Lua and other 1-based languages) or base 0 (in PHP or C/C++).
For example, ``1`` or ``0`` can be used in some contexts to refer to the first
field of a tuple.

The number of tuples in a space is unlimited.

Tuples in Tarantool are stored as
`MsgPack <https://en.wikipedia.org/wiki/MessagePack>`_ arrays.

When Tarantool returns a tuple value in the console,
by default it uses :ref:`YAML <interactive_console>` format,
for example: ``[3, 'Ace of Base', 1993]``.

.. _index-box_index:

--------------------------------------------------------------------------------
Indexes
--------------------------------------------------------------------------------

Read the full information about indexes on page :doc:`Indexes </book/box/indexes>`.

An **index** is a group of key values and pointers.

As with spaces, you should specify the index **name**, and let Tarantool
come up with a unique **numeric identifier** ("index id").

An index always has a **type**. The default index type is :ref:`TREE <indexes-tree>`.
TREE indexes are provided by all Tarantool engines, can index unique and
non-unique values, support partial key searches, comparisons and ordered results.
Additionally, memtx engine supports :ref:`HASH <indexes-hash>`,
:ref:`RTREE <indexes-rtree>` and :ref:`BITSET <indexes-bitset>` indexes.

An index may be **multi-part**, that is, you can declare that an index key value
is composed of two or more fields in the tuple, in any order.
For example, for an ordinary TREE index, the maximum number of parts is 255.

An index may be **unique**, that is, you can declare that it would be illegal
to have the same key value twice.

The first index defined on a space is called the **primary key index**,
and it must be unique. All other indexes are called **secondary indexes**,
and they may be non-unique.

.. _index-box_data-types:

--------------------------------------------------------------------------------
Data types
--------------------------------------------------------------------------------

Tarantool is both a database manager and an application server.
Therefore a developer often deals with two type sets:
the types of the programming language (such as Lua) and
the types of the Tarantool storage format (MsgPack).

.. _index-box_lua-vs-msgpack:

********************************************************
Lua versus MsgPack
********************************************************

.. container:: table

    .. rst-class:: left-align-column-1
    .. rst-class:: left-align-column-2
    .. rst-class:: left-align-column-3
    .. rst-class:: left-align-column-4

    +-------------------+-------------------------+--------------------------------+------------------------------+
    | Scalar / compound | MsgPack |nbsp| type     | Lua type                       | Example value                |
    +===================+=========================+================================+==============================+
    | scalar            | nil                     | "`nil`_"                       | ``nil``                      |
    +-------------------+-------------------------+--------------------------------+------------------------------+
    | scalar            | boolean                 | "`boolean`_"                   | ``true``                     |
    +-------------------+-------------------------+--------------------------------+------------------------------+
    | scalar            | string                  | "`string`_"                    | ``'A B C'``                  |
    +-------------------+-------------------------+--------------------------------+------------------------------+
    | scalar            | integer                 | "`number`_"                    | ``12345``                    |
    +-------------------+-------------------------+--------------------------------+------------------------------+
    | scalar            | float 64 (double)       | "`number`_"                    | ``1.2345``                   |
    +-------------------+-------------------------+--------------------------------+------------------------------+
    | scalar            | float 64 (double)       | "`cdata`_"                     | ``1.2345``                   |
    +-------------------+-------------------------+--------------------------------+------------------------------+
    | scalar            | binary                  | "`cdata`_"                     | ``[!!binary 3t7e]``          |
    +-------------------+-------------------------+--------------------------------+------------------------------+
    | scalar            | ext                     | "`cdata`_"                     | ``1.2``                      |
    |                   | (for Tarantool decimal) |                                |                              |
    +-------------------+-------------------------+--------------------------------+------------------------------+
    | scalar            | ext                     | "`cdata`_"                     | ``12a34b5c-de67-8f90-`` |br| |
    |                   | (for Tarantool uuid)    |                                | ``123g-h4567ab8901``         |
    +-------------------+-------------------------+--------------------------------+------------------------------+
    | compound          | map                     | "`table`_" (with string keys)  | ``{'a': 5, 'b': 6}``         |
    +-------------------+-------------------------+--------------------------------+------------------------------+
    | compound          | array                   | "`table`_" (with integer keys) | ``[1, 2, 3, 4, 5]``          |
    +-------------------+-------------------------+--------------------------------+------------------------------+
    | compound          | array                   | tuple ("`cdata`_")             | ``[12345, 'A B C']``         |
    +-------------------+-------------------------+--------------------------------+------------------------------+

.. NOTE::

   MsgPack values have variable lengths.
   So, for example, the smallest number requires only one byte, but the largest number
   requires nine bytes.

.. _nil: http://www.lua.org/pil/2.1.html
.. _boolean: http://www.lua.org/pil/2.2.html
.. _string: http://www.lua.org/pil/2.4.html
.. _number: http://www.lua.org/pil/2.3.html
.. _table: http://www.lua.org/pil/2.5.html
.. _cdata: http://luajit.org/ext_ffi.html#call

.. _index_box_field_type_details:

********************************************************
Field Type Details
********************************************************

.. _index-box_nil:

**nil**. In Lua a nil type has only one possible value, also called *nil*
(which Tarantool displays as ``null`` when using the default
:ref:`YAML <interactive_console>` format).
Nils may be compared to values of any types with == (is-equal)
or ~= (is-not-equal), but other comparison operations will not work.
Nils may not be used in Lua tables; the workaround is to use
:ref:`box.NULL <box-null>` because ``nil == box.NULL`` is true.
Example: ``nil``.

.. _index-box_boolean:

**boolean**. A boolean is either ``true`` or ``false``.
Example: ``true``.

.. _index-box_integer:

**integer**. The Tarantool integer type is for integers between
-9223372036854775808 and 18446744073709551615, which is about 18 quintillion.
This corresponds to number in Lua and to integer in MsgPack.
Example: ``-2^63``.

.. _index-box_unsigned:

**unsigned**. The Tarantool unsigned type is for integers between
0 and 18446744073709551615,. So it is a subset of integer.
Example: ``123456``.

.. _index-box_double:

**double**. The double field type exists
mainly so that there will be an equivalent to Tarantool/SQL's
:ref:`DOUBLE data type <sql_data_type_double>`.
In `msgpuck.h <https://github.com/rtsisyk/msgpuck>`_ (Tarantool's interface to MsgPack)
the storage type is MP_DOUBLE and the size of the encoded value is always 9 bytes.
In Lua, 'double' fields can only contain non-integer numeric values and
cdata values with double floating-point numbers.
Examples: ``1.234``, ``-44``, ``1.447e+44``. |br|
To avoid using the wrong kind of values inadvertently, use
``ffi.cast()`` when searching or changing 'double' fields.
For example, instead of
:samp:`{space_object}:insert`:code:`{`:samp:`{value}`:code:`}`
say
``ffi = require('ffi') ...``
:samp:`{space_object}:insert`:code:`({ffi.cast('double',`:samp:`{value}`:code:`)})`.
Example:

.. code-block:: none

    s = box.schema.space.create('s', {format = {{'d', 'double'}}})
    s:create_index('ii')
    s:insert({1.1})
    ffi = require('ffi')
    s:insert({ffi.cast('double', 1)})
    s:insert({ffi.cast('double', tonumber('123'))})
    s:select(1.1)
    s:select({ffi.cast('double', 1)})

Arithmetic with cdata 'double' will not work reliably, so
for Lua it is better to use the 'number' type.
This warning does not apply for Tarantool/SQL because
Tarantool/SQL does
:ref:`implicit casting <sql_data_type_conversion>`.

.. _index-box_number:

**number**. In Lua a number is double-precision floating-point, but a Tarantool
'number' field may have both
integer and floating-point values. Tarantool will try to store a Lua number as
floating-point if the value contains a decimal point or is very large
(greater than 100 trillion = 1e14), otherwise Tarantool will store it as an integer.
To ensure that even very large numbers are stored as integers, use the
:ref:`tonumber64 <other-tonumber64>` function, or the LL (Long Long) suffix,
or the ULL (Unsigned Long Long) suffix.
Here are examples of numbers using regular notation, exponential notation,
the ULL suffix and the ``tonumber64`` function:
``-55``, ``-2.7e+20``, ``100000000000000ULL``, ``tonumber64('18446744073709551615')``.

.. _index-box_decimal:

**decimal**. The Tarantool decimal type is stored as a MsgPack ext (Extension).
Values with the decimal type are not floating-point values although
they may contain decimal points.
They are exact with up to 38 digits of precision.
Example: a value returned by a function in the :ref:`decimal <decimal>` module.

.. _index-box_string:

**string**. A string is a variable-length sequence of bytes, usually represented with
alphanumeric characters inside single quotes. In both Lua and MsgPack, strings
are treated as binary data, with no attempts to determine a string's
character set or to perform any string conversion -- unless there is an optional
:ref:`collation <index-collation>`.
So, usually, string sorting and comparison are done byte-by-byte, without any special
collation rules applied.
(Example: numbers are ordered by their point on the number line, so 2345 is
greater than 500; meanwhile, strings are ordered by the encoding of the first
byte, then the encoding of the second byte, and so on, so ``'2345'`` is less than ``'500'``.)
Example: ``'A, B, C'``.

.. _index-box_bin:

**bin**. A bin (binary) value is not directly supported by Lua but there is
a Tarantool type ``varbinary`` which is encoded as MsgPack binary.
For an (advanced) example showing how to insert varbinary into a database,
see the Cookbook Recipe for :ref:`ffi_varbinary_insert <cookbook-ffi_varbinary_insert>`.
Example: ``"\65 \66 \67"``.

.. _index-box_uuid:

**uuid**. The Tarantool uuid type is stored as a MsgPack ext (Extension).
Values with the uuid type are
:ref:`Universally unique identifiers <uuid-module>`. |br|
Example: 64d22e4d-ac92-4a23-899a-e5934af5479.

.. _index-box_array:

**array**. An array is represented in Lua with ``{...}`` (`braces`_).
Examples: as lists of numbers representing points in a geometric figure:
``{10, 11}``, ``{3, 5, 9, 10}``.

**table**. Lua tables with string keys are stored as MsgPack maps;
Lua tables with integer keys starting with 1 are stored as MsgPack arrays.
Nils may not be used in Lua tables; the workaround is to use
:ref:`box.NULL <box-null>`.
Example: a ``box.space.tester:select()`` request will return a Lua table.

**tuple**. A tuple is a light reference to a MsgPack array stored in the database.
It is a special type (cdata) to avoid conversion to a Lua table on retrieval.
A few functions may return tables with multiple tuples. For tuple examples,
see :ref:`box.tuple <box_tuple>`.

.. _index-box_scalar:

**scalar**. Values in a scalar field can be boolean or integer or unsigned or double
or number or decimal or string or varbinary -- but not array or map or tuple.
Examples: ``true``, ``1``, ``'xxx'``.

.. _index-box_any:

**any**. Values in an any field can be boolean or integer or unsigned or double
or number or decimal or string or varbinary -- or array or map or tuple.
Examples: ``true``, ``1``, ``'xxx'``, ``{box.NULL, 0}``.

Examples of insert requests with different field types:

.. code-block:: tarantoolsession

    tarantool> box.space.K:insert{1,nil,true,'A B C',12345,1.2345}
    ---
    - [1, null, true, 'A B C', 12345, 1.2345]
    ...
    tarantool> box.space.K:insert{2,{['a']=5,['b']=6}}
    ---
    - [2, {'a': 5, 'b': 6}]
    ...
    tarantool> box.space.K:insert{3,{1,2,3,4,5}}
    ---
    - [3, [1, 2, 3, 4, 5]]
    ...

.. _braces: https://www.lua.org/pil/11.1.html

.. _index-box_indexed-field-types:

********************************************************
Indexed field types
********************************************************

Indexes restrict values which Tarantool may store with MsgPack. This is why,
for example, ``'unsigned'`` and ``'integer'`` are different field types although
in MsgPack they are both stored as integer values -- an ``'unsigned'`` index
contains only *non-negative* integer values while an ``‘integer’`` index contains *any*
integer values. 

Here again are the field types described in
:ref:`Field Type Details <index_box_field_type_details>`, and the index types they can fit in.
The default field type is ``'unsigned'`` and the default index type is TREE.
Although ``'nil'`` is not a legal indexed field type, indexes may contain `nil`
:ref:`as a non-default option <box_space-is_nullable>`.
Full information is in section
:ref:`Details about index field types <details_about_index_field_types>`.

.. container:: table

    .. rst-class:: left-align-column-1
    .. rst-class:: left-align-column-2
    .. rst-class:: left-align-column-3
    .. rst-class:: top-align-column-1

    .. tabularcolumns:: |\Y{0.2}|\Y{0.4}|\Y{0.2}|\Y{0.2}|

    +--------------------------------+-------------------------------------------+--------------------------------------+
    | Field type name string         | Field type |br|                           | Index type                           |
    +================================+===========================================+======================================+
    | ``'boolean'``                  | :ref:`boolean <index-box_boolean>`        | :ref:`TREE or HASH <box_index-type>` |
    |                                |                                           |                                      |
    +--------------------------------+-------------------------------------------+--------------------------------------+
    | ``'integer'``                  | :ref:`integer <index-box_integer>`        | TREE or HASH                         |
    | (may also be called ‘int’)     | which may include unsigned values         |                                      |
    |                                |                                           |                                      |
    |                                |                                           |                                      |
    +--------------------------------+-------------------------------------------+--------------------------------------+
    | ``'unsigned'``                 | :ref:`unsigned <index-box_unsigned>`      | TREE, BITSET or HASH                 |
    | (may also be called ‘uint’     |                                           |                                      |
    | or ‘num’, but ‘num’ is         |                                           |                                      |
    | deprecated)                    |                                           |                                      |
    +--------------------------------+-------------------------------------------+--------------------------------------+
    | ``'double'``                   | :ref:`double <index-box_double>`          | TREE or HASH                         |
    +--------------------------------+-------------------------------------------+--------------------------------------+
    | ``'number'``                   | :ref:`number <index-box_number>`          | TREE or HASH                         |
    |                                | which may include                         |                                      |
    |                                | :ref:`integer <index-box_integer>`        |                                      |
    |                                | or :ref:`double <index-box_double>`       |                                      |
    |                                | values                                    |                                      |
    +--------------------------------+-------------------------------------------+--------------------------------------+
    | ``'decimal'``                  | :ref:`decimal <index-box_decimal>`        | TREE or HASH                         |
    +--------------------------------+-------------------------------------------+--------------------------------------+
    | ``'string'``                   | :ref:`string <index-box_string>`          | TREE, BITSET or HASH                 |
    | (may also be called ``‘str’``) |                                           |                                      |
    +--------------------------------+-------------------------------------------+--------------------------------------+
    | ``'varbinary'``                | :ref:`varbinary <index-box_bin>`          | TREE, HASH or BITSET                 |
    |                                |                                           | (since version 2.7)                  |
    +--------------------------------+-------------------------------------------+--------------------------------------+
    | ``'uuid'``                     | :ref:`uuid <index-box_uuid>`              | TREE or HASH                         |
    +--------------------------------+-------------------------------------------+--------------------------------------+
    | ``'array'``                    | :ref:`array <index-box_array>`            | :ref:`RTREE <box_index-rtree>`       |
    +--------------------------------+-------------------------------------------+--------------------------------------+
    | ``'scalar'``                   | may include :ref:`nil <index-box_nil>`    | TREE or HASH                         |
    |                                | or :ref:`boolean <index-box_boolean>`     |                                      |
    |                                | or :ref:`integer <index-box_integer>`     |                                      |
    |                                | or :ref:`unsigned <index-box_unsigned>`   |                                      |
    |                                | or :ref:`number <index-box_number>`       |                                      |
    |                                | or :ref:`decimal <index-box_decimal>`     |                                      |
    |                                | or :ref:`string <index-box_string>`       |                                      |
    |                                | or :ref:`varbinary <index-box_bin>`       |                                      |
    |                                | values                                    |                                      |
    |                                |                                           |                                      |
    |                                | When a scalar field contains values of    |                                      |
    |                                | different underlying types, the key order |                                      |
    |                                | is: nils, then booleans, then numbers,    |                                      |
    |                                | then strings, then varbinaries.           |                                      | 
    +--------------------------------+-------------------------------------------+--------------------------------------+

.. _index-collation:

--------------------------------------------------------------------------------
Collations
--------------------------------------------------------------------------------

By default, when Tarantool compares strings, it uses what we call a
**"binary" collation**. The only consideration here is the numeric value
of each byte in the string. Therefore, if the string is encoded
with ASCII or UTF-8, then ``'A' < 'B' < 'a'``, because the encoding of ``'A'``
(what used to be called the "ASCII value") is 65, the encoding of
``'B'`` is 66, and the encoding of ``'a'`` is 98. Binary collation is best
if you prefer fast deterministic simple maintenance and searching
with Tarantool indexes.

But if you want the ordering that you see in phone books and dictionaries,
then you need Tarantool's optional collations, such as ``unicode`` and
``unicode_ci``, which allow for ``'a' < 'A' < 'B'`` and ``'a' = 'A' < 'B'``
respectively.

**The unicode and unicode_ci optional collations** use the ordering according to the
`Default Unicode Collation Element Table (DUCET) <http://unicode.org/reports/tr10/#Default_Unicode_Collation_Element_Table>`_
and the rules described in
`Unicode® Technical Standard #10 Unicode Collation Algorithm (UTS #10 UCA) <http://unicode.org/reports/tr10>`_.
The only difference between the two collations is about
`weights <https://unicode.org/reports/tr10/#Weight_Level_Defn>`_:

* ``unicode`` collation observes L1 and L2 and L3 weights (strength = 'tertiary'),
* ``unicode_ci`` collation observes only L1 weights (strength = 'primary'), so for example 'a' = 'A' = 'á' = 'Á'.

As an example, take some Russian words:

.. code-block:: text

    'ЕЛЕ'
    'елейный'
    'ёлка'
    'еловый'
    'елозить'
    'Ёлочка'
    'ёлочный'
    'ЕЛь'
    'ель'

...and show the difference in ordering and selecting by index:

* with ``unicode`` collation:

  .. code-block:: tarantoolsession

      tarantool> box.space.T:create_index('I', {parts = {{field = 1, type = 'str', collation='unicode'}}})
      ...
      tarantool> box.space.T.index.I:select()
      ---
      - - ['ЕЛЕ']
        - ['елейный']
        - ['ёлка']
        - ['еловый']
        - ['елозить']
        - ['Ёлочка']
        - ['ёлочный']
        - ['ель']
        - ['ЕЛь']
      ...
      tarantool> box.space.T.index.I:select{'ЁлКа'}
      ---
      - []
      ...

* with ``unicode_ci`` collation:

  .. code-block:: tarantoolsession

      tarantool> box.space.T:create_index('I', {parts = {{field = 1, type ='str', collation='unicode_ci'}}})
      ...
      tarantool> box.space.S.index.I:select()
      ---
      - - ['ЕЛЕ']
        - ['елейный']
        - ['ёлка']
        - ['еловый']
        - ['елозить']
        - ['Ёлочка']
        - ['ёлочный']
        - ['ЕЛь']
      ...
      tarantool> box.space.S.index.I:select{'ЁлКа'}
      ---
      - - ['ёлка']
      ...


In all, collation involves much more than these simple examples of
upper case / lower case and accented / unaccented equivalence in alphabets.
We also consider variations of the same character, non-alphabetic writing systems,
and special rules that apply for combinations of characters.

For English: use "unicode" and "unicode_ci".
For Russian: use "unicode" and "unicode_ci" (although a few Russians might
prefer the Kyrgyz collation which says Cyrillic letters 'Е' and 'Ё' are the
same with level-1 weights).
For Dutch, German (dictionary), French, Indonesian, Irish,
Italian, Lingala, Malay, Portuguese, Southern Soho, Xhosa, or Zulu:
"unicode" and "unicode_ci" will do.

**The tailored optional collations**: For other languages, Tarantool supplies tailored collations for every
modern language that has more than a million native speakers, and
for specialized situations such as the difference between dictionary
order and telephone book order.
To see a complete list say ``box.space._collation:select()``.
The tailored collation names have the form
unicode_[language code]_[strength] where language code is a standard
2-character or 3-character language abbreviation, and strength is s1
for "primary strength" (level-1 weights), s2 for "secondary", s3 for "tertiary".
Tarantool uses the same language codes as the ones in the "list of tailorable locales" on man pages of
`Ubuntu <http://manpages.ubuntu.com/manpages/bionic/man3/Unicode::Collate::Locale.3perl.html>`_ and
`Fedora <http://www.polarhome.com/service/man/?qf=Unicode%3A%3ACollate%3A%3ALocale&af=0&tf=2&of=Fedora>`_.
Charts explaining the precise differences from DUCET order are
in the
`Common Language Data Repository <https://unicode.org/cldr/charts/30/collation>`_.

.. _index-box_sequence:

--------------------------------------------------------------------------------
Sequences
--------------------------------------------------------------------------------

A **sequence** is a generator of ordered integer values.

As with spaces and indexes, you should specify the sequence **name**, and let
Tarantool come up with a unique **numeric identifier** ("sequence id").

As well, you can specify several options when creating a new sequence.
The options determine what value will be generated whenever the sequence is used.

.. _index-box_sequence-options:

********************************************************
Options for ``box.schema.sequence.create()``
********************************************************

.. container:: table

    .. rst-class:: left-align-column-1
    .. rst-class:: left-align-column-2
    .. rst-class:: left-align-column-3
    .. rst-class:: left-align-column-4
    .. rst-class:: top-align-column-1

    .. tabularcolumns:: |\Y{0.2}|\Y{0.4}|\Y{0.2}|\Y{0.2}|

    +----------------------------+----------------------------------+----------------------+--------------------+
    | Option name                | Type and meaning                 | Default              | Examples           |
    +============================+==================================+======================+====================+
    | **start**                  | Integer. The value to generate   | 1                    | start=0            |
    |                            | the first time a sequence is     |                      |                    |
    |                            | used                             |                      |                    |
    +----------------------------+----------------------------------+----------------------+--------------------+
    | **min**                    | Integer. Values smaller than     | 1                    | min=-1000          |
    |                            | this cannot be generated         |                      |                    |
    +----------------------------+----------------------------------+----------------------+--------------------+
    | **max**                    | Integer. Values larger than      | 9223372036854775807  | max=0              |
    |                            | this cannot be generated         |                      |                    |
    +----------------------------+----------------------------------+----------------------+--------------------+
    | **cycle**                  | Boolean. Whether to start again  | false                | cycle=true         |
    |                            | when values cannot be generated  |                      |                    |
    +----------------------------+----------------------------------+----------------------+--------------------+
    | **cache**                  | Integer. The number of values    | 0                    | cache=0            |
    |                            | to store in a cache              |                      |                    |
    +----------------------------+----------------------------------+----------------------+--------------------+
    | **step**                   | Integer. What to add to the      | 1                    | step=-1            |
    |                            | previous generated value, when   |                      |                    |
    |                            | generating a new value           |                      |                    |
    +----------------------------+----------------------------------+----------------------+--------------------+
    | **if_not_exists**          | Boolean. If this is true and     | false                | if_not_exists=true |
    |                            | a sequence with this name exists |                      |                    |
    |                            | already, ignore other options    |                      |                    |
    |                            | and use the existing values      |                      |                    |
    +----------------------------+----------------------------------+----------------------+--------------------+

Once a sequence exists, it can be altered, dropped, reset, forced to generate
the next value, or associated with an index.

For an initial example, we generate a sequence named 'S'.

.. code-block:: tarantoolsession

    tarantool> box.schema.sequence.create('S',{min=5, start=5})
    ---
    - step: 1
      id: 5
      min: 5
      cache: 0
      uid: 1
      max: 9223372036854775807
      cycle: false
      name: S
      start: 5
    ...

The result shows that the new sequence has all default values,
except for the two that were specified, ``min`` and ``start``.

Then we get the next value, with the ``next()`` function.

.. code-block:: tarantoolsession

    tarantool> box.sequence.S:next()
    ---
    - 5
    ...

The result is the same as the start value. If we called ``next()``
again, we would get 6 (because the previous value plus the
step value is 6), and so on.

Then we create a new table, and say that its primary key may be
generated from the sequence.

.. code-block:: tarantoolsession

    tarantool> s=box.schema.space.create('T')
    ---
    ...
    tarantool> s:create_index('I',{sequence='S'})
    ---
    - parts:
      - type: unsigned
        is_nullable: false
        fieldno: 1
      sequence_id: 1
      id: 0
      space_id: 520
      unique: true
      type: TREE
      sequence_fieldno: 1
      name: I
    ...
    ---
    ...

Then we insert a tuple, without specifying a value for the primary key.

.. code-block:: tarantoolsession

    tarantool> box.space.T:insert{nil,'other stuff'}
    ---
    - [6, 'other stuff']
    ...

The result is a new tuple where the first field has a value of 6.
This arrangement, where the system automatically generates the
values for a primary key, is sometimes called "auto-incrementing"
or "identity".

For syntax and implementation details, see the reference for
:doc:`box.schema.sequence </reference/reference_lua/box_schema_sequence>`.

.. _index-box_persistence:

--------------------------------------------------------------------------------
Persistence
--------------------------------------------------------------------------------

In Tarantool, updates to the database are recorded in the so-called
:ref:`write ahead log (WAL) <internals-wal>` files. This ensures data persistence.
When a power outage occurs or the Tarantool instance is killed incidentally,
the in-memory database is lost. In this situation, WAL files are used
to restore the data. Namely, Tarantool reads the WAL files and redoes
the requests (this is called the "recovery process"). You can change
the timing of the WAL writer, or turn it off, by setting
:ref:`wal_mode <cfg_binary_logging_snapshots-wal_mode>`.

Tarantool also maintains a set of :ref:`snapshot files <internals-snapshot>`.
These files contain an on-disk copy of the entire data set for a given moment.
Instead of reading every WAL file since the databases were created, the recovery
process can load the latest snapshot file and then read only those WAL files
that were produced after the snapshot file was made. After checkpointing, old
WAL files can be removed to free up space.

To force immediate creation of a snapshot file, you can use Tarantool's
:doc:`box.snapshot() </reference/reference_lua/box_snapshot>` request. To enable automatic creation
of snapshot files, you can use Tarantool's
:ref:`checkpoint daemon <book_cfg_checkpoint_daemon>`. The checkpoint
daemon sets intervals for forced checkpoints. It makes sure that the states
of both memtx and vinyl storage engines are synchronized and saved to disk,
and automatically removes old WAL files.

Snapshot files can be created even if there is no WAL file.

.. NOTE::

     The memtx engine makes only regular checkpoints with the interval set in
     :ref:`checkpoint daemon <book_cfg_checkpoint_daemon>` configuration.

     The vinyl engine runs checkpointing in the background at all times.

See the :ref:`Internals <internals-data_persistence>` section for more details
about the WAL writer and the recovery process.

.. _index-box_operations:

--------------------------------------------------------------------------------
Operations
--------------------------------------------------------------------------------

.. _index-box_data-operations:

********************************************************
Data operations
********************************************************

The basic data operations supported in Tarantool are:

* five data-manipulation operations (INSERT, UPDATE, UPSERT, DELETE, REPLACE), and
* one data-retrieval operation (SELECT).

All of them are implemented as functions in :ref:`box.space <box_space>` submodule.

**Examples:**

* :ref:`INSERT <box_space-insert>`: Add a new tuple to space 'tester'.

  The first field, field[1], will be 999 (MsgPack type is `integer`).

  The second field, field[2], will be 'Taranto' (MsgPack type is `string`).

  .. code-block:: tarantoolsession

      tarantool> box.space.tester:insert{999, 'Taranto'}

* :ref:`UPDATE <box_space-update>`: Update the tuple, changing field field[2].

  The clause "{999}", which has the value to look up in the index of the tuple's
  primary-key field, is mandatory, because ``update()`` requests must always have
  a clause that specifies a unique key, which in this case is field[1].

  The clause "{{'=', 2, 'Tarantino'}}" specifies that assignment will happen to
  field[2] with the new value.

  .. code-block:: tarantoolsession

      tarantool> box.space.tester:update({999}, {{'=', 2, 'Tarantino'}})

* :ref:`UPSERT <box_space-upsert>`: Upsert the tuple, changing field field[2]
  again.

  The syntax of ``upsert()`` is similar to the syntax of ``update()``. However,
  the execution logic of these two requests is different.
  UPSERT is either UPDATE or INSERT, depending on the database's state.
  Also, UPSERT execution is postponed until after transaction commit, so, unlike
  ``update()``, ``upsert()`` doesn't return data back.

  .. code-block:: tarantoolsession

      tarantool> box.space.tester:upsert({999, 'Taranted'}, {{'=', 2, 'Tarantism'}})

* :ref:`REPLACE <box_space-replace>`: Replace the tuple, adding a new field.

  This is also possible with the ``update()`` request, but the ``update()``
  request is usually more complicated.

  .. code-block:: tarantoolsession

      tarantool> box.space.tester:replace{999, 'Tarantella', 'Tarantula'}

* :ref:`SELECT <box_space-select>`: Retrieve the tuple.

  The clause "{999}" is still mandatory, although it does not have to mention
  the primary key.

  .. code-block:: tarantoolsession

      tarantool> box.space.tester:select{999}

* :ref:`DELETE <box_space-delete>`: Delete the tuple.

  In this example, we identify the primary-key field.

  .. code-block:: tarantoolsession

      tarantool> box.space.tester:delete{999}

Summarizing the examples:

* Functions ``insert`` and ``replace`` accept a tuple
  (where a primary key comes as part of the tuple).
* Function ``upsert`` accepts a tuple
  (where a primary key comes as part of the tuple),
  and also the update operations to execute.
* Function ``delete`` accepts a full key of any unique index
  (primary or secondary).
* Function ``update`` accepts a full key of any unique index
  (primary or secondary),
  and also the operations to execute.
* Function ``select`` accepts any key: primary/secondary, unique/non-unique,
  full/partial.

See reference on ``box.space`` for more
:ref:`details on using data operations <box_space-operations-detailed-examples>`.

.. NOTE::

   Besides Lua, you can use
   :ref:`Perl, PHP, Python or other programming language connectors <index-box_connectors>`.
   The client server protocol is open and documented.
   See this :ref:`annotated BNF <box_protocol-iproto_protocol>`.


********************************************************
Complexity factors
********************************************************

In reference for :ref:`box.space <box_space>` and
:doc:`/reference/reference_lua/box_index`
submodules, there are notes about which complexity factors might affect the
resource usage of each function.

.. container:: table

    .. rst-class:: left-align-column-1
    .. rst-class:: left-align-column-2

    .. tabularcolumns:: |\Y{0.2}|\Y{0.8}|

    +-------------------+----------------------------------------------------------+
    | Complexity        | Effect                                                   |
    | factor            |                                                          |
    +===================+==========================================================+
    | Index size        | The number of index keys is the same as the number       |
    |                   | of tuples in the data set. For a TREE index, if          |
    |                   | there are more keys, then the lookup time will be        |
    |                   | greater, although of course the effect is not            |
    |                   | linear. For a HASH index, if there are more keys,        |
    |                   | then there is more RAM used, but the number of           |
    |                   | low-level steps tends to remain constant.                |
    +-------------------+----------------------------------------------------------+
    | Index type        | Typically, a HASH index is faster than a TREE index      |
    |                   | if the number of tuples in the space is greater          |
    |                   | than one.                                                |
    +-------------------+----------------------------------------------------------+
    | Number of indexes | Ordinarily, only one index is accessed to retrieve       |
    | accessed          | one tuple. But to update the tuple, there must be N      |
    |                   | accesses if the space has N different indexes.           |
    |                   |                                                          |
    |                   | Note re storage engine: Vinyl optimizes away such        |
    |                   | accesses if secondary index fields are unchanged by      |
    |                   | the update. So, this complexity factor applies only to   |
    |                   | memtx, since it always makes a full-tuple copy on every  |
    |                   | update.                                                  |
    +-------------------+----------------------------------------------------------+
    | Number of tuples  | A few requests, for example SELECT, can retrieve         |
    | accessed          | multiple tuples. This factor is usually less             |
    |                   | important than the others.                               |
    +-------------------+----------------------------------------------------------+
    | WAL settings      | The important setting for the write-ahead log is         |
    |                   | :ref:`wal_mode <cfg_binary_logging_snapshots-wal_mode>`. |
    |                   | If the setting causes no writing or                      |
    |                   | delayed writing, this factor is unimportant. If the      |
    |                   | setting causes every data-change request to wait         |
    |                   | for writing to finish on a slow device, this factor      |
    |                   | is more important than all the others.                   |
    +-------------------+----------------------------------------------------------+
