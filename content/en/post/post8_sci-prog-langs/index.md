---
title: Science programming languages
summary: In this post I will talk about the languages of scientific programming.
date: 2024-08-20
authors:
  - admin
tags:
image:
---

## Introduction

In the article we will look at the following topics:

1. LaTeX markup language
2. Julia programming language
3. Octave programming language

## LaTeX markup language

 LaTeX (pronounced /liquidlɑːtɛx/ or /uraleɪtɛx/[1]) is the most popular set of macro extensions (or macro-packets) of the TeX computer layout system that facilitates the set of complex documents.

 This tool is widely used for creating scientific documents, writing books, and many other forms of publications. Not only does it allow you to create beautifully designed documents, but it also allows users to implement very quickly such complex elements of a printed set as mathematical expressions, tables, links and bibliographies, getting a consistent layout across all sections.

 With the availability of a large number of open libraries (a little bit later), LaTEX’s capabilities are almost limitless. These libraries empower users even more by allowing you to add footnotes, draw diagrams, etc.

 One of the most compelling reasons many people use LaTeX is to separate the content of a document from its style. This means that after writing the content, you can easily change its appearance. Similarly, one document style can be created and used to standardize the appearance of others.

 This allows scientific journals to create templates for the proposed materials. Such templates have a given markup, leaving only the content to be added. In fact, there are hundreds of such templates, from various resumes to slide presentations.

## Julia programming language

 The Julia language is a cross-platform compiled, freely distributed programming language (MIT license) with dynamic typing, which has a number of advantages and disadvantages. 
The B Julia is a JIT-based compilation. The Just-in-Time (JIT) compilation allows both expressiveness of modern interpreted languages and performance of languages such as C and Fortran. The JIT compiler performs the compilation during the first run of the program, extracting from the text information not explicitly specified by the programmer, and using this information to optimize the generated machine code.

 The advantages of the Julia language include the following: 
 - Simplicity is an intuitive language whose syntax resembles that of Python and MATLAB; 
 - the performance of programs written in Julia is comparable to that of programs written in C or Fortran; 
 - effective vectoring and parallelization of calculations, a large number of data types, including rational and complex numbers; 
 - calculation in the absence of some data (there is a type given missing);
 - calculation precision setting (BigFloat data type, setprecision function);
 - the implementation of symbolic calculations using macro commands;
 - using libraries written in C and Fortran, and sharing libraries with Python and R; 
 - integration with DBMS (PostgreSQL, MySQL, JSON). ;
 - support for Unicode characters; 
 - the use of the multiple dispatch paradigm - the call of the function depends little on the type of parameters of the function (parametric polymorphism); 
 - a good built-in mathematical library with linear algebra functions. 
 
 Julia is a full programming language that has integer arithmetic, floating point operations, cycles, C-like structures, and even a goto operator analogue. However, in the language there is a very convenient possibility of vectoring calculations, which is characteristic of high-level languages. 
 - If we talk about the shortcomings of the Julia language, it should be noted that the compilation time of large programs can be quite noticeable (minutes); 
 is a relatively new language, so the number of libraries in this language is relatively small ( compared to Python ); 
 - cannot create separately tested program - Julia programs work only in Julia environment.

## Octave programming language

 MGNU Octave is a free system for mathematical computing[1] that uses a high-level MATLAB compatible language.

 Octave introduces an interactive command interface to solve linear and nonlinear mathematical problems, as well as other numerical experiments. In addition, Octave can be used for batch processing. Octave uses arithmetic of real and complex scalars, vectors and matrices, has extensions for solving linear algebraic problems, finding the roots of systems of nonlinear algebraic equations, working with polynomials, solving various differential equations, integration of systems of differential and differential algebraic equations of the first order, integration of functions on finite and infinite intervals. This list can be easily expanded using the Octave language (or using dynamically downloadable modules created in C, C++, Fortran, etc.).

 Researchers at the University of Maryland in the United States performed comparative analysis of mathematical computing using MATLAB, Octave, SciLab and FreeMat in a simple and complex scenario. In the first case the system of linear equations was solved and in the second - the finite-differential discretization of the Poisson equation in two-dimensional space. The main conclusion is that GNU Octave performs better than other open mathematical packages, demonstrating a comparable result (pages 23 and 25) than Matlab.

 Scientific calculations made using open software have an additional «level of protection», because anyone can repeat the same calculations and check the validity of the results. The same calculations made on expensive software partially cut off the possibility of checking the results. The problem is actually much wider (English text) and it’s not just open or proprietary mathematics. It is no secret that scientific journals usually do not require authors to provide data and methods sufficient to guarantee the repetition of experimental results, model verification. Economists and financiers in particular often do this by simply keeping their data secret. Verification of calculations and conclusions among the sample of the array of articles with «classified» data gave unexpected results (English text). Science, like software, has to be open, which is why open math packs have value to society.

## Conclusions

In scientific programming many languages and mathematical systems are used, but listed in this article, in my opinion, the most convenient and effective.

