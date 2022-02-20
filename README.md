# DatetimeAdvanced
Replacement for FieldtypeDatetime with support for subfield selectors like year, month, hour or day of year.

Allows searching for individual components of datetime fields in
standard subfield syntax.

Also supports subfields in Lister (and probably Lister Pro, though confirmation would be appreciated).

# Status

**ATTENTION: THIS MODULES IS NO LONGER COMPATIBLE WITH LATEST VERSIONS OF PROCESSWIRE 3!**

Unfortunately, it will take until the end of the year until I can take care of the necessary changes.

## Possible subfields

- day
- month
- year (4-digit)
- hour (0..23)
- minutes
- seconds
- day_of_week (0..6, 0 = Sunday)
- day_of_year (0..365)
- week_of_year (1..53)

- date (date formatted as yyyy-mm-dd)
- time (time formatted as HH:MM:SS)

## Also installs
WireDT: wrapper class for datetime fields, necessary to support
subfield syntax when filtering PageArrays. FieldtypeDatetimeAdvanced won't work
without it.

## Requirements

Requires [timezone support](http://dev.mysql.com/doc/refman/5.7/en/time-zone-support.html) to be enabled in MySQL.

## Usage

- Unzip the module files (downloadable here through the green button) into site/modules
- Install FieldtypeDatetimeAdvanced
- Create a field with type DatetimeAdvanced and add it to your template
- Populate the field with values in some pages
- Start searching/filtering

## Examples

Simple selector by year:
```
$pagelist = $pages->find("mydatefield.year=2016");
```

Simple selector by month:
```
$maypages = $pagelist->filter("mydatefield.month=5");
```

Searching for a birthday:
```
$birthday = date('Y-m-d');
$tocelebrate = $pages->find("birthday.date=$birthday");
foreach($tocelebrate as $person) {
	echo "Happy birthday {$person->name}!";
}
```

All pages with a date in the next 7 days:
```
$start = date('z');
$end = $start + 7;
$sevendays = $pages->find("mydatefield.day_of_year>=$start, mydatefield.day_of_year<$end");
```

Directly accessing a date subfield:
```
$blogentry = $pages->get('blog-entry-1');
echo $blogentry->title . "(" . $blogentry->publishdate->year . ")";
```

strftime() and date() shorthand:
```
echo $blogentry->publishdate->strftime("%Y-%m-%d %H:%M:%S") . PHP_EOL;
echo $blogentry->publishdate->date("Y-m-d H:i:s") . PHP_EOL;
```

## Adding your own custom "subfields"

You can add your own custom subfields with formats through a hook after WireDT::getOperators.
Note, however, that the hook has to be done very early, so site/ready.php is too late. Use
site/init.php instead.

The method returns an associative array in the form:
```php

array(
	"subfield"		=> array("SQL-Format", "PHP-Format"),
	"day"			=>	array("d", "d"),
	"month"			=>	array("c", "m"),
	"year"			=>	array("Y", "Y"),
	"hour"			=>	array("H", "H"),
	"minutes"		=>	array("i", "i"),
	"seconds"		=>	array("s", "s"),
	"day_of_week"		=>	array("w", "w"),
	"day_of_year"		=>	array("j", "z"),
	"week_of_year"		=>	array("v", "W"),
	"date"			=>	array("%Y-%m-%d", "Y-m-d"),
	"time"			=>	array("T", "H:i:s"),
);

```

You can specify your own format, e.g. a combination of year and month that we call "ym" for
obvious reasons:

```php

wire()->addHookAfter("WireDT::getOperators", function(HookEvent $event) {
	$ops = $event->return;
	$ops["ym"] = array("Ym", "Ym");	// Formats for SQL and PHP are the same
	$event->return = $ops;
});
```

## A little bit of prose

This module came to be due to the repeated need to filter pages by a date span, like pages
from a certain year or in the last seven days. All the calculating back and forth to get the
correct timestamps felt a bit of an overhead when there are perfectly easy functions to get
individual date components in both PHP and MySQL.

If you are dealing with high numbers of pages (I'm talking five digits up), doing the calculation
in PHP and running queries with plain timestamps will still be the way to go performance-wise.
Filling additional fields with just the information you filter on (like month and year) when you
save a page might then also be a good approach.

### Timezone support

A prerequisite for installing this module is timezone support in MySQL. In most installations,
MySQL is installed without timezone data, thus not adjusting for possible timezone differences
between the server and clients. So, if the configured timezone in the server OS and PHP differ,
the return values of MySQL's date and time functions differ from what PHP expects, rendering
searches for date or time components unreliable at best and likely plain wrong.

This is not a problem with regular Datetime fields in ProcessWire, since all its values are
stored in timestamp format and all conversions are done by PHP, conforming to the timezone
configured there.

To use advanced selectors, though, MySQL needs to be aware of the timezone PHP is using to
return the correct components, since it stores timestamp values in UTC (GMT).

To enable timezone support, the necessary timezone data nees to be installed and MySQL
needs to be restarted. On Unix'ish systems, MySQL comes with a tool to generate the
correct timezone data from data already present in the OS, while other systems may require
you to download database files or an SQL script. See
[the MySQL documentation](http://dev.mysql.com/doc/refman/5.7/en/time-zone-support.html) for details.

### Note

These subfield selectors can unfortunately not be used for the builtin "created" and "modified" fields.

## License

Licensed under Mozilla Public License v2.0. See file LICENSE for details.
