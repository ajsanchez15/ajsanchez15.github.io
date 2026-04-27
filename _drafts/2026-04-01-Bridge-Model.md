---
title: 'Large Scale Seismic Simulations for Highway Bridges in Washington State'
date: 2026-04-01
permalink: /posts/2026/04/01/bridge-seismic-simulations
tags:
  - Earthquake Engineering
  - Bridges
---


Following my investigation into the reinforced concrete column element, it is now time to integrate that work into the larger scale UW bridge simulation code!

University of Washington researchers (thanks undergrads!) have compiled a detailed database of bridges across the state, focusing on critical routes that include I-90 between Seattle and Snoqualmie Pass, I-5 between Tacoma and Everett, I-405, SR12, HW101, and others.

Detailed study was conducted on these selected bridges to identify key characteristics of reference bridges for the inventory. Fourteen models in total were built, using numerical modeling strategies drawn from Ranf (2006),[^ranf2006] Ramanathan (2012),[^ramanathan2012] and Mangalathu et al. (2017).[^mangalathu2017] These models consider types of abutments (integral, semi-integral, and L-seat), pile and spread footing in the abutment, differing foundations, presence of internal shear keys and elastomeric bearings, transverse reinforcement ratios, and height of the columns. Two design periods are specified as pre-1976 and 1976–2018 to reflect updated code changes in required transverse reinforcement detailed in my previous post.

The reference bridge model was developed in OpenSeesPy (McKenna et al., 2010)[^mckenna2010] to simulate the seismic response of a typical three-span, two-bent reinforced concrete highway bridge in Western Washington. Thirteen models in total were built, using numerical modeling strategies drawn from previous efforts in this field. The models include detailed representations of the superstructure, intermediate bents, and abutments based on the inventory assessment.

As shown in Figure 3, the bridge superstructure was modeled using elastic beam-column elements, while the nonlinear flexural response of the columns was captured with force-based beam-column elements (Lehman & Moehle, 1998).[^lehman1998] At the ends of each column, zero-length fiber sections were added to model anchorage slip and bond flexibility (Berry & Eberhard, 2007).[^berry2007] Including these effects makes the simulation more realistic by representing the reduction in effective stiffness and increase in drift demands observed during cyclic loading of reinforced concrete columns.

<figure>
  <img src="/images/bridges/figure3-bridge-fem-schematic.png" alt="OpenSees bridge FEM schematic: elastic deck and cap beams, fiber force-based columns with bond-slip zero-length sections, zero-length abutment springs">
  <figcaption>Figure 3: Numerical modeling of bridge components—elastic superstructure and cap beams, fiber-section force-based nonlinear columns with bond-slip, and zero-length elements for abutments and foundations (OpenSeesPy).</figcaption>
</figure>

Concrete behavior was modeled using the Concrete02 material model, with separate zones for confined and unconfined concrete to reflect the improved ductility provided by transverse reinforcement; the reinforcing steel was represented with the Steel02 model with a MinMax wrapper to model bar failure in compression and tension.

Different abutment systems were modeled in detail to capture soil–structure interaction effects (Figure 4)—end bearing in compression and tension, shear key action, and foundation flexibility (both pile-supported and spread footings).

The abutment backfill soil is modeled using approaches established by Shamsabadi & Yan (2012)[^shamsabadi2012] and implemented by Mangalathu et al. (2017),[^mangalathu2017] using nonlinear springs for passive and active backfill response and the `HyperbolicGapMaterial` in OpenSeesPy. Abutment foundation modeling considered both a spread footing and a pile foundation, with models from Duncan et al. (1994)[^duncan1994] and hysteretic material models in OpenSeesPy from Mangalathu et al. (2017).[^mangalathu2017] Internal shear keys were modeled with an initial gap of 0.5 inches (Megally, 2002)[^megally2002] and used hysteretic material models from Mangalathu et al. (2017).[^mangalathu2017] Elastomeric bearing pads again used recommendations from Mangalathu et al. (2017)[^mangalathu2017] and the Steel02 model in OpenSeesPy. Pounding of the backwall follows a material model proposed by Muthukumar & DesRoches (2006)[^muthukumar2006] and implemented by Mangalathu et al. (2017)[^mangalathu2017] as an `ImpactMaterial` in OpenSeesPy. Variation in abutment foundation soil is introduced, with the reference bridge using Medium Sand, and a separate model built for Medium Clay.
This detailed approach allowed the analysis to distinguish the performance of integral, semi-integral, and L-type (seat-type) abutments, and to evaluate how variations in abutment type and soil conditions influence the bridge's seismic response.

