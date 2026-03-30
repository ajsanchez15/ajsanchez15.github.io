---
title: 'Efficient Modeling of Lightly Reinforced Concrete Columns Under Cyclic Loading'
date: 2026-03-26
permalink: /posts/2026/03/26/RC-cylic
tags:
  - Reinforced Concrete
  - Earthquake Engineering
  - Bridges
---

This month, I spent a considerable amount of time developing a reinforced concrete column model in OpenSeesPy to match the behavior of cyclic tests stored in the PEER structures database. The goal of this work is to integrate it into a detailed highway bridge model for seismic simulations.

Concrete columns typically have two types of reinforcement, longitudinal and transverse. The former primarily reinforces the column against flexural demands, while the latter reinforces the column against shear demands.

<figure>
  <img src="/images/RCCOLUMNS/Confinement.png" alt="Rectangular tied column (a) and circular spiral column (b)">
  <figcaption>Figure 1: Containment of longitudinal bars using (a) rectangular ties and (b) spiral reinforcement.</figcaption>
</figure>

Significant changes were made to bridge design practices following the 1971 San Fernando Earthquake, which caused many bridge columns across the Los Angeles County transportation networks to fail in shear. Typical transverse reinforcement details prior to this earthquake were #3 or #4 rebar at 12 inch center-to-center spacing, resulting in very low transverse reinforcement ratios of about 0.1–0.2%.

<figure>
  <img src="/images/RCCOLUMNS/shear_failure.png" alt="Bridge column shear failure from the 1971 San Fernando Earthquake">
  <figcaption>Figure 2: Bridge column shear failure resulting from the 1971 San Fernando Earthquake, showing diagonal cracking characteristic of inadequate transverse reinforcement.</figcaption>
</figure>

As such, bridges constructed before this update in design practice are at high risk of catastrophic failure. This risk is compounded in the state of Washington by the unique seismic hazard from the Cascadia Subduction Zone (CSZ). In the Pacific Northwest, the CSZ divides the Juan de Fuca and North American tectonic plates. This plate boundary generates earthquakes with tremendous magnitudes (M9.0+) due to the nature of megathrust scenarios. Considering that the last major subduction zone rupture in this region occurred in 1700, researchers are concerned about the imminent threat of another rupture, which will severely damage large areas of the Cascadia coast. Cities like Seattle surrounding the Puget Sound additionally face a large seismic threat from these events. In this region, cities are built on deep sedimentary basins, which modify the ground motions significantly due to changes in the wave propagation medium. While they are located further from the CSZ, deep sedimentary basins amplify long period content (T > 1 sec), posing significant risk to structures that respond in that resonance band.

Berry et al. (2007) developed a comprehensive set of recommendations for modeling well-confined concrete columns in OpenSeesPy, using force-based nonlinear beam-column elements with fiber sections and distributed plasticity.[^berry2007] Initially, Kan-Jen Liu, a former graduate student at UW, developed 13 detailed bridge models to represent the variety of abutment configurations, column heights, and foundation types present in the Washington State inventory based on these recommendations. However, further study on the materials used in OpenSeesPy for the well-confined model revealed some issues when extending to the case of columns with low confinement, which account for a significant portion of columns in Washington State.

For the well-confined columns, Concrete04 in OpenSeesPy was used. This material model is based on the confined and unconfined concrete model from Mander et al. (1988).[^mander1988]

<figure>
  <img src="/images/RCCOLUMNS/Mander_concrete.png" alt="Mander confined and unconfined concrete stress-strain model">
  <figcaption>Figure 3: Stress-strain model for confined and unconfined concrete, showing first hoop fracture as the failure criterion for confined concrete (Mander et al., 1988).</figcaption>
</figure>

Key to note is that this model defines failure at first hoop fracture — crushing at $\varepsilon_{cu}$ and losing all compressive capacity.

Steel02, which defines a Menegotto-Pinto model[^menegotto1973] in OpenSeesPy, is used for the longitudinal reinforcement steel.

<figure>
  <img src="/images/RCCOLUMNS/steel.png" alt="Menegotto-Pinto cyclic steel stress-strain model">
  <figcaption>Figure 4: Menegotto-Pinto cyclic stress-strain model for longitudinal reinforcement steel as implemented in Steel02 (Monti et al., 1993).</figcaption>
</figure>

