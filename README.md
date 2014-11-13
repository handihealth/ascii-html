ascii-html
==========

The latest version of [Asciidoctor](http://asciidoctor.org/) can transfer `asciidoc` to `Html` file. While the previous version of Asciidoctor did not support to transfer `asciidoc` to `html` directly. 

Transfer asciidoc to html, it should use two tools: asciidoctor and Pandoc.


##  Installation

### Install asciidoctor
The plugin depends on the Asciidoctor gem, named http://rubygems.org/gems/asciidoctor[asciidoctor].
You can install the gem using:

 $ gem install asciidoctor

### Install Pandoc
*	There is a package installer at pandoc’s [download page](https://github.com/jgm/pandoc/releases). If you later want to uninstall the package, you can do so by downloading this script and running it with perl uninstall-pandoc.pl.


## Usage

### Transfer AsciiDoc to html5 
After installation, it can use command in terminal like:

	 $ asciidoctor  rest_examples.asciidoc

By this, it output an html file of rest_examples.html which is uploaded in the same directory. 

###  Transfer AsciiDoc to Markdown
	
Firstly, transfer AsciiDoc to Docbook5 file by using following codes. 

	asciidoctor -b docbook5 rest_examples.asciidoc
	
It will output an xml file which is a docbook file. 
	
Then use Pandoc to transfer docbook file to markdown file by 

	pandoc rest_examples.xml -f docbook -t markdown -s -o rest_example.md