<figure>
  <img src="/images/bridges/figure4-abutment-spring-schematics.png" alt="Spring idealizations for L-type and semi-integral abutments (top) and integral abutments (bottom), with 2D schematics and 3D deck–abutment connectivity">
  <figcaption>Figure 4: Zero-length spring systems for L-type and semi-integral abutment types (upper) and for integral abutments (lower), modified from Mangalathu et al. (2017).[^mangalathu2017]</figcaption>
</figure>

The study used a set of simulated M9 ground motions developed by USGS and UW researchers in line with 3D Cascadia simulations (Frankel et al., 2018)[^frankel2018] to understand the seismic demand on bridges in Western Washington. The baseline motions were generated from 30 rupture scenarios that capture different rupture patterns, hypocenter locations, and subevents, representing variability in large CSZ earthquakes.

Specifically, three cities in Western Washington—Ocean Shores, Port Angeles, and Seattle (Figure 1)—were picked to capture different basin depths and distances from the fault plane, which strongly influence seismic response.

<figure>
  <img src="/images/bridges/figure1-washington-csz-sites.png" alt="Map of Washington state showing Ocean Shores, Port Angeles, and Seattle relative to the Cascadia Subduction Zone">
  <figcaption>Figure 1: Study cities and the Cascadia Subduction Zone offshore of Washington.</figcaption>
</figure>

Base ground motions were further adjusted for site-specific soil effects. Following Marafi et al. (2021),[^marafi2021] soil profiles were categorized into site subclasses C2, C4, D1, and D3 based on shear-wave velocity ($V_{S30}$). For each site class, ground motion realizations were run through a sample of ten soil profiles used to generate a large suite of motions for parametric studies.

<figure>
  <img src="/images/bridges/figure2-median-response-spectra.png" alt="Median 5% damped response spectra for Ocean Shores, Seattle, and Port Angeles by baseline and site classes C2, C4, D1, and D3">
  <figcaption>Figure 2: Median site-adjusted response spectra at Ocean Shores, Seattle, and Port Angeles by soil site class (baseline, C2, C4, D1, D3) for 30 Cascadia Subduction Zone earthquake scenarios.</figcaption>
</figure>

## References

