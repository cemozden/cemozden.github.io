---
layout: post
title:  "Generating HTML based reports by using PowerShell"
date:   2021-05-06 19:07:15 +0200
categories: powershell html report wmi
author: Cem Özden
---
Tendering efficient reports is one of the goals of every *Automation Engineers, System Administrators* in our world right now. There are great numbers of solutions to ensure this objective. When I used to work as an Automation Engineer, ***PowerShell*** was to be my wrench in terms of preparing reports. While my colleagues had opt for Excel, CSV reports over HTML based reports, Thus, I believed that It’d be great shot to start writing a post about the conjunction of PowerShell and HTML, CSS when it comes to preparing reports.

### What are the benefits of using HTML over Excel as well as CSV reports?
* HTML, CSS technologies provides the power of Web. That is, We can use all available web technologies that have been set up already.
* HTML Reports are much customizable over Excel based Reports.
* Excel requires API while HTML does not.
* CSV reports are only text formatted reports whilst HTML reports could be rendered into any sort of colored layouts.
* Writing excel reports is much complex than writing HTML reports.
* HTML Reports could be used as the body of a mail. That is, We can create automation scripts that send mails by sending HTML output as the mail body.
* HTML reports are prominently easier to be analyzed by end-users than CSV.
* The size of HTML output is usually smaller than the size of Excel files.

### Requirements

In order to be going further with an example, *I assume that the reader of this post has got* ***intermediate level of understanding, writing PowerShell Scripts, Basic level of understanding HTML as well as CSS***. I’m not going to mention client side web technologies such as JavaScript etc. in this post as it’s not going to fit our goals in this example.

### A HTML based hardware information report

