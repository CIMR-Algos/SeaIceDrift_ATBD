[]{#chap:multi label="chap:multi"}

The algorithms described in [\[preproc\]](#preproc){reference-type="ref"
reference="preproc"}, [\[motion\]](#motion){reference-type="ref"
reference="motion"}, and [\[winds\]](#winds){reference-type="ref"
reference="winds"} are the baseline for the various single/̄sensor ice
drift products and the wind-driven drift estimate.

The  sea ice drift CDR OSI/̄455 also includes a multi-sensor product,
that aims at presenting daily-complete maps of motion vectors throughout
the 30 years period, and in all seasons. The multi-sensor product can be
particularly useful for computation of trends, evaluation of model
simulations, and computing sea-ice trajectories. The present chapter
describes the processing steps involved in producing the multi-source
CDR of sea-ice drift.

The input to the multi/̄sensor ice drift algorithms are a series of
single/̄sensor products, obtained from the algorithms described in
[\[preproc\]](#preproc){reference-type="ref" reference="preproc"} and
[\[motion\]](#motion){reference-type="ref" reference="motion"}. In
addition, wind-driven drift fields
([\[winds\]](#winds){reference-type="ref" reference="winds"}) are used
to complement the satellite estimates. These wind/̄driven fields 1)
replace the single/̄sensor products in the summer season, 2) gapfill
whole winter days when satellite drift data is lacking, and 3) are
blended with the single/̄sensor satellite data in the spring and autumn
transition months.

Conversely to the NSIDC sea-ice motion dataset of
[@tschudi:2020:nsidcv4], we do not use buoy data as part of the merging.

The merging algorithm has the following steps for each date in the data
record:

1.  Selection of which single/̄sensor and wind products contributes to
    the merged product;

2.  Optimal merging or the selected sources at each grid locations;

3.  Interpolation from neighbours at grid locations with no drift
    information.

## Selection of single/̄sensor and wind drift vectors

Firstly, a check is made that there are enough vectors within each
single/̄sensor satellite product. This is done by first masking a wide
region around the pole hole (latitude $>$ 86N) and then calculating the
ratio between the number of drift vectors and the number of sea-ice grid
cells. A threshold is set at 40%, below which the entire single/̄sensor
map is discarded as not having sufficient data. This occurs more often
at the start of the Climate Data Record, where there are only one or two
satellite missions and some days with missing data. The mask over the
pole hole is to make sure the threshold is consistent across satellite
missions (with varying widths of the polar observation hole).

Very high-latitute Arctic ice drift vectors (latitude larger than 87.5N)
are not used in the merging if they have a `status_flag` different from
`nominal` (30). This is because the single/̄sensor products can be more
noisy there (borders of the swaths).

## Optimal merging of single/̄sensor drift vectors {#sec:multi:optmerging}

Here, the terminology *optimal* relates to the use of uncertainty
estimates in terms of variance, as weights in the merging formula. Let
$(u,v)_1$, $(u,v)_2$,\... be the $S$ available single-sensor drift
vectors at a given grid location (typically $S$ is between 2 - 4 in the
winter season). The multi-sensor drift vector $(u,v)_m$ at this same
location is computed as:
$$(u,v)_m = \frac{\sum_{k=1}^{S} (u,v)_k \times \frac{1}{(\sigma^{12}_k)^2}}{\sum_{k=1}^{S} \frac{1}{(\sigma^{12}_k)^2}}
   \label{eq:multi}$$

is a simplification of the full equation for combining several
multi-dimensional Gaussian estimates into an *optimal* (Maximum
Likelihood) estimate. The simplifications are:

1.  Both $u$ and $v$ components use the same value of $\sigma_k^{12}$;

2.  We do not consider correlation between the uncertainties of $u$ and
    $v$ components;

3.  We do not consider correlation between the uncertainties of the $S$
    single/̄sensor products;

4.  We do not consider correlation between the uncertainties of
    neighbouring vectors in the single/̄sensor products;

The simplifications above are used as pragmatic solution to decrease
computation time, as fully accounting for correlations would require
more advanced matrix computations. The uncertainties we use as weights
in are those documented in and for the single/̄sensor and wind drift
products, corrected for the temporal mismatch to (hence the
$\sigma^{12}$ notation).

Applying at all sea ice grid locations with at least one single/̄sensor
drift vector provides a new map of multi/̄sensor vectors $(u,v)_m$. This
new map might still have data gaps, which are filled by spatial
interpolation, as described in the next section.

Generally, the satellite/̄based single/̄sensor vectors are the only input
to the merging during the winter season, and the wind/̄driven drift
vectors the only input in the summer season.

In the transition months (spring and autumn, ), both single/̄sensor
satellite and wind-driven drift products contribute weighted by their
uncertainties, modified taking into account the day in the transition
month. An example of this, in the spring month, the uncertainties of the
CMCC-calculated drift vectors are at their nominal calculated values at
the start of the month (see Section
[\[sec:motion:uncert\]](#sec:motion:uncert){reference-type="ref"
reference="sec:motion:uncert"}), and at a high value (10km) at the end
of the month. The uncertainties on each spring day are found by linear
interpolation between these values. For the wind drift, the reverse is
true - the values are set to high (10km) at the start of spring month
and are nominal at the end. The same applies in the autumn, with the
uncertainties ramping in the opposite direction.

## Spatial interpolation for gap filling {#sec:multi:gapf}

Once the satellite and wind/̄derived drift vectors have been combined, a
gap-filling runs. The gap filling is handled with spatial interpolation.

The interpolation weight is function of the distance to neighbouring
grid cells, is gaussian-shaped, with a reference length of 200 km
(standard deviation). The maximum radius for finding neighbours is set
to 300 km. These distances are to be related to the nominal grid spacing
between vectors: 75 km.

All sea-ice drift vectors which are computed as a spatial interpolation
are flagged in the `status_flag` dataset (see ).

We do not fill gaps in grid cells where the single/̄sensor satellite
products never contains a drift vector, such as along coastlines and in
peripheral closed seas. Those cells are identified with the maximum ice
drift mask (introduced in ) and are reported in the `status_flag`
dataset (see ).

## Special cases {#sec:multi:specialcases}

The above describe the nominal multi/̄sensor approach, but there are
however special cases that we describe below.

When the removal of single/̄sensor products with low coverage results in
no valid single/̄sensor products in winter, this date is filled with the
wind/̄driven drift field to ensure that we provide some drift estimates
at all dates. This happens mostly in the first year of the data record
(1991) when the only available mission is ssmi/̄f10. This is reported in
the `status_flag` dataset (see ).

In the transition months (spring and autumn, ), when both single/̄sensor
satellite and wind-driven drift products are available, the wind product
is masked where there are no satellite/̄based drift vectors available.
This is to ensure that the wind-driven vectors, that always have a
complete coverage influence overly in these regions.

At rare occasions, the spatial interpolation step described in does not
reach far enough and leaves some cells empty. These remaining cells are
filled with wind/̄derived vectors. The vectors filled from spatial
interpolation and from wind/̄derived fields are given different
`status_flag` values (see ).

# Uncertainty associated to the multi/̄sensor product

From , the resulting uncertainty for the multi-sensor products are given
by:
$$\sigma_u = \alpha \times \sqrt{\frac{1}{\sum_{k=1}^{S} \frac{1}{(\sigma^{12}_k)^2}}}
   \label{eq:multi_uncert}$$ where $\alpha$ is an ad-hoc scaling factor
described below.

As noted in , the merging of single/̄source motion fields into the
multi/̄sensor product does not explicitly take into account a series of
correlations that might not be negligible. corresponds to having only
uncorrelated observations and might thus lead to too strong a reduction
of the merged uncertainty. The accumulated effects of these neglected
uncertainties is here partly compensated by the ad-hoc factor $\alpha$
($\alpha \gt 1$). The value $\alpha = 1.5$ was selected from looking at
the validation results of the multi/̄oi product (so that the
uncertainties computed by match the statistics of the mismatch of the
product to the buoy drift). is a simple but robust and cost-effective
solution. Future research will allow better formulation, but -as shown
in - it already performs well for our purposes.

The uncertainty for an interpolated vector () is estimated by
interpolating the uncertainties of the neighbouring vectors. In case of
gap/̄filling by the wind/̄derived vectors () the uncertainty is assigned
to fixed values from the results of the validation exercise (see ).
