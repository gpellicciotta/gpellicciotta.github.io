# Programming Language Toolkit

When creating a new programming language, all the following should be enabled:

- Tokenize + parse a source text
- Be fault tolerant during parsing, recognizing as much as possible
- Enable incremental parsing
- Build a codedom that is easy to inspect and adapt
- Make it easy to build a codedom from scratch
- Generate a codedom into source text
- Make it easy to get at semantic info like:
  - Which variables are used that match certain constraints
  - Which variable types are used that match certain constraints
  - Which statements are used that match certain constraints
  - Where are specific variables used that match certain constraints
  - Where are specific methods/procedures used that match certain constraints
  - Where are specific classes/modules used that match certain constraints
  - Which are possible code paths starting from statement x
  - Which are possible data paths starting from variable x
- Make it easy to define and apply refactorings
- Make it easy to patch/adapt a source      

## Core Concepts

Source:
```
- path
- characters
- lines
```
      
Tokenizer:
```
- Source -> List<Token>
```
        
Token:
```
- type
- fragment: source, start-loc, end-loc  
```
       
Parser:
```
- List<Token> -> CodeDom
```
     
CodeDom: 
```
- type
- tokens
* fragment
* specific attributes
```

SemanticAnalyzer:
```
- List<AbstractSyntaxTree (of type CompilationUnit)> + ProjectSettings -> SemanticInfo
```

Refactoring
      
CodeDomUpdater:
```
- CodeDom + List<Refactoring> -> CodeDom + List<SourceEditCommand>
```      

SourceEditCommand 
      
TextEditor: 
```
- Source + List<SourceEditCommand> -> Source
```

## Core Issues

### How to deal with whitespace?

Filter out at Tokenizer level: 
- For pure whitespace that would be OK but tokens for comments and other non-space texts like preprocessor directives contain important information, even if not for the pure language Parser 
Not filter out but attach whitespace tokens to following or preceding non-whitespace tokens:
- To preceding token if there is a preceding non-whitespace token on the same source text line (E.g. end-of-line comments belong to the last token on the line, unless it is a pseudo-whitespace token like a brace?)
- To next token otherwise (and there is always a virtual END-OF-SOURCE token)
Not filter out and not attach: just make sure they are not seen by the language Parser but keep in the overall token stream

### How to deal with syntax errors/garbage inside a source text?

Try to minimize to minimal source fragment but also provide a place for these ErrorSyntaxPart in the actual CodeDom at levels where it makes sense, e.g. where an expression, statement, module/class could appear. 

For those ErrorSyntaxParts still appearing at other positions... or when there is more syntax that cannot be recognized vs. what can be recognized, have a special type of CodeDom that represents this kind of UnknownSource with just islands of known syntax

An entirely different strategy would be to stop parsing at first error, but that is in general too restrictive: even if just wanting to know whether some piece of source code is syntactically valid, it is better to provide a full list of errors than just the first, and the run again to find the second, ...

### How to enable incremental parsing?

By having a parser that can recognize not only full sources but also common parts like expressions, statements lists, classes/modules, variable declarations, ...
Since this is typically needed in the context of an IDE with syntax highlighting and similar features, there is the additional problem of updating an existing CodeDom when a user types or deletes text: all source positions must get updated somehow:
- One possibility would be to have tokens store position information relative to the previous token (e.g. line + 1, column 0 or line +0, column +2). Then only the first token just after the editing location needs to be updated.
  Big disadvantage is that calculating absolute positions for a certain CodeDom sub-part, which translates to calculating the abs. positions of the first and last tokens covered, will be compute intensive.
- Another possibility could be to introduce another indirection by having tokens stored there location info as a reference to a SourceLine object and a full column position. An edit inside an existing line would then mean finding all tokens that refer to that line and updating their column positions. An edit that deletes existing or introduces new lines on the other hand would just delete, resp. create new SourceLine objects and update the line numbers of all following SourceLine objects, which should be a cheaper operation.
- Note: the MS Roslyn seems to have chosen for two trees (red-green) to solve this issue but I'm not sure anymore why they thought this was needed: maybe because tokens needed to be immutable?

### How to build a CodeDom that is easy to build from scratch and use for generating new source text?

One simple approach would be to use/build a simple parse tree or a more condensed AST (abstract syntax tree):
- Easy to introduce new (or even invalid) syntax: just new subtrees at certain locations
- Additional info (e.g. value of a literal, usage locations of a variable, ...) can easily be added by having each node store additional attributes
- But (manual) building or usage is a pain: have to make/look for nodes of the exact good type in the exact good order and there won't be any help in avoiding mistakes

Another approach is having a actual typed classes representing various language constructs (e.g. a class representing CompilationUnit, another representing an Expression, another represent an AssignStatement, ...):
- A lot easier to build and navigate since the structure follows the language syntax and having type'd objects will make it impossible to build invalid syntax trees
- A lot more difficult to design in a way that it is easy to use yet flexible enough for all likely use cases
  - what if having to build a CodeDom top-down (and hence parts aren't available yet) instead of the normal bottom-up
  - how to cater for invalid syntax parts

Whether a fully typed CodeDom or an untyped AST: how to build it in a way that allows easy re-generation as source text? When it was built by a parser, it will be associated with a list of Token objects and each Token object corresponds directly to text that could just be emitted (after taking care of positioning) but if building from scratch: will there be tokens?
- Certainly in a typed CodeDom scenario each specific CodeDom class could take care of generating tokens representing it's current state; for an untyped AST this seems difficult to impossible since it much more possible to have unexpected nodes and how to deal with those?
  
  
## References

The following are examples of software that tries to deal with all the above:
- Eclipse
  - (AST)[https://www.eclipse.org/articles/article.php?file=Article-JavaCodeManipulation_AST/index.html]
- (Roslyn)[https://github.com/dotnet/roslyn]
  - (Overview)[https://github.com/dotnet/roslyn/wiki/Roslyn%20Overview]