As an example, I’m going to create one PowerShell script that fetches Hardware Information (Computer System Information, Operating System, Processor, Memory) and generates a HTML output as well as writes it to a file. Before I started writing this post, ***Please note that*** I’d created a HTML template which consists of one *System Information Header and System Info HTML Table*. You can find the template and the source code at the end of this post from my [GitHub](http://github.com/cemozden) account.

#### Script Structure
*Preferably, It’d be better to compose a HTML template before even starting writing the script. That will help you out with analyzing a preview of your script output.*

The script will be generating the HTML file as the following,

Firstly, We distinguish the HTML template into 2 pieces. That is, One is going to tend for being the header of the template, The second piece will be playing the footer role of the template. These 2 pieces are written into 2 files named `header.html` and `footer.html`

The script will then be loading both files as well as merging the header with replacing custom template tags with values presented in the PowerShell script. Later on, We’ll be generating the dynamic part of the template from the data we collected by using [WMI](https://en.wikipedia.org/wiki/Windows_Management_Instrumentation).

Lastly, We’re going to add the footer slice of the page and write the template into one file. The structure could be presented as the following figure.

![report structure](/assets/images/structure.jpeg)

*The structure of the template which is going to be handled by the PowerShell script.*


#### Custom Template Tags

It’s pretty obvious that we might need to inject some dynamic data to distinct points of our template. To do that, We can define custom template tags (in which could be considered as variables) I would prefer the format `${tagName}` for my scripts however This format definitely depends on the person who writes the script. In this script, I’ll be defining a custom template tag ${computerName} which will be replaced by the system name during the runtime. I’ll be using the [Replace(strOldChar, strNewChar)](https://ss64.com/ps/replace.html) method of the String object in order to replace tag with the corresponding value.

### Loading and rendering custom template tags

Let’s get our hands dirty by writing the script now! Primarily, Let me begin with calling WMI classes "*win32_ComputerSystem, win32_OperatingSystem and win32_Processor* in order to collect the existing system hardware data using WMI. Then, We’ll be loading header into the script and render the tags.


The following codes are representing header.html and its rendered on PowerShell side.

```html
<!-- header.html File -->
<html>
    <head>
        <title>System Information</title>
        <style>
            body {
                font-family: Arial;
                font-size:12px;
                background-color: #ccc;
            }
            .wrapper {
                padding-top: 50px;
                margin: 0 auto;
                width: 800px;
            }
            table {
                border-collapse: collapse;
                border: 2px solid #303030;
                border-radius: 5px;
            }
            td {
                min-width: 250px;
                padding: 5px;
                border: 1px dashed #dddddd;
            }
            th {
                background-color: #616993;
                color: #FFFFFF;
                padding: 10px;
                border: 1px dashed #000000;
            }
            tr:nth-child(even) {background-color: #eeeeee;}
            tr:nth-child(odd) {background-color: #f7f7f7;}
            .hw {
                text-align: right;
                font-weight: bold;
            }
        </style>
    </head>
    <body>
        <div class="wrapper">
            <h1>${computerName} System Information</h1>
            <table>
                <tr><th>Item</th><th>Value</th></tr>
```

```powershell
#Get Hardware Information from WMI
$csWMI = Get-WMIObject -Class win32_ComputerSystem
$osWMI = Get-WMIObject -Class win32_OperatingSystem
$pWMI = Get-WMIObject -Class win32_Processor
# Create the HashTable in order to match the tags from the HTML file(s).
$tagTable = @{
    computerName = $csWMI.PSComputerName
}
$headerHTML = Get-Content .\header.html -Raw
# Change the tags to corresponding value
$tagTable.GetEnumerator() | ForEach-Object { $headerHTML = headerHTML.Replace($('${' + $_.Key + '}'), $_.Value) }
$resultHTML = $headerHTML
```

In the code above, We call the WMI classes and assign the objects into `$csWMI, $osWMI and $pWMI` variables. Likewise, I created a hash table named `$tagTable` which holds custom template tags. In our case we’ve got only 1 tag which is `computerName`. That is, The `${computerName}` tag will be replaced by the system name that we collected from `win32_ComputerSystem` class. Next line presents how to read the header file which describes our template header. Please, take an attentive look at the `-Raw` **switch of the Get-Content cmdlet**. It’s quite important to use this switch such that *Get-Content will load the content of the header file to the memory **NOT as an array of lines but as a whole string**.*

After loading the HTML code. It’s time to start replacing template tags with their values. I used the method `GetEnumerator()` of HashTable Object in order to iterate over the table. Therefore, I piped the enumerator with foreach-object such that all members of $tagTable will be injected into the header by *Replace(oldStr, newStr)* method. At last, we assign $headerHTML variable to $resultHTML which is going to be used as our final template result.

#### Creating rows of the table

```powershell
$resultHTML += $('<tr><td  class="hw">System Manufacturer</td><td>' + $csWMI.Manufacturer + '</td></tr>')
$resultHTML += $('<tr><td  class="hw">System Model</td><td>' + $csWMI.Model + '</td></tr>')
$resultHTML += $('<tr><td  class="hw">Operating System</td><td>' + $osWMI.Caption + '</td></tr>')
$resultHTML += $('<tr><td  class="hw">Install Date</td><td>' + $osWMI.ConvertToDateTime($osWMI.InstallDate).ToString('MM/dd/yyyy HH:mm:ss') + '</td></tr>')
# NOTICE: For those who has more than 1 CPU in their system (in case of servers) They need to handle the following code as an array rather than a PSObject example: $pWMI[0] etc.
$resultHTML += $('<tr><td  class="hw">Processor</td><td>' + $pWMI.Name + ' <b>' + $csWMI.NumberOfProcessors + ' Core(s), ' + $csWMI.NumberOfLogicalProcessors + ' Logical Processor(s)</b></td></tr>')
$resultHTML += $('<tr><td  class="hw">Memory</td><td>' + ([Math]::ceiling($csWMI.TotalPhysicalMemory / 1GB)) + ' GB</td></tr>')
```

The code above shows how to create the dynamic rows of the table. In this report, we’ll be creating 6 rows of the table that represents "*System Manufacturer, System Model, Operating System, Installation date of the operating system, Processor information and Memory size*". The structure of the code is quite clear. We’re concatenating **table rows (<tr> tags)** with the `$resultHTML` variable that contains the header of our template. We’d already defined the open tag of *<table>* in the header so here, we aren’t supposed to open the tag again. It’s already loaded from the header.PowerShell will be generating the table row strings by concatenating values coming from WMI objects. The first column will keep the item name information (such as System Manufacturer etc.) The second column is going to keep the value of the specific item collected from the WMI objects. In this example, I used CSS class "*hw*" in order to format item columns such that they will be positioned to the right side of their cells. (*Please see the template output below.*)

#### Adding footer and writing the template to the file.

As the latest step, We’ll be just loading the content of the footer of the template from the disk to the memory as in raw mode. Therefore, We’ll be adding the footer to the $resultHTML and as the latest step, we’ll write the value of $resultHTML to hardware_report.html file.

The following codes describes the footer.html and how its rendered in PowerShell side.

```html
<!-- footer.html File -->
            </table>
        </div>
    </body>
</html>
```

```powershell
$footerHTML = Get-Content .\footer.html -Raw
$resultHTML += $footerHTML

$resultHTML | Out-File -FilePath .\hardware_report.html
```

![html output](/assets/images/html_output.jpeg)

*The HTML output done by running the PowerShell Script.*

### The complete script

The following code is the full source of the PowerShell Script.

```powershell
#Get Hardware Information from WMI
$csWMI = Get-WMIObject -Class win32_ComputerSystem
$osWMI = Get-WMIObject -Class win32_OperatingSystem
$pWMI = Get-WMIObject -Class win32_Processor

# Create the HashTable in order to match the tags from the HTML file(s).
$tagTable = @{
    computerName = $csWMI.PSComputerName
}
$headerHTML = Get-Content .\header.html -Raw

# Change the tags to corresponding value
$tagTable.GetEnumerator() | ForEach-Object { $headerHTML = $headerHTML.Replace($('${' + $_.Key + '}'), $_.Value) }

$resultHTML = $headerHTML
$resultHTML += $('<tr><td  class="hw">System Manufacturer</td><td>' + $csWMI.Manufacturer + '</td></tr>')
$resultHTML += $('<tr><td  class="hw">System Model</td><td>' + $csWMI.Model + '</td></tr>')
$resultHTML += $('<tr><td  class="hw">Operating System</td><td>' + $osWMI.Caption + '</td></tr>')
$resultHTML += $('<tr><td  class="hw">Install Date</td><td>' + $osWMI.ConvertToDateTime($osWMI.InstallDate).ToString('MM/dd/yyyy HH:mm:ss') + '</td></tr>')

# NOTICE: For those who has more than 1 CPU in their system (in case of servers) They need to handle the following code as an array rather than a PSObject example: $pWMI[0] etc.
$resultHTML += $('<tr><td  class="hw">Processor</td><td>' + $pWMI.Name + ' <b>' + $csWMI.NumberOfProcessors + ' Core(s), ' + $csWMI.NumberOfLogicalProcessors + ' Logical Processor(s)</b></td></tr>')
$resultHTML += $('<tr><td  class="hw">Memory</td><td>' + ([Math]::ceiling($csWMI.TotalPhysicalMemory / 1GB)) + ' GB</td></tr>')

$footerHTML = Get-Content .\footer.html -Raw
$resultHTML += $footerHTML

$resultHTML | Out-File -FilePath .\hardware_report.html
```

### Suggestions

* In case of using CSS, It’s better to keep CSS code inside of HTML files rather than loading it from a CSS file. This will help you with dealing absolute path issues. Besides, Whenever a HTML report must be the body of a mail, The CSS code must be included into the mail body as well.

* If you’ve got tremendous amount of data to be added into your template, It’s better use StringBuilder rather than String Concatenation. [StringBuilder](https://docs.microsoft.com/en-us/dotnet/api/system.text.stringbuilder?view=netframework-4.8) ensures you to use mutable strings such that you can omit the memory cost of string concatenation. There is a great blog post about Immutable, Mutable strings in PowerShell. Please check this out -> [https://powershellexplained.com/2017-11-20-Powershell-StringBuilder/](https://powershellexplained.com/2017-11-20-Powershell-StringBuilder/)

### Summary

To put it in a nutshell, HTML reports are much flexible, configurable as well as customizable than Excel and CSV reports. Even though It requires some knowledge regarding HTML and CSS, I believe it worths spending some time on creating reports using HTML, CSS technologies. Besides **Microsoft Exchange** provides HTML body mails such that you can automate your scripts, create templates and send the template as a mail to responsible people.

*The limit of designing HTML reports is **you**! The more you develop yourself in terms of HTML, CSS the better your reports will be.*

### Link(s) to the code repository

You can find the example code above in my GitHub Repository.

[https://github.com/cemozden/blog_code_examples/tree/master/create-html-reports-powershell-1](https://github.com/cemozden/blog_code_examples/tree/master/create-html-reports-powershell-1)