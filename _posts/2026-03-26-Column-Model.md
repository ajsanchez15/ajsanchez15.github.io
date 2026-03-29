---
title: 'Efficient Modeling of Lightly Reinforced Concrete Columns Under Cyclic Loading'
date: 2026-03-26
permalink: /posts/2026/03/26/RC-cylic
tags:
  - Reinforced Concrete
  - Earthquake Engineering
  - Bridges
---

This month, I spent a considerable amount of time developing a reinforced concrete column model in openseespy to match the behavior of cyclic tests stored in the PEER structures database. 

Concrete columns typically have two types of reinforcement, longitudinal and transverse. The former primarily reinforces the column against flexural demands, while the later reinforces the column against shear demands. 

![Rectangular tied column (a) and circular spiral column (b)](/images/RCCOLUMNS/Confinement.png)
*Figure: Containment of longitudinal bars using (a) rectangular ties and (b) spiral reinforcement.[^ochshorn]*

Significant changes were made to bridge design practices following the 1971 San Fernando Earthquake, which caused many bridge columns across the Los Angeles County transportation networks to fail in shear. Typical transverse reinforcment details for prior to this earthquake were #3 or #4 rebar at 12 inch center to center spacing, resulting in very low transverse reinforcment ratios of about 0.1-0.2%. 

![Bridge column shear failure from the 1971 San Fernando Earthquake](/images/RCCOLUMNS/shear_failure.png)
*Figure: Bridge column shear failure resulting from the 1971 San Fernando Earthquake*

As such, bridges constructed before this update in design practice are at high risk of catastrpohic failure. This risk is compounded in the state of Washington by the unique seismic hazard from the Cascadia Subduction Zone (CSZ). In the Pacific Northwest, the CSZ divides the Juan de Fuca and North American tectonic plates. This plate boundary generates earthquakes with tremendous magnitudes (M9.0+) due to the nature of megathrust scenarios. Considering that the last major subduction zone rupture in this region occurred in 1700, researchers are concerned about the imminent threat of another rupture, which will severely damage large areas of the Cascadia coast. Cities like Seattle surrounding the Puget Sound additionally face a large seismic threat from these events. In this region, cities are built on deep sedimentary basins, which modify the ground motions significantly due to changes in the wave propagation medium. While they are located further from the CSZ, deep sedimentary basins amplify long period content (T>1 sec), posing significant risk to structures that respond in that resonance band.

Berry et. al (2007) developed a comprehensive set of recomendations for modeling well confined concrete columns in openseespy, using force based nonlinear beam column elements with fiber sections and distributed plasticity.[^berry2007] Initially, Kan-Jen Liu, former graduate student at UW, developed 13 detailed bridge models to represent the variety of abutment configurations, column heights, and foundation types present in the Washington State inventory based on these recomendations. However, further study on the materials used in openseespy for the well confined model revealed some issues when extending to the case for columns with low confinment, which account for a significant portion of columns in Washington State. 

For the well confined columns, Concrete04 in openseespy was used. This material model is based on the confined and unconfined concrete model from Mander et al. (1988).[^mander1988]

![Mander confined and unconfined concrete stress-strain model](/images/RCCOLUMNS/Mander_concrete.png)
*Figure: Stress-strain model for confined and unconfined concrete, showing first hoop fracture as the failure criterion for confined concrete (Mander et al., 1988).[^mander1988]*

Key to note is that this model defines failure at first hoop fracture--crushing at /epsilon_cu and losing all compressive capacity. 

Steel02, which defines a Menegotto-Pinto model[^menegotto1973]in OpenSeesPy, is used for the longitudinal reinforcement steel.

![Menegotto-Pinto cyclic steel stress-strain model](/images/RCCOLUMNS/steel.png)
*Figure: Menegotto-Pinto cyclic stress-strain model for longitudinal reinforcement (Monti et al., 1993).[^monti1993]*

When plotting the performance of the element against expirmental test results for well confined columns, model results match the expiermental force-displacment history well. 

**IMAGE HERE**

