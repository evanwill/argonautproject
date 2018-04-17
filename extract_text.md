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

## text extraction tools 

Here are some options:

- Adobe: the simplest option (if you have this expensive piece of software installed) is to use Adobe Acrobat. 
Export the text by setting up a batch process (Advanced > Document processing > Batch process).

- Xpdf: I had got faster output using [Xpdf](http://www.foolabs.com/xpdf/)'s built in command line function `pdftotext`. Xpdf comes installed on most linux distros, so a directory of PDFs can be processed using the default settings with the command `pdftotext *pdf`. 

- Tika: I got better results using [Apache Tika](http://tika.apache.org/). Viewing the website, Tika appears very complicated to set up and use. Don't be intimidated, if you ignore all the developer documentation, you can get going with its basic functionality very easily. Go to the [download page](https://tika.apache.org/download.html) and get the `tika-app-*.jar` version. Put this file in a handy directory. Open a terminal / command prompt and cd to the directory. The commands are listed near the bottom of the ["Getting Started"](http://tika.apache.org/1.13/gettingstarted.html) page, or type `java -jar tika-app-1.13.jar --help` (be sure to replace tika-app-1.13.jar with the exact version number you downloaded). You can start a simple GUI to play around with Tika's capabilities with the command `java -jar tika-app-1.13.jar --gui`. To quickly extract text from a directory of PDFs, use a Bash loop like `for file in ~/Directory/*.pdf; do java -jar tika-app-1.14.jar $file > $file.txt; done`.

- Tika Server: Tika does the job well, but if you are doing more than a couple PDFs you will get much better performance using Tika server. *And* Tika server allows us to intergrate other utilities into the workflow fairly easily, including [Tesseract OCR](http://wiki.apache.org/tika/TikaOCR) and some [named entity extraction tools](http://wiki.apache.org/tika/FrontPage#Named_Entity_Recognition_.28NER.29_support). There is some server specific documentation [here](https://wiki.apache.org/tika/TikaJAXRS). From the [download page](https://tika.apache.org/download.html) get the `tika-server-*.jar` file. Put in into a handy directory. To start the server locally, open a terminal in the directory and type `java -jar tika-server-1.13.jar` (be sure to replace with the exact version number you downloaded). The Tika server will start running at localhost:9998. You can send files for it to parse via http, i.e. open another terminal, cd to the directory with PDFs, and type `curl -T test.pdf http://localhost:9998/tika > test.txt` (if you are on windows, I suggest installing [Git](https://git-scm.com/downloads) which comes packaged with Git Bash allowing you to use a unix style terminal with many unix utilites). Then, if you want to do a whole directory of PDFs, use this bash command: `for file in *.pdf; do curl -T $file http://localhost:9998/tika > $file.txt; done`
