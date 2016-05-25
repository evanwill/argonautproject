# Extracting Text from PDF

## problem

The digital collection hosted on CONTENTdm features a searchable "full text" field populated by text automatically extracted from the uploaded OCRed PDFs.
However, there are a few issues with the full text field:
- the quality of OCR is bad, resulting in a transcript that is mostly gibberish with excessive white space and junk characters.
- the "full text" field is limited to 128,000 characters. Approximately 200 items exceed this limit, thus the field is arbitrarily truncated.
- the large field size seems to negatively impact the system's ability to index 

To improve this situation we must first extract the OCR transcript the PDFs so that it can be worked with.
This should be an easy task, but its not--
Because the newpapers have complex page layouts, the exact "reading" order of the text transcript must be interpreted by the exporter, 
thus different utilities create varying quality of outputs.

The simplest option is to use Adobe Acrobat. 
Export the text by setting up a batch process (Advanced > Document processing > Batch process...).

I had slightly better outcomes using [Xpdf](http://www.foolabs.com/xpdf/)'s built in command line function `pdftotext`. 
A directory of PDFs can be processed on windows using the PowerShell command: `get-childitem *.pdf | foreach-object -process { pdftotext $_.name }`
