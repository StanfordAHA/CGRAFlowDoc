How to contribute to the CGRAFlow docs:
* clone the repository master branch e.g.
  clone https://github.com/StanfordAHA/CGRAFlowDoc
* build/edit your page as a markdown file e.g.
  vim CGRAFlowDoc/hardware/CGRA-Specs/bitstream-spec.md
* for help with markdown format see
  https://guides.github.com/features/mastering-markdown/
* remember to add your doc to the table of contents file
  "CGRAFlowDoc/SUMMARY.md" so it will appear in the webpage nav bar
  
  E.g.
  
    ```* [Bitstream Specification](hardware/CGRA-Specs/bitstream-spec.md)```
* when you push your changes to the master branch, gitbook will
  automatically build the html-based master doc
  https://travis-ci.com/StanfordAHA/CGRAFlowDoc/builds
* final doc can be reviewed here: https://stanfordaha.github.io/CGRAFlowDoc/
