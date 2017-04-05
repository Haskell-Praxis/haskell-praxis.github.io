---
title: IDE comparison
author: Kevin Singer, [ADD YOUR NAME HERE]
---

* eclipseFP: not maintained any more
* kdevelop: dropped support
* leksah: 
  * can be difficult to build
  * has several bugs that cause crashes
  * <https://hackage.haskell.org/package/leksah>
* atom: 
  * installed all plugins with "ide-haskell" in the name
  * limited code transformations (e.g. "insert type", "to point free")
  * jump to definition: in same project via ghc-mod or across projects via hasktags function search
  * the local hoogle database is used for the mouse-over type-hints.
* vscode: 
  * haskero as base plugin
  * stylish-haskell and hindent format for styling 
  * phoityne for debugging
* vim: <http://www.stephendiehl.com/posts/vim_haskell.html>


|feature                 |     vscode    |atom ide-haskell|IDEA-haskforce |    leksah     |     vim       |     emacs     |
|------------------------|---------------|----------------|---------------|---------------|---------------|---------------|
|debugger                |    **yes**    |        ?       |       ?       |       ?       |       ?       |       ?       |
|auto-parantheses        |    **yes**    |    **yes**     |    **yes**    |       ?       |       ?       |       ?       |
|syntax-highlighting     |    **yes**    |    **yes**     |    **yes**    |       ?       |       ?       |       ?       |
|multi-package problem\* |    **yes**    |    **yes**     |    **yes**    |       ?       |       ?       |       ?       |
|automatic rebuild\*\*   |      no       |       no       |    **yes**    |       ?       |       ?       |       ?       |
|type-checking & errors  |  **on-save**  |    **yes**     |    **yes**    |       ?       |       ?       |       ?       |
|auto-completion         |    **yes**    |    **yes**     |    **yes**    |       ?       |       ?       |       ?       |
|support for refactoring |      no       |       no       |    **yes**    |       ?       |       ?       |       ?       |
|prettify / code styling |    **yes**    |    **yes**     |       no      |       ?       |       ?       |       ?       |
|jump to declaration     |    **yes**    |    limited     |    **yes**    |       ?       |       ?       |       ?       |
|hotkey for commenting   |    **yes**    |    **yes**     |    **yes**    |       ?       |       ?       |       ?       |
|show type-information   |    **yes**    |    **yes**     |       ?       |       ?       |       ?       |       ?       |
|show docs for functions |      no       |       no       |       ?       |       ?       |       ?       |       ?       |
|open in hoogle (browser)|      ?        | **as popups**  |    **yes**    |       ?       |       ?       |       ?       |
|local hoogle integration| via terminal  |    **yes**     | via terminal  |       ?       |       ?       |       ?       |
|ghci integration        | via terminal  |    **yes**     | via terminal  |       ?       |       ?       |       ?       |
|stack integration       | via terminal  |    **yes**     |    **yes**    |       ?       |       ?       |       ?       |
|fast (startup)          |    **yes**    | slow plugins   |       no      |       ?       |       ?       |       ?       |
|usability               |    **yes**    |    **yes**     |    **yes**    |       ?       |       ?       |       ?       |

\* [multi-package-problem](http://andrewufrank.blogspot.co.at/2017/03/the-haskell-tool-stack-for-multi.html) solvable. This also means, that other functionality (e.g. jump-to-declaration) should work across these projects.
\*\* can be done via `stack build --file-watch` in any terminal