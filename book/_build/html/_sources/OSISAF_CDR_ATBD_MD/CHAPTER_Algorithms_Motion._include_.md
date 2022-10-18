[]{#chap:motion label="chap:motion"}

[]{#sec:motion:intro label="sec:motion:intro"}

As for many other motion
tracking methods applied in geophysics
([@nsidc:1998:drift-summary; @notarstefano:2007:sst-motion]), sea ice
drift is tracked from a pair of images, with a block-based strategy.
Each block (*feature*, *sub-image*, \...) is composed of a limited
ensemble of pixels from the first image (the *reference* block) and
centred at a tracking location, for which the most similar block in the
second image (the *candidate* block) is looked for. The degree of
similarity is assessed by a metric, which often is the correlation
coefficient between the reference and candidate blocks. The maximum
correlation indicates the best match and the two-dimensional offset
between the centre points of the two blocks is the drift vector.

The description given above applies to the well known
[mcc]{acronym-label="mcc" acronym-form="singular+short"} (Maximum Cross
Correlation) technique which has been successfully applied by many
investigators
([@ninnis:1986:avhrr-drift; @emery:1991:avhrr-drift; @kwok:1998:ssmi85-drift; @haarpaintner:2006:qscat-drift; @notarstefano:2007:sst-motion; @schmetz:1993:meteosat-wind],
among others). In the MCC, wthe search for the best candidate block is
*discrete* and *exhaustive*. Discrete since the offsets between the
centre points are in whole number of pixels and exhaustive because all
candidate blocks are evaluated before the best can be chosen.

The same description applies to the [cmcc]{acronym-label="cmcc"
acronym-form="singular+short"} (Continuous Maximum Cross Correlation)
method [@lavergne:2010:cmcc-jgr]. CMCC is the strategy developed and
implemented for the low resolution sea ice drift product of the .
Conversely to the MCC, the search for the best candidate vector is
performed in a continuous manner over the two-dimensional plane and, as
a consequence, the search algorithm is not exhaustive.

Although more complex to implement and, by nature, potentially less
robust than the MCC technique, the CMCC has the advantage of removing
the *quantization noise* which has hindered the retrieval of smooth
motion vector fields when the time span between the images is shortened
([@ezraty:2007:amsr-pum; @haarpaintner:2006:qscat-drift; @kwok:1998:ssmi85-drift]).

In the following sections, the motion tracking algorithm is further
described. It consists in three steps:

-   selection of tracking locations and preliminary screening ();

-   block based maximisation of the correlation metric via the CMCC ();

-   filtering and correction step ().

Finally, describes the approach to providing per grid-cell
uncertainties.

[]{#sec:motion:selection label="sec:motion:selection"} The start
locations for the drift arrows, the tracking locations, are distributed
over 75 km EASE2 grids (). The spacing is 6 times coarser than 12.5 km
image grids, so that each grid point in the 75 km grid falls exactly
every 6 grid point of the 12.5 km grids.

Both a nominal and a reduced block shape can be activated. The nominal
blocks approach a circular area of 137.5 km diameter : they are squares
with 11x11 pixels sides from which 3 grid cells are removed at the four
corners. The reduced blocks are squares with 62.5 km side (5x5 pixels).

As in [@haarpaintner:2006:qscat-drift], the reduced block is used close
to the ice edge, the coastline or an area with missing satellite data.
The aim is to try and estimate ice drift vectors also in these
situations. The reduced block is always used in place of the nominal
one, not at the same time.

Before any motion vector is retrieved, a screening algorithm browses
through all the start locations on the 75 km grid and filters away all
grid points which do not qualify for motion extraction (e.g. over land,
over the ocean, etc\...). In the process, the shape of the block that is
to be used (nominal or reduced) is decided upon for each drift location.

Each step of the screening process is applied on the list of drift
locations that passed all the previous steps. In the event where no grid
points are left after one of the screening steps, the motion processing
is skipped for that location. The first step is applied to the list of
all the drift locations from the 62.5 km grid.

Masking of land pixel (step 1)

:   Blocks whose centre pixel is over land are discarded;

Masking of pixels with not enough ice (step 2)

:   The start and stop blocks of the two ice masks are loaded. Discard
    the grid locations whose blocks are not totally over ice.

Masking of pixels with missing data (step 3)

:   The start and stop blocks of the Laplacian images are loaded.
    Discard the grid locations whose blocks hold missing data.

Detailed flow charts for each step are given in .

Start locations that are not discarded are passed on to the motion
tracking processing, along with the information on which block shape to
use for each of them.

[]{#sec:motion:cmcc label="sec:motion:cmcc"}

## Notations

We note $\mathcal{L}_0(x,y)[i]$ the $i^{th}$ pixel of the *start*
sub-image centred at point $(x,y)$, extracted from the $\mathcal{L}_0$
image. $(x,y)$ are the coordinates expressed in the underpinning
projection, see .

The total number of pixels $N$ in a sub-image depends on the block
shape, for the nominal blocks $N_n = 109$, for the reduced blocks
$N_r = 25$.

The mean and standard deviation values for a given sub-image are:
$$\begin{aligned}
\langle \mathcal{L}_0(x,y) \rangle & = & \frac{1}{N} \sum_{i=1}^{N} \mathcal{L}_0(x,y)[i] \nonumber \\
\sigma ( \mathcal{L}_0(x,y) ) & = & \sqrt{\langle \mathcal{L}^2_0(x,y) \rangle - \langle \mathcal{L}_0(x,y) \rangle^2}  \nonumber\end{aligned}$$
Similar values can be computed for a *stop* sub-image centred at
$(u,v)$: $\langle \mathcal{L}_1(u,v) \rangle$ and
$\sigma ( \mathcal{L}_1(u,v) )$.

As introduced in , the match between a start and a stop sub-image is
evaluated via the correlation metric: $$\rho(x,y,\delta_x,\delta_y) =
\frac{
\sum_{i=1}^{N} (\mathcal{L}_0(x,y)[i]-\langle \mathcal{L}_0(x,y) \rangle) (\mathcal{L}_1(x+\delta_x,y+\delta_y)[i]-\langle \mathcal{L}_1(x+\delta_x,y+\delta_y) \rangle)
}{\sigma ( \mathcal{L}_0(x,y) ) \sigma ( \mathcal{L}_1(x+\delta_x,y+\delta_y) )}
\label{eq:correl}$$ By construction, $\rho(x,y,\delta_x,\delta_y)$ takes
values between $-1$ and $+1$. High values indicate a good match between
the sub-images. This is further interpreted as having found the offsets
$\delta_x = u - x$ and $\delta_y = v - y$ which best explain the local
change in intensity between the two sub-images. $(\delta_x,\delta_y)$ is
the drift vector.

## On the fly computation of *virtual* sub-images

Pixels of the candidate block $\mathcal{L}_1(x+\delta_x,y+\delta_y)$ are
computed from bi-linear interpolations of the pixels of $\mathcal{L}_1$.
For example, $\mathcal{L}_1(u,v)[i]$ is given by: $$\begin{split}
\mathcal{L}_1(u,v)[i] & = (1 - \epsilon_{u}) \times (1 - \epsilon_{v}) \times \mathcal{L}_1(\bar{u},\bar{v})[i] \\
        &+  (1 - \epsilon_{u}) \times \epsilon_{v} \times  \mathcal{L}_1(\bar{u},\bar{v}+s_{v})[i] \\
        &+ \epsilon_{u} \times (1 - \epsilon_{v}) \times  \mathcal{L}_1(\bar{u}+s_{u},\bar{v})[i] \\
        &+ \epsilon_{u} \times \epsilon_{v} \times \mathcal{L}_1(\bar{u}+s_{u},\bar{v}+s_{v})[i]
\end{split}
\label{eq:img_interp}$$ where $$\begin{aligned}
\bar{t} & = & \operatorname{Trunc}(t) \\
\epsilon_t & = & |t - \bar{t}| \\
s_t        & = & \frac{t}{|t|}\end{aligned}$$ For example, for $t=-2.8$,
$\bar{t}=-2$, $\epsilon_t=0.8$ and $s_t=-1$. permits computing *virtual*
sub-images at continuously varying centre points $(u,v)$ and thus
building a continuous optimisation framework to the estimation of motion
vectors from a pair of images.

Finding the motion vector $(\delta_x,\delta_y)$ at position $(x,y)$ can
be expressed as the following maximisation problem:
$$\max_{(x,y)\in\mathcal{D}} \rho(x,y,\delta_x,\delta_y)
\label{eq:maxim}$$ which is solved at all $(x,y)$ points the motion
vector is searched for. Each optimisation is conducted independently
from the others. $\mathcal{D}$ is a validity domain for
$(\delta_x,\delta_y)$. thus defines a two dimensional optimisation
problem with domain constraint.

## Maximisation of $\rho(x,y,\delta_x,\delta_y)$

is solved by the Nelder Mead algorithm
([@nelder:1968:original; @Lagarias:1998:neldermead]). This algorithm was
chosen since it is simple to implement and does not require computing
the derivatives. It furthermore has good convergence and computational
properties in problems with low dimensionality.

### Initialisation

Starting points for the optimisation are sampled on a length-angle
regular grid around point $(0,0)$ as on .

![image](data/FigStartPoints/startpoints){width="50%"}
[]{#fig:startpoints label="fig:startpoints"}

The length increment is set to $10$km and the angular increment to
$45^{\circ}$. The circle has radius $\mathbf{L}$, the maximum drift
distance defining $\mathcal{D}$.

$\rho(x,y,\delta_x,\delta_y)$ is computed at each of those points and
the best 3 vertexes are kept for initialising the Nelder Mead
optimisation.

### Numerical convergence test

Termination and convergence is tested upon via a relative difference of
function values at the current *best* and *worst* vertexes, $f_b$ and
$f_w$. Specifically, the algorithm is said to converge if and only if
$| f_b - f_w | < (f_b + f_w) \times \tau + \epsilon$, with $\tau$ and
$\epsilon$ small and positive floating point values. As a safeguard, the
maximum number of iterations is set to 1000. No convergence test is
performed on the size and shape of the final simplex.

## Implementation of the validity domain $\mathcal{D}$

$\mathcal{D}$ is a disc shaped domain expressing the *a-priori*
knowledge we bring to the optimisation problem. Its purpose is to limit
the search area for the solution vector during the optimisation process.
It is defined by a centre point $(x_c,y_c)$ and radius $\mathbf{L}$.
$$(\delta_x,\delta_y) \in \mathcal{D}_{x_c,y_c} \Leftrightarrow d(x_c,y_c;\delta_x,\delta_y) < \mathbf{L}
\label{eq:domain}$$ In , $d(x_c,y_c;\delta_x,\delta_y)$ is the distance
(along the Earth surface) between the centre point of $\mathcal{D}$ and
the tip of the drift vector $(\delta_x,\delta_y)$ (see ). $(x_c,y_c)$
represents our best *a-priori* knowledge at the time of performing the
optimisation. It is initially set to $(0,0)$.

cannot be used *as is* in the optimisation routine since it leads to
abrupt and non-linear behaviour. $\mathcal{D}$ is instead implemented as
a *soft* constraint based on a mono-dimensional sigmoid function $W(d)$:
$$W(d)   = \frac{1}{1 + e^{k(d - \mathbf{L})}} 
\label{eq:sigmoid}$$ In , $k$ is a parameter controlling the steepness
of the sigmoid around the cut-off value $\mathbf{L}$. By construction,
$W(\mathbf{L}) = 0.5$. By using a large enough value for $k$, the $W$
can be made arbitrarily close to the Heaviside step function, yet
remaining smooth and continuous.

and illustrate how the penalty is applied to the correlation function
$\rho(x,y,\delta_x,\delta_y)$.
$$\rho_D(x,y,\delta_x,\delta_y) = (\rho(x,y,\delta_x,\delta_y) + 1) \times W(d(x_c,y_c;\delta_x,\delta_y)) - 1
\label{eq:penalising}$$

plots a mono-dimensional example of applying a sigmoid penalty function
to a synthetic correlation function. Evaluations for $x$ lower than L
are dominated by the correlation value $\rho(x)$ while those occurring
outside the domain ($x$ larger than $L$) return very bad scores, that is
close to $-1$.

![image](data/FigSigmoid/sigmoids){width=".9\\textwidth"}
[]{#fig:sigmoids label="fig:sigmoids"}

In , $\rho_D$ is the penalised correlation function. Finding the maximum
of $\rho_D$ is taken as a proxy for solving the original, constrained,
optimisation problem of . $\rho_D$ is the functional entering the Nelder
Mead algorithm.

It is customary to compute $\mathbf{L}$ as a maximum expected velocity
$v_{max}$, multiplied by the time separation between the two images
$\mathbf{D}_1$ - $\mathbf{D}_0$. $\mathbf{L}$ is thus the maximum
expected straight-line distance that can be covered in the given time.

For OSI/̄455, $\mathbf{D}_1$ - $\mathbf{D}_0$ is 1 day, and $v_{max}$ is
$0.45 m.s^{-1}$. In reality, individual ice floes can be recorded with
higher hourly velocities in dynamic areas, however $v_{max}$ corresponds
to a max speed averaged over 24 hours, and a spatial extent of
approximately 100 km. This value of $v_{max}$ is a compromise between
allowing very long drift distances, controlling computation time, and
the number of erroneous vectors (see section below).

[]{#sec:motion:filtering label="sec:motion:filtering"}
Once the motion
tracking processing described above has been applied to each of the
start position selected by the preliminary checks in , a filtering step
is taken to correct or remove obviously erroneous vectors.

Causes for those erroneous vectors include:

1.  convergence of the Nelder Mead algorithm in a local (non-global)
    maximum;

2.  noise in the sub-images;

3.  edge effects in the sub-images.

Whatever the reason be, the filtering step we implement is based on the
distance from individual displacement vectors to the average of its
neighbouring vectors. If this distance is less than a fixed threshold,
the displacement vector being tested is validated and another vector is
tested upon. Otherwise, a new motion tracking optimisation is triggered.
The Nelder Mead algorithm is initialised and run like in the previous
section, except that the validity domain $\mathbf{D}$ is adapted (center
and radius) to translate the new constraint.

Let $\Delta_{\textrm{avg}}$ be the distance between the tip of the
current drift vector $(\delta_x,\delta_y)$ and the tip of the zonal
average drift vector
$(\delta^{\textrm{avg}}_x,\delta^{\textrm{avg}}_y)$. The average drift
vector is computed from the 8 neighbouring drift vectors, that is the 8
closest vectors *not including the current one*. The local $\mathbf{D}$
domain is then the disc with centre
$(\delta^{\textrm{avg}}_x,\delta^{\textrm{avg}}_y)$ and radius
$\Delta^{\textrm{avg}}_{\textrm{max}}$. In the current implementation,
$\Delta^{\textrm{avg}}_{\textrm{max}}$ is set to $10$km. Neighbouring
vectors with a maximum correlation value of less than $0.5$ are not
used, to avoid degrading the average drift field with possibly wrong
estimates.

![image](data/FigFiltering/filter){width=".5\\textwidth"}
[]{#fig:filtering label="fig:filtering"}

illustrates a typical case where a single erroneous vector is surrounded
by a smooth vector field. Since the central estimate is not used in the
average, isolated wrong vectors stand out very easily in terms of
$\Delta_{\textrm{avg}}$.

During this second optimisation, the search for the maximum is limited
to the area eclosed by the red circle. If a satisfying maximum
correlation is found inside $\mathbf{D}$ it is kept and the surrounding
average vectors are immediately updated, as well as each
$\Delta_{\textrm{avg}}$ lengths. If the constrained optimisation does
not converge or if the new vector does not have a good enough maximum
correlation value, both the old and new vectors are discarded and the
average vectors, as well as $\Delta_{\textrm{avg}}$ at the neighbouring
locations are updated.

Although the method described above works in many cases, it sometimes
fail when several erroneous vectors are close one to each other. This
happens especially when noise dominates the signal in a large region of
one of the image. If the case, the order in which the vectors are
corrected has an influence on the final efficiency for the filtering.

To minimize this influence, motion vectors are first sorted from the
largest to the shortest $\Delta_{\textrm{avg}}$ and the filtering is
applied to the vector exhibiting the worst of those distances. Since,
changing a vector has an influence on its direct neighbours, the sorting
is repeated after each correction. A mechanism is put in place to avoid
falling into an infinite loop. This strategy also ensures that the good
vectors around an erroneous estimate are not modified before the latter
is actually processed through the filter ().

[]{#sec:motion:uncert label="sec:motion:uncert"}

To provide physically-based uncertainties on a per grid cell basis is an
effort for many if not all remote sensing products. In the field of ice
motion remote sensing, recent progress have been made for example by
[@sumata:2014:icedrift_uncert], [@sumata:2015:icedrift_uncert_summer],
[@sumata:2015:uncert_vs_SAR] or [@hollands:2014:icedrift_uncert_sar],
but so far no ice motion product comes with per-grid cell uncertainty
derived from uncertainty propagation.

Following our approach with the NRT sea-ice drift product OSI/̄405, we
provide uncertainties for each drift vector based on statistics obtained
from the validation against buoy trajectories, supplemented by a
dependency on the departure of the *start* time of each drift vector ()
to the daily central time .

## Uncertainties based on validation statistics {#sec:motion:uncertvalstat}

We base our single/̄sensor uncertainties on the statistics obtained from
validating the drift vectors against buoy trajectories. These validation
statistics are different for each sensor and are binned with the value
of `status_flag` (). In addition, it is clear from graphs in  that they
vary with season and hemisphere.

summarizes the values used as single/̄sensor uncertainties for the winter
seasons in each hemisphere (see for season definitions). For the SSM/I
and SSMIS sensors, the combined standard deviations are calculated from
a weighted average of the standard deviations for each mission according
to the number of buoy matchups (noted SSM/IS). The same is done for
[amsre]{acronym-label="amsre" acronym-form="singular+short"} and
[amsr2]{acronym-label="amsr2" acronym-form="singular+short"} (noted
AMSRs). The overall uncertainties are calculated from a simple mean of
the x- and y-components of RMSE to the buoy drift.

In , the columns are for different values of `status_flag` that are
defined in  and duplicated here for convenience:

-   30 (Nominal) : The vector was retrieved by first pass of CMCC,
    independently of others.

-   20 (Small Pattern) : The CMCC was applied with a smaller radius for
    the sub-images, due to the proximity to coast, edge or missing
    value.

-   21 (Corrected by neighbours): The vector was not retrieved in the
    first CMCC step but at the second pass, constrained using the
    neighbouring vectors.

::: center
::: {#tab:stats:valid}
  Hemisphere   Product    Nominal (30)   Small patt. (20)   Corr. neighb. (21)
  ------------ --------- -------------- ------------------ --------------------
  NH           AMSRs          1.7              3.3                 8.1
               SSMI/S         2.3              3.7                 8.0
  SH           AMSRs          2.8              5.3                 8.3
               SSMI/S         3.6              6.2                 8.7

  : Values of $\sigma_k$ (unit km) used for single/̄sensor product
  uncertainty in winter, depending on the type of satellite mission,
  value of `status_flag` and hemisphere. The uncertainties are an
  average of the and components after a 24 hours displacement (see ),
  and are used for both  and .
:::
:::

During the spring and autumn transition months (), the uncertainties
from are linearly weighted with the day-in-month so that there is a
smooth transition between the nominal values in winter and a high
uncertainty value (10 km) in summer.

In summer, the wind drift model (see Chapter
[\[chap:winds\]](#chap:winds){reference-type="ref"
reference="chap:winds"}) is used instead of the satellite-derived drift.
Thus, no summer satellite-based drift vectors are distributed to the
users, and they are not given uncertainties.

## Adapting uncertainties for time mis-registration

The uncertainty values provided in the previous section are valid when
using the single/̄sensor ice drift vectors with the space-varying start
 and stop  time information provided in the product files. These can
vary between and across the product grid, depending on the orbit and
instrument characteristics (see example in  for an illustration).

When  and  are not accounted for, and the vectors are used as if from
$\mathbf{D_0}$ at to $\mathbf{D_1}$ at , the uncertainties must be
raised. We adopt a 2nd order polynomial formula for this raised
uncertainty $\sigma_k^{12} (\delta_t)$ (). $$\begin{aligned}
\label{eq:uncert:deltaT}
\delta_t & = & \lvert t - t_0 \rvert \\
\sigma_k^{12} (\delta_t) & = & 0.015 \times \delta_{t}^2 - 0.005 \times \delta_{t} + \sigma_k \nonumber\end{aligned}$$
where $\delta_t$ has units hours, and $\sigma_k$ is that from the
previous section.

It is easily seen from that the uncertainties for vectors with  close to
will not be raised significantly from $\sigma_k$.

The numerical coefficients in were re-used from those of the NRT product
OSI/̄405. In brief, the polynomial shape was obtained by collocating
buoys and the OSI/̄405 satellite products with wrong time information,
deliberately misregistered from -10 hours to +10 hours. The mismatch was
fitted to determine the polynominal coefficients for .
