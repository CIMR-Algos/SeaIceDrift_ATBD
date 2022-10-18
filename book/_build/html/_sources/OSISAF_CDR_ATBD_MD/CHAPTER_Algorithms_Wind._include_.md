[]{#chap:winds label="chap:winds"}

[]{#sec:winds:intro label="sec:winds:intro"}

In the summer period, the sea ice drift from satellite motion tracking
is less reliable due to surface melting and increased liquid water and
vapour content in the atmosphere. The free-drift method is an
alternative measure of sea-ice drift from a theoretical model based on
wind speed when the stresses within the sea ice are neglected. This
method is less valid in the winter season, as the internal sea-ice
stresses can be too high. It can also be less reliable in case of low
wind speeds.

[]{#sec:winds:theory label="sec:winds:theory"}

The free drift model of ice drift is based on the 2-dimensional equation
of motion of sea ice when internal stresses are neglected
([@mozo:2017:wind-drift; @lepparanta:2011:seaicedrift]). The symbols
used are defined in Table [1](#tab:wind_tab1){reference-type="ref"
reference="tab:wind_tab1"}.

::: {#tab:wind_tab1}
  Definition of symbols   
  ----------------------- ----------------------------
  $h$                     sea ice thickness
  $\rho$                  sea ice density
  $\vec{u}$               sea ice velocity
  $t$                     time
  $f$                     Coriolis parameter
  $\vec{k}$               vertical unit vector
  $\nabla \cdot \sigma$   internal sea ice stress
  $\vec{\tau_a}$          wind stress
  $\vec{\tau_w}$          water stress
  $g$                     gravitational acceleration
  $\eta$                  sea surface elevation
  $p_a$                   atmospheric pressure
  $\vec{U_{wg}}$          geostrophic water velocity
  $\vec{U_{a}}$           surface wind velocity
  $\rho_a$                air density
  $\rho_w$                water density
  $C_a$                   air drag coefficient
  $C_w$                   water drag coefficient
  $\theta_a$              air turning angle
  $\theta_w$              water turning angle
:::

The 3-dimensional equation of motion integrated through the thickness of
the ice to obtain a 2-dimensional motion equation is

$$h \rho  \left( \underbrace{ \frac{\partial \vec{u}}{\partial t}}_{(1)} + \underbrace{\vec{u} \cdot \nabla\vec{u}}_{(2)} + \underbrace{f\vec{k} \times \vec{u}}_{(3)} \right) = \underbrace{\nabla \cdot \vec{\sigma}}_{(4)} + \underbrace{\vec{\tau_a}}_{(5)} + \underbrace{\vec{\tau_w}}_{(6)} - \underbrace{h \rho g \nabla \vec{\eta}}_{(7)} - \underbrace{h \nabla p_a}_{(8)}
  \label{eq:wind_eq1}$$

  ------- ------------------------- ------- ----------------------
  \(1\)   Local acceleration        \(5\)   Wind stress
  \(2\)   Advective acceleration    \(6\)   Water stress
  \(3\)   Coriolis acceleration     \(7\)   Sea surface gradient
  \(4\)   Internal sea ice stress   \(8\)   Air pressure
  ------- ------------------------- ------- ----------------------

The local and advective accelerations and the air pressure are
negligible. In the free drift model, which is a good approximation for
individual ice floes and non-compact ice fields, the internal sea-ice
stress is set to 0. With the further approximation that the Coriolis and
sea surface gradient term can be neglected, which is valid in the case
of moderate to high surface winds, this reduces simply to

$$\vec{\tau_a} + \vec{\tau_w} \approx  0
  \label{eq:wind_eq3}$$

The Coriolis effect causes the stress to not act in the direction of the
wind or the water current. This effect can be parameterised by turning
angles, so the wind stress can be written as

$$\vec{\tau_a} = \rho_a C_a |U_a| e^{i\theta_a} \vec{U_a}
  \label{eq:wind_eq4}$$

Due to the stronger coupling of the water and ice, the water stress is
calculated using the difference between the geostrophic current and the
sea-ice velocity. The water stress also includes a turning angle and can
be expressed as

$$\vec{\tau_w} = \rho_w C_w |U_{wg} - u| e^{i\theta_w} (\vec{U_{wg}} - \vec{u})
  \label{eq:wind_eq5}$$

With the addition of an error term, the sea-ice drift can finally be
expressed as

$$\vec{u} = A \vec{U_a} + \vec{U_{wg}} + \vec{e} \;\;\;\;\; \mathrm{where} \;\;\; A = \sqrt{\frac{\rho_aC_a}{\rho_wC_w}} e^{-i(\theta_w - \theta_a)}
  \label{eq:wind_eq6}$$

Equation [\[eq:wind_eq6\]](#eq:wind_eq6){reference-type="ref"
reference="eq:wind_eq6"} can be further written with matrices, such that

$$\vec{u} = Gm_{est} + \vec{e} \;\;\;\;\; \mathrm{where} \;\;\; G = \begin{pmatrix} \vec{U_a} & 1 \end{pmatrix} \;\;\;\;\; \mathrm{and} \;\;\; m_{est} = \begin{pmatrix} A \\ \vec{U_{wg}} \end{pmatrix}
  \label{eq:wind_eq7}$$

[]{#sec:winds:paramtuning label="sec:winds:paramtuning"}

Both sea ice drift and wind observations are used to solve for the model
parameters $m_{est}$ in an inverse problem (parameter tuning). A least
squares estimation to this problem where $\vec{e}$ is zero
([@menke:2012:dataanalysis]) is

$$%  m_{est} = [G^H G]^{-1} G^H \vec{u}
  m_{est} = [G^T G]^{-1} G^T \vec{u}
  \label{eq:wind_eq8}$$

where $G^T$ is the transpose of $G$.

The error term $e$, which provides information on the validity of the
model, is solved as []{#eq:wind_eqn6 label="eq:wind_eqn6"}

$$%  e = (I - G[G^H G]^{-1} G^H) \vec{u}
  e = (I - G[G^T G]^{-1} G^T) \vec{u}
  \label{eq:wind_eq9}$$

where $I$ is the identity matrix.

In order to generate monthly maps of wind-model parameters, sea-ice
drift data computed with the CMCC algorithm
([\[motion\]](#motion){reference-type="ref" reference="motion"}) from
the 36.5 GHz imagery of the [amsre]{acronym-label="amsre"
acronym-form="singular+short"} and [amsr2]{acronym-label="amsr2"
acronym-form="singular+short"} sensors for spring-summer-autumn periods,
as well as ERA5 wind data are used. The tuning of the free/̄drift model
thus covers only the period 2002/̄2020, and does not include the
[ssmi]{acronym-label="ssmi" acronym-form="singular+short"} and
[ssmis]{acronym-label="ssmis" acronym-form="singular+short"} missions.
This is because the near/̄90 GHz imagery of these missions does not allow
the retrieval of sea-ice motion during summer (surface melt and
atmospheric wetness), and neither do their 37 or 19 GHz imagery (too
coarse resolution).

For implementation reasons, parameter tuning ( and ) is actually solved
using complex numbers, with the x-component of the vectors being the
real part of the complex number and the y-component of the parameter
being the imaginary part.

A series of masking steps then take place. First, the parameter fields
are masked with a landmask and a monthly climatological maximum extent
mask.

.

Then, a gap-filling by extrapolation step is applied. This extrapolation
is necessary because we tune the wind-driven model on the later part of
the timeseries ([amsre]{acronym-label="amsre"
acronym-form="singular+short"} and [amsr2]{acronym-label="amsr2"
acronym-form="singular+short"}) but we must be able to apply the model
in the early decade of the CDR as well, when sea ice covered more area
(in the Arctic). The grid cells that neither have inverted parameters,
land, the \"no/̄ice\" region or the \"no/̄drift\" region are gap-filled by
extrapolation.

Finally, the parameters are smoothed using a Gaussian weighting
resampling method with a smaller radius to remove small speckle.

These maps of parameters and residuals are saved in monthly parameter
files. In Figures
[\[fig:inv_params_nh\]](#fig:inv_params_nh){reference-type="ref"
reference="fig:inv_params_nh"} and
[\[fig:inv_params_sh\]](#fig:inv_params_sh){reference-type="ref"
reference="fig:inv_params_sh"} examples of these parameters are shown.

![image](data/inv_params_nh_2013-2020_jul.jpg){width="95%"}
[]{#fig:inv_params_nh label="fig:inv_params_nh"}

![image](data/inv_params_sh_2012-2019_dec.jpg){width="95%"}
[]{#fig:inv_params_sh label="fig:inv_params_sh"}

A rapid analysis of these maps reveal that:

-   the wind transfer coefficient ($|A|$) takes expected values in the
    Arctic (about 2%);

-   the turning angle ($\theta$) is as expected around $-20^{\circ}$ in
    the Arctic.

-   the wind transfer coefficient ($|A|$) is somewhat larger in the
    Antarctic (thinner sea ice);

-   the turning angle ($\theta$) is as expected positive in the
    Antarctic around $+20^{\circ}$.

-   the magnitude of the residuals is smaller in the Antarctic than in
    the Arctic, indicating that the free-drift model is more valid for
    the Antarctic sea ice (less internal stress).

[]{#sec:winds:winddrift label="sec:winds:winddrift"}

The model parameters from are then used in equation
[\[eq:wind_eq7\]](#eq:wind_eq7){reference-type="ref"
reference="eq:wind_eq7"}: using wind velocity data to calculate sea ice
drift.

For each date, two ice drift maps are calculated using the ERA5 wind
valid for that date, and the parameter files from the two bracketing
months. The two ice drift maps then enter a weighted mean, according to
the day of the month: a linear interpolation is used to combine the two
ice drift maps in proportion to the time period between the current date
and the two bracketing months. This linear interpolation is to avoid
sudden changes in sea-ice drift vectors at the transition between
months.

A final step in the wind-drift calculation is to apply the sea-ice mask
for the current date, to ensure that wind-driven vectors are only
reported over sea ice. The ice mask is created from the daily
single/̄sensor satellite drift products from the same day.

[]{#sec:winds:winddriftuncert label="sec:winds:winddriftuncert"}

Uncertainties are calculated for the wind drift products from the
validation statistics, by matching buoy data to ice drift product data
in the same way as for the single/̄sensor satellite products (Section
[\[sec:motion:uncertvalstat\]](#sec:motion:uncertvalstat){reference-type="ref"
reference="sec:motion:uncertvalstat"}). Since we prepare daily
ERA5-winds directly from 12UTC as input to the wind drift calculation,
there is no need for the correction for the departure of the start time
 of the drift vectors.

The product uncertainty for the wind drift is shown in , which is
equivalent to for the single/̄sensor satellite products. The flags are
either \"nominal\" vectors (wind parameters directly inverted from drift
data), or \"gap-filled\" vectors extrapolation of the parameter maps was
involved.

::: center
::: {#tab:stats:validwind}
  Hemisphere   Product      Nominal (30)   Gapfill param (23)
  ------------ ----------- -------------- --------------------
  NH           era5-wind        2.6               1.8
  SH           era5-wind        3.0               5.2

  : Values of $\sigma_k$ (unit km) for wind drift uncertainty in summer,
  depending on value of `status_flag` and hemisphere. The uncertainties
  are an average of the  and  components after a 24 hours displacement
  (see ), and are used for both components.
:::
:::

We underline that the uncertainties reported in for the wind-driven
models are for the spring-summer-autumn months only. In winter,
wind-driven drift vectors are not processed.
