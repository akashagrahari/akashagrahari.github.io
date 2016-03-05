---
layout: post
title: Handling Excel Data with PHP
sub-title: Using PHPExcel, you can easily read, create and modify excel data
permalink: handling-excel-php
---

A couple of months ago during an internship, I was asked to tackle an inventory management problem, and develop efficient algorithms to develop replenishment models to keep the inventory levels low. For this task I had to parse a large number of excel files to get the initial population of data. Granted, PHP was certainly not the best choice for this, and even for a browser based application, I could certainly have a third party script and called it from PHP. I ended up using PHP though, since time was scarce.

<!--break-->

I came across this package called PHPExcel. You can check their codeplex page [here](https://phpexcel.codeplex.com).

In this article, I will introduce you to the set of classes contained in PHPExcel and what all can you accomplish with it through some examples.

You can go ahead and download the files from the link provided above.

Before you get started, you might probably have to do a *set_include_path* at the starting of the document to point the interpreter to the location of the PHPExcel Classes.

{% highlight js %}
set_include_path('#path');
{% endhighlight %}

You can do an ini_set and change the **include path** if you are working on a PHP version < 4.3.0. This should be followed by a -

{% highlight js %}
require_once 'PHPExcel.php';
{% endhighlight %}

Now you are ready to use the methods and classes provided in the PHPExcel package.

### Starting up — Creating your first excel file

You can now go ahead and make an object of the PHPExcel class. This object acts as a handle to read and write data from/to a file.

{% highlight js %}
$PHPExcelObject = new PHPExcel();
{% endhighlight %}

You can set the document properties like this —

{% highlight js %}
$PHPExcelObject ->getProperties()->setCreator("AA_isnowonline");
{% endhighlight %}

You can similarly set other properties like title, subject, description, last modified by, etc. by using similar function. The function name is generally “set” with the property name in Camel Case. You can obviously use method chaining to set several properties at once in a single statements as in —

{% highlight js %}
$PHPExcelObject ->getProperties()->setCreator("AA_isnowonline")
                               ->setTitle("Example Excel Document");
{% endhighlight %}

The indexes of sheets are given by users while creating new sheets. The index for the first sheet created is ‘0' by default.

So, if you want a particular sheet to open up (“Active”) while opening a document, you can use the following syntax -

{% highlight js %}
$PHPExcelObject ->setActiveSheetIndex(#index);
{% endhighlight %}

Creating new sheets can be achieved by-

{% highlight js %}
$PHPExcelObject ->createSheet(#index);
{% endhighlight %}

It is really important set values to individual cells after setting the desired sheet as active. I personally use a separate variable to store that pointer so that I don’t have to go to the trouble of writing while chaining methods.

You can either set cell values by Excel cell addresses or through an index representing the rows and columns. Use the following methods on a pointer referring to a sheet.

{% highlight js %}
setCellValue('A1','Foo');
setCellValueByColumnAndRow(0, 2, 'Bar'); //the two methods
{% endhighlight %}

Running the above two commands will set A1 to ‘Foo’ and A2 to ‘Bar’. Remember — the row index starts from 1 and not 0.

We have a sample ready. But we have not yet created an excel file. For performing any kind of input/output with respect to PHPExcel, you need to include/require *IOFactory.php*. This file is located in the PHPExcel folder in the package that you downloaded earlier today.

Now you can perform the following steps —

{% highlight js %}
$PHPExcelWriter = PHPExcel_IOFactory::createWriter($PHPExcelObject, 'Excel2007');
$PHPExcelWriter->save(str_replace('*.php','*.xlsx', __FILE__));
{% endhighlight %}

You can replace *.php with your filename which contains your code and replace *.xlsx with the name you desire to give your final excel file. You can have several output formats for the final file. You’ll have to appropriately change the required parameters according to the file format. The homepage (Link given above) gives a brief of what all input (Excel 2007, CSV, etc) and output (Excel 2007, PDF, HTML, etc) formats it can handle.

### Reading excel files into your program

To read a file, the following syntax can be used —

{% highlight js %}
$PHPExcelRead = PHPExcel_IOFactory::load("#file_path");
{% endhighlight %}
Make sure you have included the *IOFactory.php* file before you go ahead with this.

You can use the `getWorksheetIterator()` method to traverse across the sheets of the file and use a foreach loop to read the values in cells. Here is an example —

{% highlight js %}
foreach ($PHPExcelRead->getWorksheetIterator() as $worksheet)
{% endhighlight %}

You can use methods like `getHighestRow()`, `getHighestColumn()`, `getCellByColumnAndRow($col, $row)`, etc to get the various values. At this point I would like to mention that the `getCellByColumnAndRow()` method returns an object and you will have to perform -

{% highlight js %}
$worksheet->getCellByColumnAndRow($col, $row)->getValue();
{% endhighlight %}

to handle the actual value in your program.

These are the basics of reading and writing an excel file through PHPExcel. You can handle a lot more with PHPExcel like formula, charts, etc.

{% include twitter_plug.html %}
