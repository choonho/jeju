# Test of INI style document

## Create new INI file

edit test.ini
~~~text
[section1]
key1 = value1
key2 = value2

[section2]
key3 = value3
key4 = value4

[section3]
key5 = value5

~~~

## Update INI content

To update ini file, language format must be ini
<pre>
~~~ini
...
~~~
</pre>

* *value1* of section1 is changed *value1-1*.
* new section *section4* is added.
* *...* in the section is ignored.

edit test.ini

~~~ini
[section1]
key1 = value1-1
...
[section4]
key6 = value6
~~~

## Checkout result

This result must be:
<pre>
[section1]
key1 = value1-1
key2 = value2

[section2]
key3 = value3
key4 = value4

[section3]
key5 = value5

[section4]
key6 =value6
</pre>
