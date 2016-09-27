# DatetimeAdvanced
Replacement for FieldtypeDatetime with support for subfield selectors like year, month, hour or day of year.

Allows searching for individual components of datetime fields in
standard subfield syntax.

## Status

Alpha. Please provide feedback.

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

## Also installs
WireDT: wrapper class for datetime fields, necessary to support
subfield syntax when filtering PageArrays. FieldtypeDatetimeAdvanced won't work
without it.

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

## A little bit of prose

This module came to be due to the repeated need to filter pages by a date span, like pages
from a certain year or in the last seven days. All the calculating back and forth to get the
correct timestamps felt a bit of an overhead when there are perfectly easy functions to get
individual date components in both PHP and MySQL.

If you are dealing with high numbers of pages (I'm talking five digits up), doing the calculation
in PHP and running queries with plain timestamps will still be the way to go performance-wise.
Filling additional fields with just the information you filter on (like month and year) when you
save a page might then also be a good approach.

### Note

These subfield selectors can unfortunately not be used for the builtin created and modified fields.

## License

Licensed under Mozilla Public License v2.0. See file LICENSE for details.
