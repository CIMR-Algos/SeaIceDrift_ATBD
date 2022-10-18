[]{#chap:preproc label="chap:preproc"}

[]{#sec:inputsat label="sec:inputsat"}

This section briefly describes the type of satellite data used as input
to the products. We focus here on the characteristics that are most
relevant to the processing of sea-ice motion.

## U.S. DoD DMSP SSM/I and SSMIS

The [ssmi]{acronym-label="ssmi" acronym-form="singular+short"} and
[ssmis]{acronym-label="ssmis" acronym-form="singular+short"} instruments
on board the [dmsp]{acronym-label="dmsp" acronym-form="singular+short"}
platforms of the US Department of Defence ([dod]{acronym-label="dod"
acronym-form="singular+short"}) have been the workhorse of sea-ice
remote sensing for decades. Together with its predecessor the
[smmr]{acronym-label="smmr" acronym-form="singular+short"} they allowed
monitoring of the polar regions since the late 1970s.

They are conically-scanning multi-frequency microwave radiometers with
imaging frequencies ranging from 19.3 GHz (Ku-band) to near/̄90 GHz
(85.5 GHz for SSM/I and 91.1 GHz for SSMIS). The two near/̄90 GHz
channels (one at Vertical polarization, the other at Horizontal
polarization) have the highest spatial resolution ( 13 x 14 km -3dB
diameter of the Instantaneous Field Of View -
[ifov]{acronym-label="ifov" acronym-form="singular+short"}). We use
these near/̄90 GHz channels (both polarizations) when preparing the
OSI/̄455 products.

OSI/̄455 uses SSM/I F10, F11, F13, F14, and F15 missions, as well as
SSMIS F16, F17, and F18.

[smmr]{acronym-label="smmr" acronym-form="singular+short"} had no
near/̄90 GHz imagery and the very first SSM/I mission (F08) had an early
failure of the 85.5 GHz channels. They are not used in this first first
version of OSI/̄455.

The Fundamental Climate Data Record ([fcdr]{acronym-label="fcdr"
acronym-form="singular+short"}) Release 4 from the
[cmsaf]{acronym-label="cmsaf" acronym-form="singular+short"} is our
input source of SSM/I and SSMIS data.

## JAXA AMSR-E and GCOM-W1 AMSR2

The [amsre]{acronym-label="amsre" acronym-form="singular+short"}
(2002-2011) and [amsr2]{acronym-label="amsr2"
acronym-form="singular+short"} (2012 onwards) instruments are
conically-scanning multi-frequency microwave radiometers, with imaging
frequencies ranging from 6.9 GHz (C-band) to 89.0 GHz (W-band). OSI/̄455
relies on the 36.5 GHz imagery (Ka-band, both polarizations) as it
offers the best compromise between retrieval accuracy and spatial
resolution (7 x 12 km). The 89 GHz channels have higher resolution, but
are also more affected by atmospheric liquid water path and surface
melting, and are consequently not used. The 18.7 GHz channel has
previously been found useful for sea-ice motion tracking during summer
[@kwok:2008:summer-drift] but our experience preparing OSI/̄455 was that
the 36.5 GHz imagery provided as good results when compared to buoy
trajectories.

[]{#sec:inputwinds label="sec:inputwinds"}

Wind fields from Numerical Weather Prediction (NWP) models are used to
drive a simplified (free-drift) model of sea-ice motion during the
summer season ([\[winds\]](#winds){reference-type="ref"
reference="winds"}). For OSI/̄455, wind vectors are accessed from the
C3S/ECMWF ERA5 reanalysis [@hersbach:2020:era5].

[]{#sec:icemask label="sec:icemask"}

We use land, ocean, and sea-ice cover information for the start and stop
dates to classify each cell in the image grid as:

1.  over land;

2.  close to coast;

3.  over open ice;

4.  over closed ice;

5.  in open water.

Land and open-water masks for the start and stop timestamps are taken
from the  v2 Climate Data Records of sea-ice concentration: OSI/̄450 and
OSI/̄430/̄b [@lavergne:2019:sicv2]. Both SIC CDRs are on the
`nh_ease2-250` and `sh_ease2-250` grids and remapped to `nh_ease2-125`
and `sh_ease2-125` for our purpose. No gap-filling is required since the
two SIC CDRs is already interpolated for missing data.

[]{#sec:preproc label="sec:preproc"}

## Daily averaging of swath data {#sec:tcimgs}

A daily average map of brightness temperatures is computed from all
swath in the range $\mathbf{D}@00$ to $\mathbf{D}@24$. Each satellite
FoV is remapped onto the `nh_ease2-125` and `sh_ease2-125` grids ().

We note $\gamma_t[i,j]$ an individual, pre-processed satellite
observation with sensing time $t$ ($t$ in hours, $t \in [0:24[$), with
nearest neighbour cell $[i,j]$.

### Temporal weight during the integration window {#sec:tcimgs:time}

For reducing the span of sensing time contributing to each location, yet
ensuring spatial continuity, a temporal weighting function
$\mathcal W_T(t)$ is applied:
$$W_T(t) = - \frac{1}{12} \times | 12 - t | + 1
\label{eq:time-weighting}$$ As can be seen from , $W_T(0) = W_T(24) = 0$
and $W_T(12) = 1$, with linear variations around the central $t = 12$
value.

### Spatial weight into neighbouring pixels {#sec:tcimgs:space}

Because the spatial sampling of the sensors we are interested in is
close to the remapping grid resolution (12.5km), it is necessary to
implement a limited area spatial weighting for each $\gamma_t[i,j]$. In
this scheme, each $\gamma_t[i,j]$ contributes to cell $[i,j]$ as well as
to the 8 neighbouring grid cells $[n,m]$ with weight
$\mathcal W^{i,j}_S(n,m)$: $$\begin{aligned}
\label{eq:spatial-weighting}
\mathcal W^{i,j}_S(n,m)    & = & \exp(-0.5 \times \frac{\ell^2}{\sigma_{\ell}^2}) \\
n \in [i-1,i+1] & \textrm{ and } & m \in [j-1,j+1] \nonumber\end{aligned}$$

In , $\ell(n,m,i,j)$ is the cartesian distance between the center of the
FoV and the center of the grid cell (using a spherical Earth
approximation) and $\sigma_{\ell}$ is a distance controlling the shape
of the gaussian weight function, and tuned visually for each microwave
channel and target grid resolution.

The weighting function defined by does not take into account the shape
and orientation of the actual footprint but it proves enough for the
purpose of filling the gaps that would result from a simplistic
nearest-neighbour gridding strategy. $\sigma_{\ell}$ is an important
tuning parameter: too large a value will coarsen the resulting image and
thus degrade the spatial resolution, too small a value will create
gridding artefacts that can fool the motion tracking algorithm.

### Weighting equation {#sec:tcimgs:weqn}

Finally, the satellite signal value assigned to $\mathcal{I}[i,j]$ is
computed as a weighted average over several $\gamma_t[n,m]$,
$t \in [0,24[$ and $(n,m) \in [i-1,i+1]\times[j-1,j+1]$:
$$\begin{aligned}
\label{eq:tcimgs:weqn}
\mathcal{I}[i,j] & = & \frac{1}{\mathcal{K}} \sum_{n=i-1}^{i+1} \sum_{m=j-1}^{j+1} \sum_{k=1}^{k(n,m)} W^{n,m}_S[i,j] \times W_T(t(k)) \times \gamma_{t(k)}[n,m] \\
\mathcal{K} & = & \sum_{n=i-1}^{i+1} \sum_{m=j-1}^{j+1} \sum_{k=1}^{k(n,m)} W^{n,m}_S[i,j] \times W_T(t(k)) \nonumber\end{aligned}$$
$k(n,m)$ is the number of $\gamma_t$ that contribute to cell $[n,m]$.
$t(k)$ is the sensing time associated to each of them.

At this stage, there is no special treatment for pixels classified as
land, sea-ice or open water. is adapted along the borders of the grid.

### Mean sensing time {#sec:tcimgs:avgt}

is not only applied to compute $\mathcal{I}[i,j]$ from $\gamma_t$ but
for the *average sensing time* $\mathcal{T}[i,j]$ as well. The latter is
an important quantity as it translates into position-dependent start and
stop time for the drift vectors. It is computed by only with $t(k)[n,m]$
in place for $\gamma_{t(k)}[n,m]$.

### Effect of the temporal weighting function $W_T(t)$ {#sec:tcimgs:remarkOnWt}

The temporal weighting introduced in act as a sharpening filter towards
the central time of the averaging period. It aims at reducing the
*motion blur* which arises from the steady displacement of sea ice
during the averaging period of the daily image.

## Feature enhancement with Laplacian filtering {#sec:lap}

As in [@ezraty:2007:qscat-pum], a Laplacian filter is applied to the
average image computed in . It aims at:

-   removing intensity gradients across the image plane as well as
    between the start and stop image;

-   enhancing intensity patterns;

The Laplacian (or Laplace operator) $\nabla^2$ is closely linked to the
second derivatives of multi-dimensional functions
([@www:wikipedia-LaplaceOperator]) and is computed as:
$$\begin{split}
\mathcal{L}[i,j] &= \frac{1}{N^{+}} \sum_{n=i-1}^{i+1} \sum_{m=j-1}^{j+1}
                       \delta_{\textrm{na}}(n,m) \delta_{\textrm{ice}}(n,m) \delta_1^{i,j}(n,m) \mathcal{I}[n,m] \\
                 &- \frac{1}{N^{-}} \sum_{n=i-2}^{i+2} \sum_{m=j-2}^{j+2}
               \delta_{\textrm{na}}(n,m) \delta_{\textrm{ice}}(n,m) \delta_2^{i,j}(n,m) \mathcal{I}[n,m] 
\end{split}
\label{eq:lap}$$ with $$\begin{aligned}
\delta_k^{i,j}(n,m) & = & \left\{ \begin{array}{ll} 1 & \mbox{if $|i-n| = k$ or $|j-m| = k$};\\ \nonumber
             0 & \mbox{otherwise}.\end{array} \right.   \\ \nonumber
N^{+} &=& \sum_{n=i-1}^{i+1} \sum_{m=j-1}^{j+1} \delta_{\textrm{na}}(n,m) \delta_{\textrm{ice}}(n,m) \delta_1^{i,j}(n,m)  \leq  8 \\ \nonumber
N^{-} &=& \sum_{n=i-2}^{i+2} \sum_{m=j-2}^{j+2} \delta_{\textrm{na}}(n,m) \delta_{\textrm{ice}}(n,m) \delta_2^{i,j}(n,m)  \leq  16 \nonumber\end{aligned}$$
In , $\delta_{\textrm{na}}(n,m)$ has value $0$ if $\mathcal{I}[n,m]$ is
non available (a missing value from the swath) and
$\delta_{\textrm{ice}}(n,m)$ is $1$ only over *open or closed ice*, as
specified by the ice mask (). It means that only *valid*, *sea ice*
pixels enter the Laplacian field in order to limit spurious features
along the ice edge, coastline or close to the polar observation hole.

$\mathcal{L}[i,j]$ is only computed if the centre cell $[i,j]$ is itself
over sea ice, $\delta_{\textrm{ice}}(i,j)=1$.

In the event when $N^{+} < 5$ or $N^{-} < 9$, not enough pixels are
available for computing $\mathcal{L}$ and a missing value is stored at
grid cell $[i,j]$.

Note that, conversely to [@ezraty:2007:qscat-pum], no median or further
filtering is applied on the Laplacian image.

\(a\)

\(b\)

\(c\)

[]{#fig:tcimgs+time+lap label="fig:tcimgs+time+lap"}

Example images of the information available at the end of this
pre-processing step are given in . The ice motion tracking algorithm is
applied on the pair of $\mathcal{L}_0$ and $\mathcal{L}_1$ images. It is
described in [\[motion\]](#motion){reference-type="ref"
reference="motion"}.

## Pre-processing of wind data

The $u$ and $v$ components of the 10 m wind velocities () are remapped
and rotated onto the 75 km EASE2 grids of the sea-ice drift product. The
hourly wind components are then time-averaged across a 1 day period
(12utc to 12utc) that correspond to the time-span of the drift products.
