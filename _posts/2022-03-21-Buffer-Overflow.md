---
layout: single
title: Buffer-Overflow
excerpt: "Buenas! Hoy os traigo un poco de Buffer-Overflow, algo que he estado tocando un poco ultimamente y me gustaria compartir con vosotros, espero que os sirva y os guste!"
date: 2022-03-21
classes: wide
header:
  teaser: /assets/images/overflow.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - Buffer-Overflow
  - Scripts
  - infosec
tags:
  -   Buffer-Overflow
---

### 1. Buffer-Overflow Básico
[Buffer-Overflow Básico](https://h4ckbl0g.github.io/Buffer-Overflow-basico/#)
En este post os enseño lo mas basico del buffer-overflow, os lo recomiendo si es la primera vez que tocais BOF.
### 2. Buffer-Overflow TRUN function vulnserver
[Buffer-Overflow TRUN function vulnserver](https://h4ckbl0g.github.io/Buffer-Overflow-trun/#)
En este post os muestro la vulnerabilidad existente en la funcion TRUN de vulnserver, como explotarlo y aprovechar el BOF para obtener una rev shell.
### 3. Buffer-Overflow SEH (GMON function) vulnserver
[Buffer-Overflow SEH (GMON function) vulnserver](https://h4ckbl0g.github.io/Buffer-Overflow-seh/#)
En este post explotamos la vulnerabilidad de la funcion GMON el cual podemos explotar a traves de un SEH (Structured Exception Handling) Overflow.
### 4. Buffer-Overflow Bypass DEP vulnserver
[Buffer-Overflow Bypass DEP vulnserver](https://h4ckbl0g.github.io/Buffer-Overflow-byp-dep/#)
En este post muestro como se puede explotar un BOF haciendo bypass del DEP (Data Execution Prevention) de Windows.
### 5. Buffer-OverFlow Bypass de NX y ASLR
[Buffer-OverFlow Bypass de NX y ASLR](https://h4ckbl0g.github.io/Buffer-Overflow-byp-nx-aslr/#)
En este post realizamos un bypass de NX (No Execution) y ASLR (Address Space Layout Randomization)
### 6. Buffer-OverFlow 64bits
[Buffer-OverFlow 64bits](https://h4ckbl0g.github.io/Buffer-Overflow-64bits/#)
En este post os muestro como cambia la explotacion de un binario de 64 bits.