---
layout: post
title: Code Satisfiability
date: 2015-03-28 23:06:30.000000000 -07:00
type: post
published: true
status: publish
categories:
- Application Security
- Testing
tags:
- Concolic Testing
- LLVM
- Satisfiability Modulo Theories
- SMT
- solvers
excerpt: Overview of the usage of satisfiability modulo theory solver for code vulnerability
  testing.
---
<p><strong>Code Satisfiability</strong> - Achievable in constrained contexts.</p>
<p><strong>Code Validity</strong> - Very difficult, I haven't found much academic work that tackles that successfully.</p>
<p>When we want to thoroughly test code, specifically for security vulnerabilities. We have a couple of options:</p>
<ul>
<li>Code audit - Slow, manual, not scalable, but can find complex bugs</li>
<li>Blind fuzzing - Fast, scalable, but mostly catches the lowest hanging fruit.</li>
<li>Whitebox fuzzing - Fast, scalable and can identify some intermediate complexity bugs</li>
<li>Symbolic execution - Can be exhaustively slow, flexes all input combinations, so has the potential to identify complex bugs, at scale.</li>
</ul>
<p>There is lots of academic work being done on code satisfiability, lots of amazing work. However there appears to be duplication of implementations in the academic world.</p>
<hr />
<p><em>Thesaurus check:</em></p>
<p>Dynamic Symbolic Execution <span class="_Tgc">≈</span> Concolic Testing (Or as I liked to call it "Mixed symbolic and concrete testing")</p>
<p>The aim of Concolic/Dynamic... is to deal with the path explosion that comes from testing every valid state of variables in symbolic execution.</p>
<hr />
<p>One of the key strategies, outlined well by the <a title="KATCH" href="http://srg.doc.ic.ac.uk/files/papers/katch-fse-13.pdf" target="_blank">KATCH</a> paper is to use a regression suite to allow you concretely execute a binary up to the vicinity of the code under test, after which you can symbolically execute. This limits the search space which helps to deal with the path explosion issue.</p>
<p>To take a step back to the basics of how to symbolically execute a piece of code, you can either take one of 2 approaches:

Translate source code to an abstract syntax tree, then walk that to generate SMTLIB2 compliant language like <a title="Z3" href="http://z3.codeplex.com/" target="_blank">Z3</a> or <a title="STP" href="https://sites.google.com/site/stpfastprover/" target="_blank">STP</a> to execute the function symbolically.</p>
!["AST Approach"](/assets/smt-blog-post-diagram-p1.png)
<p>Ideally using something like <a title="ANTLR" href="http://www.antlr.org/" target="_blank">ANTLR</a> to do the AST generation as there are ANTLR grammars available for nearly all the popular source code languages.</p>
<ul>
<li>Good (pretty much the only way) for dynamic languages</li>
<li>More simplistic component design.</li>
<li>Lots of code code to maintain</li>
</ul>

Or we have the option to use the abstraction of <a title="LLVM" href="http://llvm.org/" target="_blank">LLVM</a> or <a title="Binary Analysis Platform (BAP)" href="http://bap.ece.cmu.edu/" target="_blank">Binary Analysis Platform (BAP)</a>, which is the most popular with the existing academic tools.</p>
!["bitcode Approach"](/assets/smt-blog-post-diagram-p2.png)
<p>Adding in the LLVM Intermediate Representation (IR) abstraction removes the maintenance overhead of the grammar transform work, the LLVM IR is walkable similar to an AST.</p>
<ul>
<li>Less code to maintain</li>
<li>Can analyze code without having source and any binary that LLVM can give you IR for should work with your solver.</li>
<li>Concrete execution can be achieved through LLVM too, so no requirement for a separate test runner.</li>
<li>Doesn't work for non-compiled languages. e.g. Javascript, however there are workarounds like <a title="Jalangi" href="https://github.com/SRA-SiliconValley/jalangi" target="_blank">Jalangi</a></li>
</ul>
<p>References:</p>
<ul>
<li><a href="https://klee.github.io/" target="_blank">KLEE</a> - Framework to allow you to declare specific values in your code to be symbolically executed, uses LLVM alongside a solver of your choice, the default is STP.</li>
<li>KATCH - An automated patch analysis tool that uses KLEE and some interesting Control Flow Graph distancing to concretely execute your code to the vicinity of the patch and then symbolically execute the variables in that area, using KLEE</li>
</ul>
