---
layout:     post
title:      Distinction of Different Legislative Text Types
date:       2016-10-25 13:15:00
summary:    A short tutorial on distinguishing Greek laws from other types of legislative texts in the Government Gazzette issues
categories: legislation code
--- 

In this tutorial we focus on the distinction of Greek laws from other types of legislative text (e.g. presidential decrees, ministerial decisions etc.) that are present in the Official Government Gazette issues. While at this stage we put our effort solely on laws, it is our future intention to examine whether modeling regular expressions for structural analysis can be applied to other types of legislative texts (according to the rules that the Central Legislatorial Committee has published). 

A considerable number of Greek laws refers to ratifications of international agreements. In this case, the agreement text is usually incorporated into the PDF file as a scanned image requiring OCR (Optical Character Recognition) techniques, a step that imposes significant obstacles in text analysis. Even if plain text was available, these agreements are usually drafted either in the language of the contracting parties, or in *"... Chinese, French, Russian, English and Spanish versions which are equally authentic ..."* in accordance to Article 111 of the United Nations Charter. As our patterns are modeled in Greek, the latter would be detrimental for pattern matching within the contents, which is necessary in the framework of the automated recognition and application of modifications or for identifying the structural components of the text. Consequently, these laws should be excluded. 

The distinction is based on the fact that specific words are used to flag the beginning and the end of each legislative text type. For instance, each law always begins with the word "Law" followed by its number and ends with a reference to the President of the Hellenic Republic. However, we faced several problems during the design phase of the regular expressions that were needed. One problem identified is the ambiguity in the use of the apostrophe in the word "ΥΠ", which is used before the word "number" and after the word "law" in the Greek language. We observed the use of three different types of apostrophes resulting in difficulty locating and extracting laws. Furthermore, there is also a case where the reference to the President of the Hellenic Republic at the end of the law (Government Gazette 203, Issue A, 2004) was wrong, stating *"THE PRESIDENT OF THE PARLIAMENT"* instead of "THE PRESIDENT OF THE HELLENIC REPUBLIC", naming Karolos Papoulias (President of the Hellenic Republic at that time) as the person signing the law.

The main problem, nevertheless, is the use of Latin characters (looking alike in the the Greek and Latin alphabets, such as O, N or M) mixed with Greek characters in words that are used to flag the start or the end of each law, a case which may pass unnoticed by the reader but causes serious problems in the process of pattern matching.

Based on the above analysis and observations, fourteen (14) different regular expressions are necessary to identify and export laws texts as shown in the image below (red characters indicate the existence of Latin characters instead of Greek): 

![Regular expressions for the identification of laws]({{ site.baseurl }}/images/laws_matching_patterns.jpg)

Our analysis on texts published between 2004 and 2015 shows that in a total of 3209 PDF files, 1128 laws have been exported, from which 578 were excluded from further analysis since they were ratifying international agreements that the parliament had approved. As a result, our main experimental sample consists of 550 laws. 

The source code of the module for the extraction of laws from the plain texts of the Government Gazzette issues can be found [here](https://github.com/OpenLawsGR/greek_laws_consolidation_code/blob/master/python_code/extract_laws.py) while the code that excludes international agreements [here](https://github.com/OpenLawsGR/greek_laws_consolidation_code/blob/master/python_code/split_laws_international_agreements.py)
