# Spout

[![Latest Stable Version](https://poser.pugx.org/box/spout/v/stable)](https://packagist.org/packages/box/spout)
[![Project Status](http://opensource.box.com/badges/active.svg)](http://opensource.box.com/badges)
[![Build Status](https://travis-ci.org/box/spout.svg?branch=master)](http://travis-ci.org/box/spout)
[![Code Coverage](https://scrutinizer-ci.com/g/box/spout/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/box/spout/?branch=master)
[![Total Downloads](https://poser.pugx.org/box/spout/downloads)](https://packagist.org/packages/box/spout)
[![License](https://poser.pugx.org/box/spout/license)](https://packagist.org/packages/box/spout)

Spout is a PHP library to read and write CSV and XLSX files, in a fast and scalable way.
Contrary to other file readers or writers, it is capable of processing very large files while keeping the memory usage really low (less than 10MB).

Join the community and come discuss about Spout: [![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/box/spout?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

## Installation

### Composer (recommended)

Spout can be installed directly from [Composer](https://getcomposer.org/).

Add "box/spout" as a dependency in your project's composer.json file:
```json
"require": {
    "box/spout": "~2.0"
}
```

Then run the install command from Composer:
```
php composer.phar install
```

### Manual installation

If you can't use Composer, no worries! You can still install Spout manually.

> Before starting, make sure your system meets the [requirements](#requirements).

1. Download the source code from the [Releases page](https://github.com/box/spout/releases)
2. Extract the downloaded content into your project.
3. Add this code to the top controller (index.php) or wherever it may be more appropriate:
```php
require_once '[PATH/TO]/src/Spout/Autoloader/autoload.php'; // don't forget to change the path!
```


## Requirements

* PHP version 5.4.0 or higher
* PHP extension `php_zip` enabled
* PHP extension `php_xmlreader` enabled
* PHP extension `php_simplexml` enabled


## Basic usage

### Reader

#### How to read a CSV file?

```php
use Box\Spout\Reader\ReaderFactory;
use Box\Spout\Common\Type;

$reader = ReaderFactory::create(Type::CSV);
$reader->open($filePath);

foreach ($reader->getSheetIterator() as $sheet) {
    foreach ($sheet->getRowIterator() as $row) {
        // do stuff
    }
}

$reader->close();
```

#### How to read a XLSX file?

```php
use Box\Spout\Reader\ReaderFactory;
use Box\Spout\Common\Type;

$reader = ReaderFactory::create(Type::XLSX);
$reader->open($filePath);

foreach ($reader->getSheetIterator() as $sheet) {
    foreach ($sheet->getRowIterator() as $row) {
        // do stuff
    }
}

$reader->close();
```

If there are multiple sheets in the file, the reader will read through all of them sequentially.

### Writer

### How to create a CSV file?

```php
use Box\Spout\Writer\WriterFactory;
use Box\Spout\Common\Type;

$writer = WriterFactory::create(Type::CSV);
$writer->openToFile($filePath); // write data to a file or to a PHP stream
$writer->addRow($singleRow); // add a row at a time
$writer->close();
```

### How to create a XLSX file?

```php
use Box\Spout\Writer\WriterFactory;
use Box\Spout\Common\Type;

$writer = WriterFactory::create(Type::XLSX);
$writer->openToBrowser($fileName); // stream data directly to the browser
$writer->addRows($multipleRows); // add multiple rows at a time
$writer->close();
```

For XLSX files, the number of rows per sheet is limited to 1,048,576 (see [Office OpenXML specs](http://office.microsoft.com/en-us/excel-help/excel-specifications-and-limits-HP010073849.aspx)). By default, once this limit is reached, the writer will automatically create a new sheet and continue writing data into it.


## Advanced usage

### Configuring the CSV reader and writer

It is possible to configure both the CSV reader and writer to specify the field separator as well as the field enclosure:
```php
use Box\Spout\Reader\ReaderFactory;
use Box\Spout\Common\Type;

$reader = ReaderFactory::create(Type::CSV);
$reader->setFieldDelimiter('|');
$reader->setFieldEnclosure('@');
```

Additionally, if you need to read non UTF-8 files, you can specify the encoding of your file this way:
```php
$reader->setEncoding('UTF-16LE');
```

The writer always generate CSV files encoded in UTF-8, with a BOM.

### Configuring the XLSX writer

#### Row styling

It is possible to apply some formatting options to a row. Spout supports fonts as well as alignment styles.

```php
use Box\Spout\Common\Type;
use Box\Spout\Writer\WriterFactory;
use Box\Spout\Writer\Style\StyleBuilder;

$style = (new StyleBuilder())
           ->setFontBold()
           ->setFontSize(15)
           ->setFontColor(Color::BLUE)
           ->setShouldWrapText()
           ->build();

$writer = WriterFactory::create(Type::XLSX);
$writer->openToFile($filePath);

$writer->addRowWithStyle($singleRow, $style); // style will only be applied to this row
$writer->addRow($otherSingleRow); // no style will be applied
$writer->addRowsWithStyle($multipleRows, $style); // style will be applied to all given rows

$writer->close();
```

Unfortunately, Spout does not support all the possible formatting options yet. But you can find the most important ones:

Category  | Property      | API
----------|---------------|---------------------------------------
Font      | Bold          | `StyleBuilder::setFontBold()`
          | Italic        | `StyleBuilder::setFontItalic()`
          | Underline     | `StyleBuilder::setFontUnderline()`
          | Strikethrough | `StyleBuilder::setFontStrikethrough()`
          | Font name     | `StyleBuilder::setFontName('Arial')`
          | Font size     | `StyleBuilder::setFontSize(14)`
          | Font color    | `StyleBuilder::setFontSize(Color::BLUE)`<br>`StyleBuilder::setFontSize(Color::rgb(0, 128, 255))`
Alignment | Wrap text     | `StyleBuilder::setShouldWrapText()`


#### Strings storage

XLSX files support different ways to store the string values:
* Shared strings are meant to optimize file size by separating strings from the sheet representation and ignoring strings duplicates (if a string is used three times, only one string will be stored)
* Inline strings are less optimized (as duplicate strings are all stored) but is faster to process

In order to keep the memory usage really low, Spout does not optimize strings when using shared strings. It is nevertheless possible to use this mode.
```php
use Box\Spout\Writer\WriterFactory;
use Box\Spout\Common\Type;

$writer = WriterFactory::create(Type::XLSX);
$writer->setShouldUseInlineStrings(true); // default (and recommended) value
$writer->setShouldUseInlineStrings(false); // will use shared strings
```

> ##### Note on Apple Numbers and iOS support
>
> Apple's products (Numbers and the iOS previewer) don't support inline strings and display empty cells instead. Therefore, if these platforms need to be supported, make sure to use shared strings! 


#### New sheet creation

It is also possible to change the behavior of the writer when the maximum number of rows (1,048,576) have been written in the current sheet:
```php
use Box\Spout\Writer\WriterFactory;
use Box\Spout\Common\Type;

$writer = WriterFactory::create(Type::XLSX);
$writer->setShouldCreateNewSheetsAutomatically(true); // default value
$writer->setShouldCreateNewSheetsAutomatically(false); // will stop writing new data when limit is reached
```

#### Using custom temporary folder

Processing XLSX files require temporary files to be created. By default, Spout will use the system default temporary folder (as returned by sys_get_temp_dir()). It is possible to override this by explicitly setting it on the reader or writer:
```php
use Box\Spout\Writer\WriterFactory;
use Box\Spout\Common\Type;

$writer = WriterFactory::create(Type::XLSX);
$writer->setTempFolder($customTempFolderPath);
```

### Playing with XLSX sheets

When creating a XLSX file, it is possible to control in which sheet the data will be written to.
At any point, you can retrieve the current sheet and set the current sheet:
```php
$firstSheet = $writer->getCurrentSheet();
$writer->addRow($rowForSheet1); // writes the row to the first sheet

$newSheet = $writer->addNewSheetAndMakeItCurrent();
$writer->addRow($rowForSheet2); // writes the row to the new sheet

$writer->setCurrentSheet($firstSheet);
$writer->addRow($anotherRowForSheet1); // append the row to the first sheet
```

It is also possible to retrieve all the sheets currently created:
```php
$sheets = $writer->getSheets();
```

If you rely on the sheet's name in your application, you can access it and customize it this way:
```php
// Accessing the sheet name when reading
foreach ($reader->getSheetIterator() as $sheet) {
    $sheetName = $sheet->getName();
}

// Accessing the sheet name when writing
$sheet = $writer->getCurrentSheet();
$sheetName = $sheet->getName();

// Customizing the sheet name when writing
$sheet = $writer->getCurrentSheet();
$sheetName = $sheet->setName('My custom name');
``` 

> Please note that Excel has some restrictions on the sheet's name:
> * it must not be blank
> * it must not exceed 31 characters
> * it must not contain these characters: \ / ? * : [ or ]
> * it must not start or end with a single quote
> * it must be unique
>
> Handling these restrictions is the developer's responsibility. Spout does not try to automatically change the sheet's name, as one may rely on this name to be exactly what was passed in.


### Fluent interface

Because fluent interfaces are great, you can use them with Spout:
```php
use Box\Spout\Writer\WriterFactory;
use Box\Spout\Common\Type;

$writer = WriterFactory::create(Type::XLSX);
$writer->setTempFolder($customTempFolderPath)
       ->setShouldUseInlineStrings(true)
       ->openToFile($filePath)
       ->addRow($headerRow)
       ->addRows($dataRows)
       ->close();
```


## Running tests

On the `master` branch, only unit and functional tests are included. The performance requires very large files and have been excluded.
If you just want to check that everything is working as expected, executing the tests of the master branch is enough.

If you want to run performance tests, you will need to checkout the `perf-tests` branch. Multiple test suites can then be run, depending on the expected output:

* `phpunit` - runs the whole test suite (unit + functional + performance tests)
* `phpunit --testuite no-perf-tests` - only runs the unit and functional tests
* `phpunit --testuite perf-tests` - only runs the performance tests

For information, the performance tests take about one hour to run (processing 2 million rows files is not a quick thing).


## Frequently Asked Questions

#### How can Spout handle such large data sets and still use less than 10MB of memory?

When writing data, Spout is streaming the data to files, one or few lines at a time. That means that it only keeps in memory the few rows that it needs to write. Once written, the memory is freed.

Same goes with reading. Only one row at a time is stored in memory. A special technique is used to handle shared strings in XLSX, storing them - if needed - into several small temporary files that allows fast access.

#### How long does it take to generate a file with X rows?

Here are a few numbers regarding the performance of Spout:

|                                  | 2,000 rows (6,000 cells) | 200,000 rows (600,000 cells) | 2,000,000 rows (6,000,000 cells) |
| :------------------------------- | :----------------------: | :--------------------------: | :------------------------------: |
| Read CSV                         | < 1 second               | 4 seconds                    | 2-3 minutes                      |
| Write CSV                        | < 1 second               | 2 seconds                    | 2-3 minutes                      |
| Read XLSX (using inline strings) | < 1 second               | 35-40 seconds                | 18-20 minutes                    |
| Read XLSX (using shared strings) | 1 second                 | 1-2 minutes                  | 35-40 minutes                    |
| Write XLSX                       | 1 second                 | 20-25 seconds                | 8-10 minutes                     |

#### Does Spout support charts or formulas?

No. This is a compromise to keep memory usage low. Charts and formulas requires data to be kept in memory in order to be used.
So the larger the file would be, the more memory would be consumed, preventing your code to scale well.


## Support

Need to contact us directly? Email oss@box.com and be sure to include the name of this project in the subject.


## Copyright and License

Copyright 2015 Box, Inc. All rights reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