When plotting the performance of the element against experimental test results for well-confined columns, model results approximate the experimental force-displacement history reasonably well.

<figure>
  <img src="/images/RCCOLUMNS/lehman1015.png" alt="Force-displacement hysteresis for well-confined column lehman1015">
  <figcaption>Figure 5: Comparison of model (blue) and experimental (black) force-displacement response for well-confined column specimen lehman1015 using Steel02 + Concrete04 (Lehman et al., 1998).</figcaption>
</figure>

However, when considering the performance of the element against experimental test results for poorly confined columns, model results do not match the experimental force-displacement history, as the concrete in the column seems to fail too early.

<figure>
  <img src="/images/RCCOLUMNS/chai3.png" alt="Force-displacement hysteresis for poorly-confined column chai3">
  <figcaption>Figure 6: Comparison of model (blue) and experimental (black) force-displacement response for poorly-confined column specimen chai3 using Steel02 + Concrete04. The model fails to capture the strength degradation seen in the experimental data (Chai et al., 1991).</figcaption>
</figure>

This behavior is fixed by changing the concrete in the model to Concrete02, which defines a Kent-Scott-Park[^kent1971]<sup>,</sup>[^karsan1969] object that has a much higher $\varepsilon_{cu}$, with a linear decline in strength after peak compressive strength (still using the Mander parameters for $f'_{cc}$[^mander1988]), and some residual strength after crushing (0.1 $f'_{cc}$) defined by Zhou and Kunnath (2022).[^zhou2022]

<figure>
  <img src="/images/RCCOLUMNS/concrete02.png" alt="Concrete02 stress-strain response for inner core fiber">
  <figcaption>Figure 7: Concrete02 stress-strain response for the inner core fiber of a reinforced concrete column, showing the Kent-Scott-Park softening branch and residual compressive strength after crushing.</figcaption>
</figure>

Zhou and Kunnath (2022)[^zhou2022] provided a comprehensive column element for nonductile bridge columns, focusing on defining capacity limit states. Following their recommendations, the column is rebuilt with 4 Gauss-Lobatto integration points.

<figure>
  <img src="/images/RCCOLUMNS/Chai3_s02.png" alt="Force-displacement hysteresis for chai3 with Steel02 and Concrete02">
  <figcaption>Figure 8: Comparison of model (blue) and experimental (black) force-displacement response for poorly-confined column specimen chai3 using Steel02 + Concrete02. Switching to Concrete02 improves the match by extending the concrete crushing strain (Chai et al., 1991).</figcaption>
</figure>

<figure>
  <img src="/images/RCCOLUMNS/FE_scheme.png" alt="Distributed-plasticity finite element scheme with bond slip and shear components">
  <figcaption>Figure 9: Distributed-plasticity column model with force-based fiber beam-column element for flexure, aggregated elastic shear at each integration point, and a zero-length section for bond slip at the base (Berry & Eberhard, 2007).</figcaption>
</figure>

Additionally, the steel model is changed to capture degradation of the column near failure. Zhou and Kunnath (2022)[^zhou2022] defined a custom material model using the Hysteretic material in OpenSeesPy, which is one potential option. This model captures strain hardening and degradation on the tensile side, and buckling effects on the compression side. The buckling effects for lightly reinforced concrete columns are significant, as there is a larger effective length present for the rebar.

<figure>
  <img src="/images/RCCOLUMNS/Kunnath_tension.png" alt="Zhou and Kunnath hysteretic steel model — tension degradation">
  <figcaption>Figure 10: Hysteretic steel model capturing tension degradation and strain hardening effects for lightly reinforced columns (Zhou & Kunnath, 2022).</figcaption>
</figure>

<figure>
  <img src="/images/RCCOLUMNS/Kunnath_compression.png" alt="Zhou and Kunnath hysteretic steel model — compression buckling">
  <figcaption>Figure 11: Hysteretic steel model capturing compression buckling effects for lightly reinforced columns (Zhou & Kunnath, 2022).</figcaption>
</figure>

<figure>
  <img src="/images/RCCOLUMNS/NandP5_hystereticSM.png" alt="Force-displacement hysteresis for NandP5 with HystereticSM and Concrete02">
  <figcaption>Figure 12: Comparison of model (blue) and experimental (black) force-displacement response for poorly-confined column specimen NandP5 using HystereticSM + Concrete02 (Ranf et al., 2006).</figcaption>
