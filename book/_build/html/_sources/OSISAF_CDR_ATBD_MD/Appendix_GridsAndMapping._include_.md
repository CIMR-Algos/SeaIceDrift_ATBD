[]{#chap:grids label="chap:grids"}

In this section, we introduce the formulas and equations entering the
reprojection steps during both the preprocessing and the motion
tracking.

[]{#sec:app:mapping label="sec:app:mapping"}

For OSI/̄455, we use the Equal-Area Scalable Earth (EASE) grid version 2
([@brodzik/etal:2012]), which is a Lambert Azimuthal Equal Area (LAEA)
projection on a WGS84 Geographic Datum. We use the PROJ library
([@www:PROJlib]) and its python wrapper for transforming back and forth
between (longitude, latitude) and (x, y).

[]{#sec:app:gridding label="sec:app:gridding"} If inside the grid
limits, a $(x,y)$ point is enclosed in a grid cell $[i,j]$. Grids are
defined by number of lines and columns, spatial resolution and offsets
(see ). The *gridding* process is controlled by following equations:
$$\begin{aligned}
\label{eq:griddingI}
i   &=&  \round \left ( \frac{x*0.001 - \mathbf{B}_x}{\mathbf{A}_x}  \right ) \\
\label{eq:griddingJ}
j   &=&  \round \left ( -\frac{y*0.001 - \mathbf{B}_y}{\mathbf{A}_y} \right ) \\
(i,j) & \in & [0,n_x-1] \times [0,n_y-1] \nonumber\end{aligned}$$ The
asymmetry between and
[\[eq:griddingJ\]](#eq:griddingJ){reference-type="ref"
reference="eq:griddingJ"} is because point
$(x,y) \equiv (\mathbf{B}_x,\mathbf{B}_y)$ defines the *upper*-right
corner of the grid.

The inverse relationships to and
[\[eq:griddingJ\]](#eq:griddingJ){reference-type="ref"
reference="eq:griddingJ"} are thus: $$\begin{aligned}
x   &=&  \left ( i * \mathbf{A}_x +  \mathbf{B}_x  \right ) \times 1000 \nonumber \\
y   &=&  \left ( -j * \mathbf{A}_y +  \mathbf{B}_y  \right ) \times 1000 \nonumber \end{aligned}$$

[]{#sec:app:other label="sec:app:other"} The distance $L$ along the
Earth shape between two $(\lambda_0,\phi_0)$ and $(\lambda_1,\phi_1)$
points is given by: $$\begin{aligned}
\label{eq:EarthDist}
\Delta_{\lambda} &=& \frac{1}{2} (\lambda_1 - \lambda_0) \nonumber \\
\Delta_{\phi} &=& \frac{1}{2} (\phi_1 - \phi_0) \nonumber \\
\mathbf{a} &=& \sin^2(\Delta_{\lambda})+ \cos(\lambda_0)\cos(\lambda_1)*\sin^2(\Delta_{\phi}) \nonumber \\
\mathbf{c} &=& 2 \atantwo \left ( \sqrt{ \mathbf{a}},\sqrt{ \mathbf{1-a}} \right ) \nonumber \\
L &=& \textrm{R}_{\textrm{Earth}} * \mathbf{c} \end{aligned}$$ is valid
for a spherical Earth only, which is not exactly compatible with our
choice of datum, but is a good enough approximation for all practical
purposes.

Computing the distance between two $(x_0,y_0)$ and $(x_1,y_1)$ points
only requires the prior processing through the forward mapping operator
().