1. Ranf R. *Performance Based Evaluation of Seismic Modeling Strategies for Bridges*. PhD thesis, University of Washington, 2006.
2. Ramanathan KN. *Next Generation Seismic Fragility Curves for California Bridges Incorporating the Evolution in Seismic Design Philosophy*. Ph.D. thesis, 2012. [Online]. Available: <https://ui.adsabs.harvard.edu/abs/2012PhDT.......173R> [accessed July 10, 2025].
3. Chen S, Xie Y, Wu C, Burton HV, Padgett JE. High-resolution regional seismic loss assessment of reinforced concrete bridges using component-level fragility models and repair cost estimations. *Earthquake Engineering & Structural Dynamics* [early view]. DOI: [10.1002/eqe.70083](https://doi.org/10.1002/eqe.70083).
4. Nielson BG, DesRoches R. Seismic fragility methodology for highway bridges using a component level approach. *Earthquake Engineering & Structural Dynamics* 2007; 36(6): 823–839. DOI: [10.1002/eqe.655](https://doi.org/10.1002/eqe.655).
5. Mangalathu S, Jeon J-S, Padgett JE, DesRoches R. Performance-based grouping methods of bridge classes for regional seismic risk assessment: Application of ANOVA, ANCOVA, and non-parametric approaches. *Earthquake Engineering & Structural Dynamics* 2017; 46(14): 2587–2602. DOI: [10.1002/eqe.2919](https://doi.org/10.1002/eqe.2919).
6. Wu C, Burton HV, Zsarnóczay Á, et al. Modeling post-earthquake functional recovery of bridges. *Earthquake Spectra* 2025. Published online May 1, 2025. DOI: [10.1177/87552930251321301](https://doi.org/10.1177/87552930251321301).
7. Nielson BG, DesRoches R. Analytical seismic fragility curves for typical bridges in the central and southeastern United States. *Earthquake Spectra* 2007; 23(3): 615–633. DOI: [10.1193/1.2756815](https://doi.org/10.1193/1.2756815).
8. Shao Y, Xie Y. Seismic risk assessment of highway bridges in western Canada under crustal, subcrustal, and subduction earthquakes. *Structural Safety* 2024; 108: 102441. DOI: [10.1016/j.strusafe.2024.102441](https://doi.org/10.1016/j.strusafe.2024.102441).
9. Pacific Northwest Seismic Network. Cascadia Subduction Zone. Published online 2021. <https://pnsn.org/outreach/earthquakesources/csz>
10. Marafi NA, Eberhard MO, Berman JW, Wirth EA, Frankel AD. Effects of deep basins on structural collapse during large subduction earthquakes. *Earthquake Spectra* 2017; 33(3): 963–997. DOI: [10.1193/071916eqs114m](https://doi.org/10.1193/071916eqs114m).
11. Marafi NA. *Impacts of an M9 Cascadia Subduction Zone Earthquake on Structures Located in Deep Sedimentary Basins*. Ph.D. thesis, University of Washington, 2019. Published online February 22, 2019. <http://hdl.handle.net/1773/43338> [accessed October 16, 2024].
12. Frankel A, Wirth E, Marafi N, Vidale J, Stephenson W. Broadband synthetic seismograms for magnitude 9 earthquakes on the Cascadia megathrust based on 3D simulations and stochastic synthetics, Part 1: Methodology and overall results. *Bulletin of the Seismological Society of America* 2018; 108(5A): 2347–2369. DOI: [10.1785/0120180034](https://doi.org/10.1785/0120180034).
13. Wirth EA, Frankel AD, Marafi N, Vidale JE, Stephenson WJ. Broadband synthetic seismograms for magnitude 9 earthquakes on the Cascadia megathrust based on 3D simulations and stochastic synthetics, Part 2: Rupture parameters and variability. *Bulletin of the Seismological Society of America* 2018; 108(5A): 2370–2388. DOI: [10.1785/0120180029](https://doi.org/10.1785/0120180029).
14. De Zamacona Cervantes G. *Response of Idealized Structural Systems to Simulated M9 Cascadia Subduction Zone Earthquakes Considering Local Soil Conditions*. Master’s thesis, University of Washington, 2019.
15. AASHTO. *LRFD Bridge Design Specifications*. American Association of State Highway and Transportation Officials, 2017.
16. Kortum Z. *Impacts of Cascadia Subduction Zone M9 Earthquakes on Bridges in Washington State: SDOF Idealized Bridges*. Master’s thesis, University of Washington, 2021.
17. Deierlein G, Moehle J. *A Framework Methodology for Performance-Based Earthquake Engineering*.
18. Ning C, Xie Y, Burton H, Padgett JE. Enabling efficient regional seismic fragility assessment of multi-component bridge portfolios through Gaussian process regression and active learning. *Earthquake Engineering & Structural Dynamics* 2024; 53(9): 2929–2949. DOI: [10.1002/eqe.4144](https://doi.org/10.1002/eqe.4144).
19. Nassar M, Abo El Ezz A. Fragility-based seismic risk assessment of reinforced concrete bridge columns. *Infrastructures* 2025; 10(5): 123. DOI: [10.3390/infrastructures10050123](https://doi.org/10.3390/infrastructures10050123).
20. Zhou J, Kunnath S. *Capacity Limit States for Nonductile Bridge Columns*. Pacific Earthquake Engineering Research Center, University of California, Berkeley, CA, 2022. DOI: [10.55461/FUTW2909](https://doi.org/10.55461/FUTW2909).
21. Marafi NA, Grant A, Maurer BW, Rateria G, Eberhard MO, Berman JW. A generic soil velocity model that accounts for near-surface conditions and deeper geologic structure. *Soil Dynamics and Earthquake Engineering* 2021; 140: 106461. DOI: [10.1016/j.soildyn.2020.106461](https://doi.org/10.1016/j.soildyn.2020.106461).
22. Federal Highway Administration. National Bridge Inventory. Published online 2024.
23. Federal Highway Administration. NBI Element Data. Published online 2024.
24. Ranf RT. *Model Selection for Performance-Based Earthquake Engineering of Bridges*.
25. Lehman D, Moehle J. *Seismic Performance of Well-Confined Concrete Bridge Columns*, PEER Report 1998-01. Pacific Earthquake Engineering Research Center, 1998. <https://peer.berkeley.edu/publications/1998-01> [accessed October 2, 2025].
26. McKenna F, Scott MH, Fenves GL. Nonlinear finite-element analysis software architecture using object composition. *Journal of Computing in Civil Engineering* 2010; 24(1): 95–107. DOI: [10.1061/(ASCE)CP.1943-5487.0000002](https://doi.org/10.1061/(ASCE)CP.1943-5487.0000002).
27. Shamsabadi A, Yan L. Closed-form force-displacement backbone curves for bridge abutment-backfill systems. Published online June 20, 2012. DOI: [10.1061/40975(318)159](https://doi.org/10.1061/40975(318)159).
28. Duncan JM, Evans LT, Ooi PSK. Lateral load analysis of single piles and drilled shafts. *Journal of Geotechnical Engineering* 1994; 120(6): 1018–1033. DOI: [10.1061/(ASCE)0733-9410(1994)120:6(1018)](https://doi.org/10.1061/(ASCE)0733-9410(1994)120:6(1018)).
29. Megally S. Seismic response of sacrificial shear keys in bridge abutments. 2002. <https://www.researchgate.net/publication/268394682_Seismic_response_of_sacrificial_shear_keys_in_bridge_abutments> [accessed October 4, 2025].
30. Muthukumar S, DesRoches R. A Hertz contact model with non-linear damping for pounding simulation. *Earthquake Engineering & Structural Dynamics* 2006; 35(7): 811–828. DOI: [10.1002/eqe.557](https://doi.org/10.1002/eqe.557).
31. Berry MP, Eberhard M. *Performance Models for Flexural Damage in Reinforced Concrete Columns*, PEER Report 2003-18. Pacific Earthquake Engineering Research Center, 2003. <https://peer.berkeley.edu/publications/2003-18> [accessed August 23, 2025].
32. Berry MP, Eberhard M. *Performance Modeling Strategies for Modern Reinforced Concrete Bridge Columns*, PEER Report 2007-07. Pacific Earthquake Engineering Research Center, 2007. <https://peer.berkeley.edu/publications/2007-07> [accessed August 23, 2025].
33. Kowalsky MJ, Priestley MJN. Improved analytical model for shear strength of circular reinforced concrete columns in seismic regions. [Online]. <https://www.concrete.org/publications/internationalconcreteabstractsportal.aspx?m=details&ID=4633> [accessed September 18, 2025].
34. Baker JW. Efficient analytical fragility function fitting using dynamic structural analysis. *Earthquake Spectra* 2015; 31(1): 579–599. DOI: [10.1193/021113EQS025M](https://doi.org/10.1193/021113EQS025M).

---

[^ranf2006]: Ranf R. *Performance Based Evaluation of Seismic Modeling Strategies for Bridges*. PhD thesis, University of Washington, 2006.

[^ramanathan2012]: Ramanathan KN. *Next Generation Seismic Fragility Curves for California Bridges Incorporating the Evolution in Seismic Design Philosophy*. Ph.D. thesis, 2012. <https://ui.adsabs.harvard.edu/abs/2012PhDT.......173R> [accessed July 10, 2025].

[^mangalathu2017]: Mangalathu S, Jeon J-S, Padgett JE, DesRoches R. Performance-based grouping methods of bridge classes for regional seismic risk assessment: Application of ANOVA, ANCOVA, and non-parametric approaches. *Earthquake Engineering & Structural Dynamics* 2017; 46(14): 2587–2602. DOI: [10.1002/eqe.2919](https://doi.org/10.1002/eqe.2919).

[^frankel2018]: Frankel A, Wirth E, Marafi N, Vidale J, Stephenson W. Broadband synthetic seismograms for magnitude 9 earthquakes on the Cascadia megathrust based on 3D simulations and stochastic synthetics, Part 1: Methodology and overall results. *Bulletin of the Seismological Society of America* 2018; 108(5A): 2347–2369. DOI: [10.1785/0120180034](https://doi.org/10.1785/0120180034).

[^marafi2021]: Marafi NA, Grant A, Maurer BW, Rateria G, Eberhard MO, Berman JW. A generic soil velocity model that accounts for near-surface conditions and deeper geologic structure. *Soil Dynamics and Earthquake Engineering* 2021; 140: 106461. DOI: [10.1016/j.soildyn.2020.106461](https://doi.org/10.1016/j.soildyn.2020.106461).

[^mckenna2010]: McKenna F, Scott MH, Fenves GL. Nonlinear finite-element analysis software architecture using object composition. *Journal of Computing in Civil Engineering* 2010; 24(1): 95–107. DOI: [10.1061/(ASCE)CP.1943-5487.0000002](https://doi.org/10.1061/(ASCE)CP.1943-5487.0000002).

[^lehman1998]: Lehman D, Moehle J. *Seismic Performance of Well-Confined Concrete Bridge Columns*, PEER Report 1998-01. Pacific Earthquake Engineering Research Center, 1998. <https://peer.berkeley.edu/publications/1998-01> [accessed October 2, 2025].

[^berry2007]: Berry MP, Eberhard M. *Performance Modeling Strategies for Modern Reinforced Concrete Bridge Columns*, PEER Report 2007-07. Pacific Earthquake Engineering Research Center, 2007. <https://peer.berkeley.edu/publications/2007-07> [accessed August 23, 2025].

[^shamsabadi2012]: Shamsabadi A, Yan L. Closed-form force-displacement backbone curves for bridge abutment-backfill systems. Published online June 20, 2012. DOI: [10.1061/40975(318)159](https://doi.org/10.1061/40975(318)159).

[^duncan1994]: Duncan JM, Evans LT, Ooi PSK. Lateral load analysis of single piles and drilled shafts. *Journal of Geotechnical Engineering* 1994; 120(6): 1018–1033. DOI: [10.1061/(ASCE)0733-9410(1994)120:6(1018)](https://doi.org/10.1061/(ASCE)0733-9410(1994)120:6(1018)).

[^megally2002]: Megally S. Seismic response of sacrificial shear keys in bridge abutments. 2002. <https://www.researchgate.net/publication/268394682_Seismic_response_of_sacrificial_shear_keys_in_bridge_abutments> [accessed October 4, 2025].

[^muthukumar2006]: Muthukumar S, DesRoches R. A Hertz contact model with non-linear damping for pounding simulation. *Earthquake Engineering & Structural Dynamics* 2006; 35(7): 811–828. DOI: [10.1002/eqe.557](https://doi.org/10.1002/eqe.557).

[^aashto2017]: AASHTO. *LRFD Bridge Design Specifications*. American Association of State Highway and Transportation Officials, 2017.
