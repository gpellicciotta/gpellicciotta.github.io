# Parsing Notes

## Building Blocks & Phases
Necessary building blocks:
 - tokenizing text into tokens
 - parsing text into abstract syntax trees
 - building symbol tables  
 - incremental re-parsing sub-ASTs 
 - incremental re-tokenizing sub-tokenstream-sequences

Parsing is a multi-layered process consisting of:
  1. Splitting an input text in lines and keeping track of line-boundaries
  2. Turning input text into tokens
  3. Turning tokens into ASTs

 The reverse - generation - is also possible:
  4. Turning ASTs back into tokens
  5. Turning token-streams back into text

## Design Choices

### Why tokens?
- because formatting and syntax-highlighting are typically token-based
- to encapsulate all position information: ASTs then only indirectly have positions, via their tokens
- tokenizing first is assumed to have a large performance benefit (to be proven however)

### Why Abstract Syntax Trees?
- because they are easier to work with (than concrete syntax trees based on formal grammers)
- by letting them represent all concrete syntax, we should have best of both worlds

### How to Enable Editing/Incremental Re-Parsing?
- Based on editing position, the top of the AST covering both the syntax just before and just after the 
  editing position needs to be found
- Then re-tokenize from the position of the token just before until the token just after, before editing
- Then re-parse the whole token-stream associated with the original AST

### How to Update Position Information After Editing/Re-parsing?

## Possible Design

```
enum SyntaxType {
  ... all tokens and syntax parts...
}

class SyntaxElement {
  SyntaxType Type;
}

// A node represents the AST and contains/encompasses other nodes and tokens
class SyntaxNode extends SyntaxElement {
  SyntaxElement[] children;
}

// A token is associated with a source location
class SyntaxToken extends SyntaxElement {
  Location SourceLocation;
}




## References
- Roslyn is an open-source implementation that has solved all typical challenges and has proven
  to work, since it is the basis of the actual C# and VB compilers:
  - [red-green trees](https://ericlippert.com/2012/06/08/red-green-trees/)
  - [performance considerations](https://github.com/KirillOsenkov/Bliki/wiki/Roslyn-Immutable-Trees)
  - [source code](https://github.com/dotnet/roslyn)
