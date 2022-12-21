# COBOL Tools
COBOL is an old language but still with limited tool support: there exist compilers and (limited) IDEs, but that's pretty much it, at least as far as I know.
No pretty-printers, no documentation generators, no refactoring tools, limited interaction with other environments...

As I happen to work in a software migration company that is highly involved in the migration of legacy COBOL code, it is in my own interest to make working with COBOL more easy and flexible...
A side note: todays commercial COBOL offerings (MicroFocus's ServerExpress etc., Fujitsu's NetCOBOL, IBM's Enterprise COBOL) are very expensive!

Therefore, and also just for proving to myself that better stuff is possible, I'm going to create a set of COBOL-focused tools:

- __Preprocessor__:  
  Just a 'normal' COBOL preprocessor which will expand copybooks and apply REPLACE actions. 
  This in itself is a useful tool in the light of working with other preprocessors (e.g. SQL preprocessors) which cannot see 'beyond' a COPY statement. Moreover, the basic functionality (tokenization, COPY/REPLACE-interpretation) included in a preprocessor will be needed in all other tools anyway.
- __Documentation Generator__: 
  Similar in spirit to javadoc and doxygen, this will generate an interlinked web-site for a COBOL project, providing an overview of all files, programs, sections, paragraphs and variables that are part of the projects. 
- __Pretty Printer__: 
  Reformat a COBOL source file according to a configurable set of layout settings.
- __Static Analyzer__: 
  A static analyzer in the footsteps of findbugs, pmd, checkstyle which will analyze complexity metrics and check for duplicate code and a range of other potential issues.
- __Batch Editor__: 
  A tool which allows to script certain edit actions to a COBOL source, similar in spirit to sed but instead of being text/line oriented only, this tool should also be able to find locations based on the recognition of actual COBOL syntax. The edit actions will be: insert-before, insert-after and replace.
- __Advanced Search__: 
  As a fundamental building block for the 'batch editor', but also useful in itself: a tool that can search for the occurrence of certain COBOL syntax in a set of COBOL sources. The general search should be based on a mix of using regular expressions on actual COBOL grammar descriptions. Next to that, a set of preconfigured 'searches' will be available including: all usage locations of a variable, all call locations of a program, all copy locations of a copybook, ...

If I ever get here, there are two possibilities:
- I'm completely fed up with COBOL and this is it! Stop. Basta.
- Or I start working on my own COBOL compiler: probably compiling to Java bytecode or .NET CLI.