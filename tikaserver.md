# How to set up Tika Server on a VM

Set up a Ubuntu server VM, `ssh` in and follow these instructions to set up a Tika Server with NER and OCR enabled. For full documentation check the [Tika Server wiki](https://wiki.apache.org/tika/TikaJAXRS).

## Install dependencies

Add Java, Tesseract OCR, and Apache to your machine:

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install default-jre tesseract-ocr apache2
```

## Create a tika user

Create a non-root user and password:

```
sudo useradd -m tika-user
sudo passwd tika-user
```

## Get Tika

Its easiest to use the pre-built tika `.jar` files. 
Switch to tika-user (`sudo su - tika-user`), and create a directory `mkdir tika` and `cd tika`.
Download Tika from a mirror (find one on this [page](https://tika.apache.org/download.html)) with `wget`.
For example `wget http://apache.parentingamerica.com/tika/tika-server-1.13.jar`.

## Get NER

Its easy to extend Tika with other services. Because we installed Tesseract OCR, it will automatically start using it ([more info](http://wiki.apache.org/tika/TikaOCR)). 
We can also add Named Entity Recognition. For full information and how to install more packages, check the [wiki](http://wiki.apache.org/tika/FrontPage#Named_Entity_Recognition_.28NER.29_support). 
Lets install the basic OpenNLP NER, full instructions are [here](http://wiki.apache.org/tika/TikaAndNER). Its easy:

Create a directory `mkdir tika-ner-resources`,  and `cd tika-ner-resources`.
Now we need to download models from OpenNLP and put them in the correct directory. To make this easier, set up some varibles. 
```
PATH_PREFIX="$PWD/org/apache/tika/parser/ner/opennlp"
URL_PREFIX="http://opennlp.sourceforge.net/models-1.5"
```
Now, create the directory and download the NER models for person, location, and org:
```
mkdir -p $PATH_PREFIX
wget "$URL_PREFIX/en-ner-person.bin" -O $PATH_PREFIX/ner-person.bin
wget "$URL_PREFIX/en-ner-location.bin" -O $PATH_PREFIX/ner-location.bin
wget "$URL_PREFIX/en-ner-organization.bin" -O $PATH_PREFIX/ner-organization.bin
```

## Create Tika Config

It is necessary to create a config file to tell Tika which parsers to use. 
In the `tika` directory create a file named `tika-config.xml` (for example `nano tika-config.xml`).
Add this xml:
```
<?xml version="1.0" encoding="UTF-8"?>
<properties>
    <parsers>
        <parser class="org.apache.tika.parser.DefaultParser">
                <mime>application/pdf</mime>
                <mime>image/tiff</mime>
        </parser>
        <parser class="org.apache.tika.parser.ner.NamedEntityParser">
            <mime>text/plain</mime>
            <mime>text/html</mime>
            <mime>application/xhtml+xml</mime>
        </parser>
    </parsers>
</properties>
```

## Start Tika Server

Tika Server runs at port 9998, so be sure to create a proxy or open the port in your security settings. 
To start the server you need to use absolute paths to the files. 
If you don't NER and OCR will not be available.
We can use `screen` to start the process and detach from the terminal so that it will continue running. 
We start the server using `java` and give it the classpath. 
Make sure the set the host option to `*` and include the `tika-config.xml` file. 

```
screen
sudo   
sudo su - tika-user 
cd tika 
java -classpath /home/tika-user/tika/tika-ner-resources:/home/tika-user/tika/tika-server-1.13.jar org.apache.tika.server.TikaServerCli --host=* --port=9998 --config=tika-config.xml
```
To detach from the terminal, press `ctrl+a` and `d`. The server will continue to run without your terminal. 

## Using Tika

Now the the server is running anyone can send files to Tika to parse. It is easiest to do this using `curl`. Here is the basic usage: 

Test Tika Server:
`curl -X GET http://206.12.96.54:9998/tika`

Send an OCR-ed PDF to Tika and get extracted text:
`curl -T test.pdf http://206.12.96.54:9998/tika > text.txt`

Send an image to Tika and get OCR text:
`curl -T test.tif http://206.12.96.54:9998/tika > text.txt`

Send plain text to Tika and get NER output:
`curl -T text.txt http://206.12.96.54:9998/meta > meta.txt`

Get NER output as JSON:
`curl -T text.txt http://206.12.96.54:9998/meta -H "Accept: application/json" > meta.json`
