# Bayesian KMO

*Emil Niclas Meyer-Hansen*

04/06/2025

The Bayesian KMO index is a novel re-conceptualization of the Kaiser-Meyer-Olkin (KMO) index that enables researchers to incorporate prior information and make coherent probabilistic statements about the sampling adequacy of a data matrix.

The Bayesian KMO (BKMO) index is introduced in the research paper *Revisiting ‘Little Jiffy, Mark IV’: Towards a Bayesian KMO index* (newest version published June 4th, 2025). This paper subscribes to the open science standard and is made freely available.

Materials are also made available on its [*Open Science Framework* (OSF) project page] (https://osf.io/t3upd/) (DOI: [10.17605/OSF.IO/T3UPD](https://doi.org/10.17605/OSF.IO/T3UPD)).

**Keywords**: *Kaiser-Meyer-Olkin index, KMO, Bayesian Kaiser-Meyer-Olkin index, BKMO, Measure of Sampling Adequacy, MSA, Bayesian Measure of Sampling Adequacy, BMSA, Robust KMO index, Bootstrap KMO index, Bayesian inference, Likelihood-based inference, Frequentist inference*

## Summary
The paper thoroughly reviews the most popular iteration of the KMO index, formally denoted a Measure of Sampling Adequacy (MSA), including the formula underlying its calculation, its mathematical properties, interpretation, robustness, and inferential viability. Revealing that the classical KMO index is inherently Frequentist and not suited for application within the Bayesian statistical framework, Bayes theorem is used to re-conceptualize the KMO index. The BKMO index is then introduced, followed by a demonstration of its application. An R function implementing the BKMO index is provided and its areas of application are discussed.

## License (Addendum)
Except where otherwise indicated, all contents of this document and associated files are licensed under the *Creative Commons Attribution 4.0 International License* ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)). All software, source code, executable code, code snippets, code chunks, algorithms, and/or scripts within this document and associated files are expressly excluded from the foregoing license, and unless otherwise indicated, are instead licensed under the *GNU General Public License, version 3* ([GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.html)). By engaging with this document and/or any associated files, which include, but are not necessarily limited to, downloading, using, and/or distributing any of them, in parts of whole, you agree to comply with the applicable license terms for the respective content types.
