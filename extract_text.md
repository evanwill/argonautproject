# Extracting Text from PDF

## problem

The digital collection hosted on CONTENTdm features a searchable "full text" field populated by text automatically extracted from the uploaded OCRed PDFs.
However, there are a few issues with the full text field:
- the quality of OCR is bad, resulting in a transcript that is mostly gibberish with excessive white space and junk characters.
- the "full text" field is limited to 128,000 characters. Approximately 200 items exceed this limit, thus the field is arbitrarily truncated.
- the large field size seems to negatively impact the system's ability to index 

To improve this situation we must first extract the OCR transcript from the PDFs so that it can be worked with.
This should be an easy task, but its not--
Because the newspapers have complex page layouts, the exact "reading" order of the text transcript must be interpreted by the exporter, 
thus different utilities create varying quality of outputs. 
Here are some options:

- The simplest option is to use Adobe Acrobat (if you have this expensive piece of software installed...). 
Export the text by setting up a batch process (Advanced > Document processing > Batch process).

- I had slightly better outcomes using [Xpdf](http://www.foolabs.com/xpdf/)'s built in command line function `pdftotext`. 
A directory of PDFs can be processed on windows using the PowerShell command: `get-childitem *.pdf | foreach-object -process { pdftotext $_.name }`

- I got even better results using [Apache Tika](http://tika.apache.org/). Viewing the website, Tika appears very complicated to set up and use. Don't be intimidated, if you ignore all the developer documentation, you can get going with its basic functionality very easily. Go to the [download page](https://tika.apache.org/download.html) and get the `tika-app-*.jar` version. Put this file in a handy directory. Open a terminal / command prompt and cd to the directory. The commands are listed near the bottom of the ["Getting Started"](http://tika.apache.org/1.13/gettingstarted.html) page, or type `java -jar tika-app-1.13.jar --help` (be sure to replace tika-app-1.13.jar with the exact version name you downloaded). You can start a simple GUI to play around with Tika's capabilities with the command `java -jar tika-app-1.13.jar --gui` . I used this powershell command to quick and dirty test extracting text from a directory of PDFs, ```get-childitem C:\argonautTest\*.pdf | foreach-object -process { java -jar tika-app-1.13.jar -t $_ > "$($_.name) .txt" }```