However, when considering the performance of the element against expirmental test results for poorly confined columns, model results do not match the expiermental force-displacment history, as the concrete in the column seems to fail too early.

**IMAGE HERE**

This behavior is fixed by chaging the concrete in the model to Concrete02, which defines a Kent-Scott-Park[^kent1971]<sup>,</sup>[^karsan1969] object that has much higher epsilon_cu, with a linear decline in strength after peak compressive strength (still using the Mander parameters for fcc[^mander1988]), and some residual strength after crushing (0.1 fcc) defined by Zhou and Kunnath (2022).[^zhou2022] 

**IMAGE HERE**

Zhou and Kunnath (2022)[^zhou2022] provided a comprehesive column element for nonductile bridge columns, focuing on defining capactity limit states. Following their recomendations, the column is rebuilt with 4 Gauss-Lobbotto intergation points.

Additonally, the steel model is changed to capture degredation of the column near failure. Zhou and Kunnath (2022)[^zhou2022] defined a custom material model using the Hystertic material in openseespy, which is one potential option. This model captures strain hardening and degredation on the tensile side, and buckling effects on the compression side. The buckling effects for lightly reinforced concrete columns are significant, as there is a larger effective length present for the rebar.

![Zhou and Kunnath hysteretic steel model](/images/RCCOLUMNS/Kunnath_tension.png)
*Figure: Hysteretic steel model capturing tension degradation effects for lightly reinforced columns (Zhou & Kunnath, 2022).*
![Zhou and Kunnath hysteretic steel model](/images/RCCOLUMNS/Kunnath_compression.png)
*Figure: Hysteretic steel model capturing buckling effects for lightly reinforced columns (Zhou & Kunnath, 2022).*

One potential concern for this application is the incycle deteriation of the column, which 


Another option is to provide MinMax wrappers to the steel02 material. This wrapper essentially acts like an "off" switch once the steel reaches perscribed strain in either compression or tension. This "fracture behavior" is seen in some of the later cycles, which  allows degredation to occur in the model.


---

[^berry2007]: Berry MP, Eberhard M. *Performance Modeling Strategies for Modern Reinforced Concrete Bridge Columns*, PEER Report 2007-07. Pacific Earthquake Engineering Research Center, 2007. <https://peer.berkeley.edu/publications/2007-07> [accessed August 23, 2025].
[^zhou2022]: Zhou J, Kunnath S. *Capacity Limit States for Nonductile Bridge Columns*. Pacific Earthquake Engineering Research Center, University of California, Berkeley, CA; 2022. DOI: [10.55461/FUTW2909](https://doi.org/10.55461/FUTW2909).
[^mander1988]: Mander JB, Priestley MJN, Park R. Theoretical Stress‐Strain Model for Confined Concrete. *Journal of Structural Engineering* 1988; 114(8): 1804–1826. DOI: [10.1061/(ASCE)0733-9445(1988)114:8(1804)](https://doi.org/10.1061/(ASCE)0733-9445(1988)114:8(1804)).
[^kent1971]: Kent DC, Park R. Flexural members with confined concrete. *Journal of the Structural Division, ASCE* 1971; 97(7): 1969–1990.
[^karsan1969]: Karsan ID, Jirsa JO. Behavior of concrete under compressive loading. *Journal of the Structural Division, ASCE* 1969; 95(12): 2543–2564.
[^ochshorn]: Ochshorn J. *Structural Elements for Architects and Builders*, Third Edition. Chapter 5: Reinforced Concrete Columns. <https://jonochshorn.com/structuralelements/book/5.06-columns.html>
[^menegotto1973]: Menegotto M, Pinto E. Method of analysis for cyclically loaded reinforced concrete plane frames including changes in geometry and non-elastic behavior of elements under combined normal force and bending. *Proceedings, IABSE Symposium*. Lisbon, Portugal; 1973.
[^monti1993]: Monti G, Spacone E, Filippou FC. *Model for anchored reinforcing bars under seismic excitations*. Report No. UCB/EERC-93/08. Earthquake Engineering Research Center, University of California, Berkeley; December 1993.