</figure>

Potential concerns for this application include the in-cycle deterioration of the column, which could lead to strength losses larger than desired at large deformations, and buckling occurring too early in the displacement history.

<figure>
  <img src="/images/RCCOLUMNS/HystereticSM_concrete02.png" alt="Force-displacement hysteresis for chai3 with HystereticSM and Concrete02">
  <figcaption>Figure 13: Comparison of model (blue) and experimental (black) force-displacement response for poorly-confined column specimen chai3 using HystereticSM + Concrete02, capturing in-cycle strength degradation and rebar buckling effects (Chai et al., 1991).</figcaption>
</figure>

Another option is to provide MinMax wrappers to the Steel02 material. This wrapper essentially acts like an "off" switch once the steel reaches a prescribed strain in either compression or tension. This "fracture behavior" is seen in some of the later cycles, which allows degradation to occur in the model.

<figure>
  <img src="/images/RCCOLUMNS/NandP5_wrapper.png" alt="Force-displacement hysteresis for NandP5 with Steel02 MinMax wrapper and Concrete02">
  <figcaption>Figure 14: Comparison of model (blue) and experimental (black) force-displacement response for poorly-confined column specimen NandP5 using Steel02 with MinMax wrapper + Concrete02, simulating fracture behavior at prescribed strain limits (Ranf et al., 2006).</figcaption>
</figure>

---

[^berry2007]: Berry MP, Eberhard M. *Performance Modeling Strategies for Modern Reinforced Concrete Bridge Columns*, PEER Report 2007-07. Pacific Earthquake Engineering Research Center, 2007. <https://peer.berkeley.edu/publications/2007-07> [accessed August 23, 2025].
[^zhou2022]: Zhou J, Kunnath S. *Capacity Limit States for Nonductile Bridge Columns*. Pacific Earthquake Engineering Research Center, University of California, Berkeley, CA; 2022. DOI: [10.55461/FUTW2909](https://doi.org/10.55461/FUTW2909).
[^mander1988]: Mander JB, Priestley MJN, Park R. Theoretical Stress‐Strain Model for Confined Concrete. *Journal of Structural Engineering* 1988; 114(8): 1804–1826. DOI: [10.1061/(ASCE)0733-9445(1988)114:8(1804)](https://doi.org/10.1061/(ASCE)0733-9445(1988)114:8(1804)).
[^kent1971]: Kent DC, Park R. Flexural members with confined concrete. *Journal of the Structural Division, ASCE* 1971; 97(7): 1969–1990.
[^karsan1969]: Karsan ID, Jirsa JO. Behavior of concrete under compressive loading. *Journal of the Structural Division, ASCE* 1969; 95(12): 2543–2564.
[^ochshorn]: Ochshorn J. *Structural Elements for Architects and Builders*, Third Edition. Chapter 5: Reinforced Concrete Columns. <https://jonochshorn.com/structuralelements/book/5.06-columns.html>
[^ranf2006]: Ranf RT, Eberhard MO, Stanton JF. Effects of displacement history on failure of lightly confined bridge columns. *ACI Special Publications* 2006; 236: 23.
[^chai1991]: Chai YH, Priestley MJN, Seible F. *Flexural Retrofit of Circular Reinforced Concrete Bridge Columns by Steel Jacketing*. Report No. SSRP91/06. Department of Applied Mechanics and Engineering Sciences, University of California, San Diego, La Jolla, CA; 1991.
[^lehman1998]: Lehman DE, Calderone AJ, Moehle JP. Behavior and Design of Slender Columns Subjected to Lateral Loading. *Proceedings of the Sixth U.S. National Conference on Earthquake Engineering*. Oakland, CA: Earthquake Engineering Research Institute; 1998.
[^menegotto1973]: Menegotto M, Pinto E. Method of analysis for cyclically loaded reinforced concrete plane frames including changes in geometry and non-elastic behavior of elements under combined normal force and bending. *Proceedings, IABSE Symposium*. Lisbon, Portugal; 1973.
[^monti1993]: Monti G, Spacone E, Filippou FC. *Model for anchored reinforcing bars under seismic excitations*. Report No. UCB/EERC-93/08. Earthquake Engineering Research Center, University of California, Berkeley; December 1993.
