---
nocite: "[@sesp:2014]"
---

# Introduction {#chap:intro}

## Scope of this document

This Algorithm Theoretical Basis Document describes the algorithm
baseline for the  global Climate Data Record of Sea Ice Drift (OSI/̄455).

As described in , OSI/̄455 has a strong heritage from the near-real-time
(NRT) product OSI/̄405, which is why several of the algorithm steps are
common to the two product families.

Note that the  also produces a near-real-time sea-ice drift product
(OSI/̄407) from the visible and thermal-infrared imagery of the AVHRR
instruments.

## Reference and applicable documents

User related aspects of the product (like file format and output
specifications) are to be found in the Product User's Manual
([pum]{acronym-label="pum" acronym-form="singular+short"})
[\[\]](#doc:pum). Results from validation against on-ice drifters are
gathered in an associated Scientific Validation Report
([svr]{acronym-label="svr" acronym-form="singular+short"})
[\[\]](#doc:val).

### Reference documents

::: {#doc:val}
  ------------------------------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  $[\refidpum]$[]{#doc:pum label="doc:pum"}   EUMETSAT OSI SAF *Product User's Manual for the Low Resolution Sea Ice Drift Climate Data Record v1 (OSI-455)* SAF/OSI/CDOP3/MET/TEC/MA/418, version 1.0, August 2022
  $[\refidval]$                               EUMETSAT OSI SAF *Validation Report for the Low Resolution Sea Ice Drift Climate Data Record v1 (OSI-455)* SAF/OSI/CDOP3/MET/SCI/RP/419, version 1.0, August 2022
  ------------------------------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

### Applicable documents

::: {#doc:prd}
  --------------- ----------------------------------------------------------------------------------------------------------
  $[\refidprd]$   EUMETSAT OSI SAF *Product Requirements Document* SAF/OSI/CDOP3/MF/MGT/PL/2-001, version 1.90, 31/12/2021
  --------------- ----------------------------------------------------------------------------------------------------------
:::

[]{#sec:notations label="sec:notations"}

### Date and Time {#sec:notations:datetime}

As any image-based motion tracking algorithms, the low resolution sea
ice drift processing quantifies the amount of motion between two time
stamps by analysing the change in intensity patterns between two images.
Those are referred to as $\mathcal{I}_0$ (start image) and
$\mathcal{I}_1$ (stop image). Unless specified otherwise, subscript $0$
($1$) always refers to the *start* (*stop*) time stamp of the
displacement.

The  low resolution sea ice drift product is computed between daily maps
of satellite signals, as opposed to between individual satellite swaths
[@lavergne:2021:s2s]. $\mathbf{D}_0$ ($\mathbf{D}_1$) is the date for
the start (stop) of the displacement. $\mathbf{D}@\textrm{hh}$ can be
used to express a time stamp in a specific day. All hours are *UTC*.

### Grids and spatial projection {#sec:notations:projs}

To ensure consistency with other sea-ice CDRs from , ESA Climate Change
Initiative, and the Copernicus Climate Change, OSI/̄455 is prepared on an
Equal-Area Scalable Earth (EASE2) grid [@brodzik/etal:2012], with a
WGS84 geographic datum.

OSI/̄455 product files use two EASE2 polar grids (Northern and Southern
Hemisphere) with a 75 km grid spacing (first row in ). Two other grid
spacings (12.5 km and 25 km) are also used during the processing.

::: center
::: {#tab:gridsCDR}
  Id                                 n     n    $\mathbf{A}\subs{x}$ \[km\]   $\mathbf{A}\subs{y}$ \[km\]   $\mathbf{B}\subs{x}$ \[km\]   $\mathbf{B}\subs{y}$ \[km\]
  -------------------------------- ----- ----- ----------------------------- ----------------------------- ----------------------------- -----------------------------
  `nh_ease2-750`, `sh_ease2-750`    144   144              75.0                          75.0                         -5362.5                       -5362.5
  `nh_ease2-250`, `sh_ease2-250`    432   432              25.0                          25.0                         -5387.5                       5387.5
  `nh_ease2-125`, `sh_ease2-125`    864   864              12.5                          12.5                        -5393.75                       5393.75
:::
:::

[]{#tab:gridsCDR label="tab:gridsCDR"}

describes the mapping formulas from a geographical point
$(\lambda,\phi)$ into a $(x,y)$ point in the projection plane and,
eventually, into $[i,j]$ indexes in the two-dimensional array of the
grid. Those formulas are used in the rest of the document each time a
remapping is needed.

Cell coordinates always point to the center of a grid cell. Thus, point
$[i,j]$ is at the centre of a square grid cell (pixel) whose corner
points are $(i-\frac{1}{2},j-\frac{1}{2})$,
$(i-\frac{1}{2},j+\frac{1}{2})$, \...

### Seasons {#sec:notations:season}

The  sea-ice drift CDR has global coverage and thus covers all sea ice
in the Northern Hemisphere (NH) and Southern Hemisphere (SH). The
quality of the drift estimates vary with season, and it is convenient to
refer to winter, summer, and transition (spring and autumn) months. The
definition of these periods are in .

::: center
::: {#tab:seasons}
            Northern Hemisphere   Southern Hemisphere
  -------- --------------------- ---------------------
  Winter         Nov - Apr            Apr - Sept
  Spring            May                   Oct
  Summer        Jun - Sept             Nov - Feb
  Autumn            Oct                   Mar
:::
:::

[]{#tab:seasons label="tab:seasons"}

### Units {#sec:notations:units}

Unless otherwise stated, all vector components  and  stand for the sea
ice displacement *over a 24 hours period*, *along the two axis of the
EASE2 grid*. They have unit $km$.
