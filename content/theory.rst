.. _theory:

Background theory
=================

Introduction
------------

The EM1DFM and EM1DFMFWD programs are designed to interpret frequency-domain, small loop, electromagnetic data over a 1D layered Earth.
These programs model the Earth's frequency-domain electromagnetic response due to a small inductive loop source which carries a sinusoidal time-varying current. 
The data are the secondary magnetic field which results from currents and magnetization induced in the Earth.

.. _theory_source:

Details regarding the source and receiver
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

EM1DFM and EM1DFMFWD assume that the transmitter loop size is sufficiently small that it may be represented by a magnetic dipole.
The programs also assume that each receiver loop is sufficiently small and that they may be considered point receivers; i.e.
the spatial variation in magnetic flux through the receiver loop is negligible.

.. _theory_domain:

Details regarding the domain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^


.. figure:: ../images/domain.png
     :align: right
     :figwidth: 50%

     Layered 1D model describing the Earth for each sounding.


EM1DFM and EM1DFMFWD model the Earth's response for measurements above a stack of uniform, horizontal
layers. **The coordinate system used for describing the Earth models has z as positive downwards, with the
surface of the Earth at z=0**. The z-coordinates of the source and receiver locations (which must be above
the surface) are therefore negative. Each layer is defined by a thickness, an electrical conductivity, and a magnetic susceptibility (if desired).
It is the physical properties of the layers that are obtained during the inversion of all measurements
gathered at a single location, with the depths to the layer interfaces remaining fixed. If measurements made
at multiple locations are being interpreted, the corresponding one-dimensional models are juxtaposed to
create a two-dimensional image of the subsurface.

Because horizontal position is meaningless in a model that varies only vertically, all measurements that
are to be inverted for a single one-dimensional model must be grouped together as a “sounding”.
A sounding can be considered a distinct collection of FEM measurements (transmitters, receivers and frequencies). Each
different sounding is inverted for a separate one-dimensional model. The horizontal location at which a
sounding is centered is not used within the program, but is written out to distinguish results for different
soundings.

.. figure:: ../images/soundings_domain.png
     :align: center
     :width: 700

     Two distinct groupings of transmitters and receivers (soundings) at the same location (left). Different soundings used to map lateral variation in the Earth (right). **Click to enlarge**.


The electrical conductivity of Earth materials varies over several orders of magnitude, making it more
natural to invert for the logarithms of the layer conductivities rather than the conductivities themselves.
This also ensures that conductivities in the constructed model are positive. It is not appropriate, however, to
invert for the logarithms of the layer susceptibilities. Near-zero values of susceptibility would become overly
important and large values would be overestimated (:cite:`Zhang1997`). Therefore susceptibilities
are found directly by the inversion, with the option of imposing a positivity constraint.


.. _theory_data:

Details regarding the data
^^^^^^^^^^^^^^^^^^^^^^^^^^

For a magnetic dipole source oriented in the x, y and/or z-direction, EM1DFM and EM1DFMFWD can handle any combination of:

    - frequencies for the transmitter current
    - separations and heights for the transmitter and receiver(s)
    - in phase and quadrature components of the response
    - x, y and z-components of the response

Programs EM1DFM and EM1DFMFWD accept observations in four different forms: values of the
secondary magnetic field (total minus free-space) normalized by the free-space field and given in parts-per-
million, values of the secondary field normalized by the free-space field and given in percent, values of the
secondary H-field in A/m, and values of the total H-field in A/m. If the transmitter and receiver have
the same orientation, the x-, y- and z-components of the secondary field are normalized by the
x-, y- and z-components of the free-space field respectively. If the transmitter and receiver have different orientations,
the secondary field is normalized by the magnitude of the free-space field.

.. _theory_fwd:

Forward Modeling
----------------

.. note:: This code uses a left-handed coordinate system with X (Easting), Y (Northing) and Z (+ve downwards) with a time-dependency of :math:`e^{+ i\omega t}`.

The method used to compute the magnetic field values for a particular source-receiver arrangement over a
layered Earth model is the matrix propagation approach described in Farquharson (:cite:`Farquharson2003`). The method
uses the z-component of the Schelkunoff F-potential (:cite:`Ward1987`):

.. math::
    \begin{align}
    \mathbf{E} &= -\nabla \times \mathbf{F} \\
    \mathbf{H} &= -\sigma \mathbf{F} + \dfrac{1}{i \omega \mu} \nabla \big ( \nabla \cdot \mathbf{F} \big )
    \end{align}
    :name: Schelkunoff

where :math:`\mathbf{E}`, :math:`\mathbf{H}`, :math:`\sigma` and :math:`\mu` are the electric field, magnetic field,
conductivity and magnetic permeability, respectively, within the uniform region for which this equation is valid. Note that the time-dependence :math:`e^{i\omega t}` has been suppressed.
The permeability is related to the susceptibility (:math:`\kappa`) via the following equation:

.. math::
    \mu = \mu_0 \big ( 1 + \kappa \big )
    :name: susc_def

where :math:`\mu_0` is the permeability of free space. 
In the :math:`j^{th}` layer (where :math:`j>0`) with conductivity :math:`\sigma_j` and permeability :math:`\mu_j`, the
z-component of the Schelkunoff potential satisfies the following equation (assuming the quasi-static approximation):

.. math::
    \nabla^2 F_j - i\omega \mu_j \sigma_j F = 0
    :name: Helmholtz

Applying the two-dimensional Fourier transform to eq. :eq:`Helmholtz` gives:

.. math::
    \frac{d^2 \tilde{F}_j}{dz^2} - u_j^2 \tilde{F}_j = 0
    :name: Helmholtz1D

where :math:`u_j^2 = k_x^2 + k_y^2 + i \omega \mu_j \sigma_j`, and :math:`k_x` and :math:`k_y` are the horizontal wavenumbers. The
solution to this equation is:

.. math::
    \tilde{F}_j (k_x,k_y,z,\omega ) = D_j (k_x, k_y, \omega) \, e^{-u_j (z-z_j)} + U_j (k_x, k_y, \omega) \, e^{u_j (z-z_j)}
    :name: Helmholtz_gen_sol

where :math:`D_j` and :math:`U_j` are the coefficients of the downward and upward-decaying parts of the solution, respectively. At
the interface between layer :math:`j-1` and layer :math:`j`, which is at depth :math:`z_j`, the conditions on :math:`\tilde{F}` are:

.. math::
    \begin{align}
    \tilde{F}_{j-1} \Big |_{z=z_j} &= \tilde{F}_j \Big |_{z=z_j}, \\
    \dfrac{1}{\mu_{j-1}} \dfrac{d \tilde{F}_{j-1}}{dz} \Bigg |_{z=z_j} &= \dfrac{1}{\mu_{j}} \dfrac{d \tilde{F}_{j}}{dz} \Bigg |_{z=z_j}
    \end{align}
    :name: bound_cond

Applying these conditions to the solutions for :math:`j \geq 2` gives:

.. math::
    \begin{bmatrix} e^{-u_{j-1} t_{j-1}} & e^{u_{j-1} t_{j-1}} \\ - \frac{u_{j-1}}{\mu_{j-1}} e^{-u_{j-1} t_{j-1}} & \frac{u_{j-1}}{\mu_{j-1}} e^{u_{j-1} t_{j-1}} \end{bmatrix}
    \begin{bmatrix} D_{j-1} \\ U_{j-1} \end{bmatrix} =
    \begin{bmatrix} 1 & 1 \\ -\frac{u_j}{\mu_j} & \frac{u_j}{\mu_j} \end{bmatrix}
    \begin{bmatrix} D_{j} \\ U_{j} \end{bmatrix},
    :name: Layer_soln

where :math:`t_{j-1} = z_j - z_{j-1}` is the thickness of layer :math:`j-1`. Through factoring and rearranging, the above equation can be re-expressed as:

.. math::
    \begin{bmatrix} D_{j-1} \\ U_{j-1} \end{bmatrix} =
    e^{u_{j-1}t_{j-1}} \mathbf{M_j} \begin{bmatrix} D_{j} \\ U_{j} \end{bmatrix},
    :name:

where

.. math::
    \mathbf{M_j} = \begin{bmatrix} \frac{1}{2} \Big ( 1 + \frac{\mu_{j-1} u_j}{\mu_j u_{j-1}} \Big ) & \frac{1}{2} \Big ( 1 - \frac{\mu_{j-1} u_j}{\mu_j u_{j-1}} \Big ) \\
    \frac{1}{2} \Big ( 1 - \frac{\mu_{j-1} u_j}{\mu_j u_{j-1}} \Big ) e^{-2u_{j-1} t_{j-1}} & \frac{1}{2} \Big ( 1 + \frac{\mu_{j-1} u_j}{\mu_j u_{j-1}} \Big ) e^{-2u_{j-1} t_{j-1}} \end{bmatrix}
    \;\;\;\; \textrm{for} \;\;\;\; j \geq 2
    :name:

In layer 0 (the air interface), :math:`\tilde{F}` is given by:

.. math::
    \tilde{F}_0 = D_0 e^{-u_0 z} + U_0 e^{u_0 z},
    :name:

which leads to

.. math::
    \begin{bmatrix} D_0 \\ U_0 \end{bmatrix} = \mathbf{M_1} \begin{bmatrix} D_1 \\ U_1 \end{bmatrix}
    :name: Layer_soln_0

and

.. math::
    \mathbf{M_1} = \begin{bmatrix} \frac{1}{2} \Big ( 1 + \frac{\mu_0 u_1}{\mu_1 u_0} \Big ) & \frac{1}{2} \Big ( 1 - \frac{\mu_0 u_1}{\mu_1 u_0} \Big ) \\
    \frac{1}{2} \Big ( 1 - \frac{\mu_0 u_1}{\mu_1 u_0} \Big ) & \frac{1}{2} \Big ( 1 + \frac{\mu_0 u_1}{\mu_1 u_0} \Big ) \end{bmatrix}
    :name:

Using eqs. :eq:`Layer_soln` and :eq:`Layer_soln_0`, we can relate the coefficients :math:`U_0` and :math:`D_0` of the solution in the air to the coefficients :math:`U_M` and :math:`D_M` of
the solution in the basement halfspace:

.. math::
    \begin{bmatrix} D_0 \\ U_0 \end{bmatrix} = \mathbf{M_1} exp \Bigg ( \sum_{j=2}^M u_{j-1} t_{j-1} \Bigg ) \prod_{j=2}^M \mathbf{M_j} \begin{bmatrix} D_M \\ U_M \end{bmatrix}
    :name: Matrix_soln

There is no upward-decaying part of the solution in the basement halfspace (thus :math:`U_M = 0`). In the air, the
downward-decaying part is due to the source (thus :math:`D_0 = D_0^s`). Eq. :eq:`Matrix_soln` can therefore be rewritten as:

.. math::
    \begin{bmatrix} D_0^2 \\ U_0 \end{bmatrix} = E \, \mathbf{ P} \begin{bmatrix} D_M \\ 0 \end{bmatrix}
    :name: Matrix_soln2

where the matrix :math:`\mathbf{P}` is given by

.. math::
    \mathbf{P} = \mathbf{M_1} \prod_{j=2}^M \mathbf{M_j}
    :name: M_prod

and the factor :math:`E` is given by:

.. math::
    E = exp \Bigg ( \sum_{j=2}^M u_{j-1} t_{j-1} \Bigg )
    :name:

From eq. :eq:`Matrix_soln2`, we see that:

.. math::
    D_M = \frac{1}{E} \frac{1}{P_{11}} D_0^s
    :name:

and

.. math::
    U_0 = E \, P_{21} \, D_M
    :name:

Substituting the previous two equations gives:

.. math::
    U_0 = \frac{P_{21}}{P_{11}} D_0^s
    :name:

which does not involve any exponential terms whose arguments have positive real parts, making this formulation inherently stable.
The solution for :math:`\tilde{F}` in the air halfspace is therefore given by:

.. math::
    \tilde{F}_0 = D_0^s \Bigg ( e^{-u_0 z} + \frac{P_{21}}{P_{11}} e^{u_0 z} \Bigg )
    :name: Final_soln

For a unit vertical magnetic dipole source at a height :math:`h` (i.e. :math:`z = -h` for :math:`h>0`) above the surface of the Earth:

.. math::
    D_0^s = \frac{i\omega \mu_0}{2 u_0}e^{-u_0 h}
    :name: Source_vert

(:cite:`Ward1987`, eq. 4.40), and for a unit x-directed magnetic dipole source at :math:`z=-h`:

.. math::
    D_0^s = - \frac{i\omega \mu_0}{2} \frac{ik_x}{k_x^2 + k_y^2} e^{-u_0 h}
    :name: Source_horiz

(:cite:`Ward1987`, eq. 4.106). Once whichever of these terms is appropriate is substituted into
eq. :eq:`Final_soln`, the solution is completed by converting the required inverse two-dimensional Fourier transform to
a Hankel transform, and using eq. :eq:`Schelkunoff` to obtain the three components of the H-field above the Earth model (:math:`z<0`).
For a z-directed magnetic dipole source at (:math:`0,0,-h`) such that :math:`h>0`:

.. math::
    \begin{align}
    H_x(x,y,z,\omega) &= \frac{1}{4\pi} \frac{x}{r} \int_0^\infty \Big ( e^{-\lambda |z+h|} - \frac{P_{21}}{P_{11}} e^{\lambda (z-h)} \Big ) \lambda^2 J_1(\lambda r) d\lambda \\
    H_y(x,y,z,\omega) &= \frac{1}{4\pi} \frac{y}{r} \int_0^\infty \Big ( e^{-\lambda |z+h|} - \frac{P_{21}}{P_{11}} e^{\lambda (z-h)} \Big ) \lambda^2 J_1(\lambda r) d\lambda \\
    H_z(x,y,z,\omega) &= \frac{1}{4\pi}             \int_0^\infty \Big ( e^{-\lambda |z+h|} + \frac{P_{21}}{P_{11}} e^{\lambda (z-h)} \Big ) \lambda^2 J_0(\lambda r) d\lambda
    \end{align}
    :name: Soln_zdip

And for a x-directed magnetic dipole source at (:math:`0,0,-h`) such that :math:`h>0`:

.. math::
    \begin{align}
    H_x(x,y,z,\omega) =& -\frac{1}{4\pi} \Big ( \frac{1}{r} - \frac{2x^2}{r^3} \Big ) \int_0^\infty \Big ( e^{-\lambda |z+h|} - \frac{P_{21}}{P_{11}} e^{\lambda (z-h)} \Big ) \lambda J_1(\lambda r) d\lambda \\
    &-\frac{1}{4\pi} \frac{x^2}{r^2} \int_0^\infty \Big ( e^{-\lambda |z+h|} - \frac{P_{21}}{P_{11}} e^{\lambda (z-h)} \Big ) \lambda^2 J_0(\lambda r) d\lambda \\
    H_y(x,y,z,\omega) =& \frac{1}{2\pi} \frac{xy}{r^3} \int_0^\infty \Big ( e^{-\lambda |z+h|} - \frac{P_{21}}{P_{11}} e^{\lambda (z-h)} \Big ) \lambda J_1(\lambda r) d\lambda \\
    &-\frac{1}{4\pi} \frac{xy}{r^2} \int_0^\infty \Big ( e^{-\lambda |z+h|} - \frac{P_{21}}{P_{11}} e^{\lambda (z-h)} \Big ) \lambda^2 J_0(\lambda r) d\lambda \\
    H_z(x,y,z,\omega) =& \frac{1}{4\pi} \frac{x}{r} \int_0^\infty \Big ( e^{-\lambda |z+h|} + \frac{P_{21}}{P_{11}} e^{\lambda (z-h)} \Big ) \lambda^2 J_1(\lambda r) d\lambda
    \end{align}
    :name: Soln_xdip


The Hankel transforms in eqs. :eq:`Soln_zdip` and :eq:`Soln_xdip` are computed using the digital filtering routine of Anderson
(:cite:`Anderson1982`). The kernels of these equations are pre-computed at a certain number of logarithmically-spaced values of :math:`\lambda`.
Anderson’s routine then extracts the values of the kernels at the values of :math:`\lambda` it requires by cubic
spline interpolation. The number of values of :math:`\lambda` at which the kernels are pre-computed (50 minimum) can
be specified in the input file “em1dfm.in”; see “line 11” in the :ref:`input file<inputEM1DFM>` description.

There are three places where previously-computed components of eqs. :eq:`Soln_zdip` and :eq:`Soln_xdip` can be re-used. The
propagation of the matrices through the layers depends on frequency, and must be re-done for each different
value. However, the propagated matrix :math:`\mathbf{P}`, and hence the ratio :math:`P_{21}/P_{11}`, does not depend on the relative
location and orientation of the transmitter and receiver, and so can be re-used for all transmitters and
receivers for the same frequency. Furthermore, if there are multiple transmitter-receiver pairs with the same
height (and the same frequency), there is no difference in the kernels of their Hankel transforms, and so the
values of the kernels computed for one pair can be re-used for all the others. It is to ensure this grouping of
the survey parameters that the observations file is structured the way it is (see the :ref:`observation file<supportingFiles_obs>`).

The individual propagation matrices :math:`\mathbf{M_j}`, and each matrix computed in the construction of the propagation matrix :math:`\mathbf{P}`, are saved in the forward-modelling routine. These are then re-used in the computation of the sensitivities.

.. _theory_sensitivities:

Computing Sensitivities
-----------------------

The inverse problem of determining the conductivity and/or susceptibility of the Earth from electromagnetic
measurements is nonlinear. Program EM1DFM uses an iterative procedure to solve this problem. At each
iteration the linearized approximation of the full nonlinear problem is solved. This requires the Jacobian
matrix for the sensitivities, :math:`\mathbf{J} = (\mathbf{J^\sigma}, \mathbf{J^\kappa})` where:

.. math::
    \begin{align}
    J_{ij}^\sigma &= \frac{\partial d_i}{\partial log \, \sigma_j} \\
    J_{ij}^\kappa &= \frac{\partial d_i}{\partial k_j}
    \end{align}
    :name: Sensitivity

in which :math:`d_i` is the :math:`i^{th}` observation, and :math:`\sigma_j` and :math:`\kappa_j` are the conductivity and susceptibility of the :math:`j^{th}` layer.

The algorithm for computing the sensitivities is obtained by differentiating the expressions for the H-fields (see :eq:`Soln_zdip` and :eq:`Soln_xdip`)
with respect to the model parameters (:cite:`Farquharson2003`). For example, the
sensitivity with respect to :math:`m_j` (either the conductivity or susceptibility of the :math:`j^{th}` layer) of the
z-component of the H-field for a z-directed magnetic dipole source is given by differentiating the third expression in :eq:`Soln_zdip`:

.. math::
    \frac{\partial H_z}{\partial m_j} (x,y,z,\omega) = \frac{1}{4\pi} \int_0^\infty \Big ( e^{-\lambda |z+h|} + \frac{\partial}{\partial m_j} \Bigg [ \frac{P_{21}}{P_{11}} \Bigg ] e^{\lambda (z-h)} \Big ) \lambda^2 J_0(\lambda r) d\lambda
    :name: Sensitivity_z

The derivative of the coefficient is simply:

.. math::
    \frac{\partial}{\partial m_j} \Bigg [ \frac{P_{21}}{P_{11}} \Bigg ] = \frac{\partial P_{21}}{\partial m_j} \frac{1}{P_{11}} - \frac{\partial P_{11}}{\partial m_j} \frac{P{21}}{P_{11}^2}
    :name:

where :math:`P_{11}` and :math:`P_{21}` are elements of the propagation matrix :math:`\mathbf{P}` given by eq. :eq:`M_prod`. The derivative of :math:`\mathbf{P}` with respect to :math:`m_j` (for :math:`1 \leq j \leq M-1`) is

.. math::
    \frac{\partial \mathbf{P}}{\partial m_j} = \mathbf{M_1 M_2 ... M_{j-1}} \Bigg ( \frac{\partial \mathbf{M_j}}{\partial m_j} \mathbf{M_{j+1}} + \mathbf{M_j} \frac{\partial \mathbf{M_{j+1}}}{\partial m_j} \Bigg ) \mathbf{M_{j+2} ... M_M}
    :name:

The sensitivities with respect to the conductivity and susceptibility of the basement halfspace are given by

.. math::
    \frac{\partial \mathbf{P}}{\partial m_M} = \mathbf{M_1 M_2 ... M_{M-1}} \frac{\partial \mathbf{M_M}}{\partial m_M} 
    :name:

The derivatives of the individual layer matrices with respect to the conductivities and susceptibilities are
straightforward to derive, and are not given here.

Just as for the forward modelling, the Hankel transform in eq. :eq:`Sensitivity_z`, and those in the corresponding
expressions for the sensitivities of the other observations, are computed using the digital filtering routine of Anderson (:cite:`Anderson1982`).

The partial propagation matrices

.. math::
    \mathbf{P_k} = \mathbf{M_1} \prod_{j=2}^k \mathbf{M_j}, \;\;\; k=2,...,M
    :name:

are computed during the forward modelling, and saved for re-use during the sensitivity computations. This
sensitivity-equation approach therefore has the efficiency of an adjoint-equation approach.

.. _theory_inversion:

Inversion Methodologies
-----------------------

In program EM1DFM, there are four different inversion algorithms. They all have the same :ref:`general formulation <theory_inversion_gen>`, but differ in their treatment of the trade-off parameter (see :ref:`fixed trade-off <theory_inversion_fixed>`, :ref:`discrepency principle <theory_inversion_disc>`, :ref:`GCV <theory_inversion_gcv>` and :ref:`L-curve criterion <theory_inversion_lcurve>`).
In addition, there are four possibilities for the Earth model constructed by the inversion: 

    1) conductivity only
    2) susceptibility only (with positivity enforced)
    3) conductivity and susceptibility (with positivity of the susceptibilities enforced)
    4) conductivity and susceptibility (without the positivity constraint)

.. _theory_inversion_gen:

General formulation
^^^^^^^^^^^^^^^^^^^

The aim of each inversion algorithm is to construct the simplest model that adequately reproduces the
observations. This is achieved by posing the inverse problem as an optimization problem in which we recover the model that minimizes the objective function:

.. math::
    \Phi = \phi_d + \beta \phi_m - \gamma \phi_{LB}
    :name: ObjectiveFun

The three components of this objective function are as follows. :math:`\phi_d` is the data misfit:

.. math::
    \phi_d = \| \mathbf{W_d} (\mathbf{d - d^{obs}} ) \|^2
    :name:

where :math:`\| \, \cdot \, \|` represents the :math:`l_2`-norm, :math:`d^{obs}` is the vector containing the
:math:`N` observations, and :math:`d` is the forward-modelled data. It is assumed that the noise in the observations is Gaussian and uncorrelated, and that the
estimated standard deviation of the noise in the :math:`i^{th}` observation is of the form :math:`s_0 \hat{s}_i`, where :math:`\hat{s}_i` indicates the
amount of noise in the :math:`i^{th}` observation relative to that in the others, and is a scale factor that specifies
the total amount of noise in the set of observations. The matrix :math:`\mathbf{W_d}` is therefore given by:

.. math::
    \mathbf{W_d} = \textrm{diag} \big \{ 1/(s_0 \hat{s}_1), ..., 1/(s_0 \hat{s}_N) \}
    :name:


The model-structure component of the objective function is :math:`\phi_m`. In its most general form it contains four terms:

.. math::
    \begin{split}
    \phi_m =& \; \alpha_s^\sigma \big \| \mathbf{W_s^\sigma} \big ( \mathbf{m^\sigma - m_s^{\sigma , ref}} \big ) \big \|^2\\
    &+ \alpha_z^\sigma \big \| \mathbf{W_z^\sigma} \big ( \mathbf{m^\sigma - m_z^{\sigma , ref}} \big ) \big \|^2\\
    &+ \alpha_s^\kappa \big \| \mathbf{W_s^\kappa} \big ( \mathbf{m^\kappa - m_s^{\kappa , ref}} \big ) \big \|^2\\
    &+ \alpha_z^\kappa \big \| \mathbf{W_z^\kappa} \big ( \mathbf{m^\kappa - m_z^{\kappa , ref}} \big ) \big \|^2
    \end{split}
    :name: MOF

where :math:`\mathbf{m^\sigma}` is the vector containing the logarithms of the layer conductivities, and :math:`\mathbf{m^\kappa}` is the vector containing
the layer susceptibilities. The matrices :math:`\mathbf{W_s^\sigma}` and :math:`\mathbf{W_s^\kappa}` are:

.. math::
    \mathbf{W_s^\sigma} = \mathbf{W_s^\kappa} = \textrm{diag} \big \{ \sqrt{t_1}, ..., \sqrt{t_{m-1}}, \sqrt{t_{M-1}} \big \}
    :name:

where :math:`t_j` is the thickness of the :math:`j^{th}` layer. And the matricies :math:`\mathbf{W_z^\sigma}` and :math:`\mathbf{W_z^\kappa}` are:

.. math::
    \mathbf{W_z^\sigma} = \mathbf{W_z^\kappa} =
    \begin{bmatrix} -\sqrt{\frac{2}{t_1 + t_2}} & \sqrt{\frac{2}{t_1 + t_2}} & & & & \\
    & -\sqrt{\frac{2}{t_2 + t_3}} & \sqrt{\frac{2}{t_2 + t_3}} & & & \\
    & & \ddots & & & \\
    & & & -\sqrt{\frac{2}{t_{M-2} + t_{M-1}}} & \sqrt{\frac{2}{t_{M-2} + t_{M-1}}} & \\
    & & & & -\sqrt{\frac{2}{t_{M-1}}} & \sqrt{\frac{2}{t_{M-1}}} \\
    & & & & & 0 \end{bmatrix}
    :name:

The rows of any of these four weighting matrices can be scaled if desired (see file for :ref:`additional model-norm weights<supportingFiles_weight>`). The
vectors :math:`\mathbf{m_s^{\sigma , ref}}`, :math:`\mathbf{m_z^{\sigma , ref}}`, :math:`\mathbf{m_s^{\kappa , ref}}` and :math:`\mathbf{m_z^{\kappa , ref}}`
contain the layer conductivities/susceptibilities for the four possible reference models. The four terms in
:math:`\phi_m` therefore correspond to the “smallest” and “flattest” terms for the
conductivity and susceptibility parts of the model. The relative importance of the four terms is governed by
the coefficients :math:`\mathbf{\alpha_s^{\sigma}}`, :math:`\mathbf{\alpha_z^{\sigma}}`, :math:`\mathbf{\alpha_s^{\kappa}}` and :math:`\mathbf{\alpha_z^{\kappa}}`
, which are discussed in the :ref:`general formulation of the inversion problem<theory_inversion_gen>`. :math:`\beta` is the trade-off parameter that
balances the opposing effects of minimizing the misfit and minimizing the amount of structure in the model.
It is the different ways in which :math:`\beta` is determined that distinguish the four inversion algorithms in program
EM1DFM from one another. They are described in the next sections.

Finally, the third component of the objective function is a logarithmic barrier term:

.. math::
    \phi_{LB} = \sum_{j-1}^M \textrm{log} \, (c\kappa_j)
    :name: barrier_cond

where :math:`c` is a constant, usually equal to 1. This term is how the positivity constraint on the layer susceptibilities
is enforced. It, and its coefficient :math:`\gamma`, are described :ref:`here<theory_pos_sus>`.

As mentioned in the :ref:`computing sensitivities <theory_sensitivities>` section, the inverse problem considered here is nonlinear. It is solved using an
iterative procedure. At the :math:`n^{th}` iteration, the actual objective function being minimized is:

.. math::
    \Phi^n = \phi_d^n + \beta^n \phi_m^n - \gamma^n \phi^n_{LB}
    :name: Objective_Fcn

In the data misfit :math:`\phi_d^n`, the forward-modelled data :math:`d_n` are the data for the model that is sought at the current iteration. These data
are approximated by:

.. math::
    \mathbf{d^n} = \mathbf{d}^{n-1} + \mathbf{J}^{\sigma, n-1} \delta \mathbf{m}^\sigma + \mathbf{J}^{\kappa, n-1} \delta \mathbf{m}^\kappa
    :name: DataPerturb

where :math:`\delta \mathbf{m}^\sigma = \mathbf{m}^{\sigma , n} - \mathbf{m}^{\sigma , n-1}\;` \& :math:`\;\delta \mathbf{m}^\kappa = \mathbf{m}^{\kappa , n} - \mathbf{m}^{\kappa , n-1}`, and
:math:`\mathbf{J}^{\sigma , n-1}` \& :math:`\mathbf{J}^{\kappa , n-1}` are the two halves of the Jacobian matrix given by :eq:`Sensitivity` and evaluated for the model from the previous iteration. At
the :math:`n^{th}` iteration, the problem to be solved is that of finding the change, (:math:`\delta \mathbf{m}^\sigma , \delta \mathbf{m}^\kappa`) to the model which
minimizes the objective function :math:`\Phi^n`. Differentiating eq. :eq:`Objective_Fcn` with respect to the components of :math:`\delta \mathbf{m}^\sigma` \& :math:`\delta \mathbf{m}^\kappa`, and
equating the resulting expressions to zero, gives the system of equations to be solved. The derivatives of :math:`\phi^n_d` (incorporating the approximation of eq. :eq:`DataPerturb`) and
are straightforward to calculate. However, a further approximation must be made to linearize the derivatives of the logarithmic barrier term:

.. math::
    \begin{split}
    \frac{\partial \phi^n_{LB}}{\partial \delta m_k^\kappa} &= \frac{\partial}{\partial \delta \kappa_k} \sum_{j=1}^M \textrm{log} \big ( \kappa_j^{n-1} + \delta \kappa_j \big ) \\
    &= \frac{1}{\kappa_k^{n-1} + \delta \kappa_j} \\
    & \approx \frac{1}{\kappa_k^{n-1}} \Bigg ( 1 - \frac{\delta \kappa_k}{\kappa_k^{n-1}} \Bigg )
    \end{split}
    :name:

The linear system of equations to be solved for (:math:`\delta \mathbf{m}^\sigma , \delta \mathbf{m}^\kappa`) is therefore:

.. math::
    \begin{split}
    & \bigg [ \mathbf{J}^{n-1 \, T} \mathbf{W_d}^T \mathbf{W_d} \mathbf{J}^{n-1} + \beta^n \sum_{i=1}^2 \mathbf{W_i}^T \mathbf{W_i} + \frac{\gamma^n}{2} \mathbf{\hat{X}}^{n-1 \, T} \mathbf{\hat{X}}^{n-1} \bigg ] \delta \mathbf{m} = \\
    & \mathbf{J}^{n-1 \, T} \mathbf{W_d}^{n-1} \mathbf{W_d} \big ( \mathbf{d^{obs}} - \mathbf{d}^{n-1} \big )
    + \beta^n \sum_{i=1}^2 \mathbf{W_i}^T \mathbf{W_i} \big ( \mathbf{m_i^{ref} - \mathbf{m}^{n-1}} \big )
    + \frac{\gamma^n}{2} \mathbf{\hat{X}}^{n-1 \, T} \mathbf{\hat{X}}^{n-1} \mathbf{m}^{n-1}
    \end{split}
    :name: Systemdm

where :math:`T` denotes the transpose and:

.. math::
    \begin{split}
    \mathbf{J}^{n-1} &= \big ( \mathbf{J}^{\sigma , n-1} \mathbf{J}^{\kappa , n-1} \big ) \\
    \mathbf{W_1} &= \begin{bmatrix} \sqrt{\alpha_s^\sigma} \mathbf{W}_s^\sigma & 0 \\ 0 & \sqrt{\alpha_s^\kappa} \mathbf{W}_s^\kappa \end{bmatrix} \\ 
    \mathbf{W_2} &= \begin{bmatrix} \sqrt{\alpha_z^\sigma} \mathbf{W}_z^\sigma & 0 \\ 0 & \sqrt{\alpha_z^\kappa} \mathbf{W}_z^\kappa \end{bmatrix} \\
    \mathbf{m_1^{ref}} &= \big ( \mathbf{m}_s^{\sigma , ref \, T} \mathbf{m}_s^{\kappa , ref \, T} \big )^T \\
    \mathbf{m_2^{ref}} &= \big ( \mathbf{m}_z^{\sigma , ref \, T} \mathbf{m}_z^{\kappa , ref \, T} \big )^T \\
    \mathbf{\hat{X}}^{n-1} &= \big ( 0 \, (\mathbf{X}^{n-1})^{-1} \big )
    \end{split}
    :name:

where :math:`\mathbf{\hat{X}}^{n-1} = \textrm{diag} \{ m_1^{\kappa, n-1}, ... , m_M^{\kappa, n-1} \}`. The solution to eq. :eq:`Systemdm` is equivalent to the least-squares solution of:

.. math::
    \begin{bmatrix} \mathbf{W_d J}^{n-1} \\ \sqrt{\beta^n} \mathbf{W_1} \\ \sqrt{\beta^n} \mathbf{W_2} \\ \sqrt{\gamma^n/2} \, \mathbf{\hat{X}}^{n-1} \end{bmatrix} \delta \mathbf{m} =
    \begin{bmatrix} \mathbf{W_d } ( \mathbf{d^{obs} - d}^{n-1} ) \\ \sqrt{\beta^n} \mathbf{W_1} ( \mathbf{m_1^{ref} - m}^{n-1} ) \\ \sqrt{\beta^n} \mathbf{W_2}( \mathbf{m^{ref} - m}^{n-1} ) \\ \sqrt{\gamma^n/2} \, \mathbf{\hat{X}}^{n-1} \mathbf{m}^{n-1} \end{bmatrix}
    :name: SystemdmLSQ

Once the step :math:`\delta \mathbf{m}` has been determined by the solution of eq. :eq:`Systemdm` or eq. :eq:`SystemdmLSQ`, the new model is given by:

.. math::
    \mathbf{m}^n = \mathbf{m}^{n-1} + \nu \delta \mathbf{m}
    :name: mod_update

There are two conditions on the step length :math:`\nu`. First, if positivity of the layer susceptibilities is being enforced:

.. math::
    \nu \delta \kappa_j > -\kappa_j^{n-1}
    :name: cond1

must hold for all :math:`j=1,...,M`. Secondly, the objective function must be decreased by the addition of the
step to the model:

.. math::
    \phi_d^n + \beta^n \phi_m^n - \gamma^n \phi_{LB}^n < \phi_d^{n-1} + \beta^n \phi_m^{n-1} - \gamma^n \phi_{LB}^{n-1}
    :name: cond2

where :math:`\phi_d^n` is now the misfit computed using the full forward modelling for the new model :math:`\mathbf{m}^n`. To determine
:math:`\mathbf{m}^n`, a step length (:math:`\nu`) of either 1 or the maximum value for which eq. :eq:`cond1` is true (whichever is greater) is
tried. If eq. :eq:`cond2` is true for the step length, it is accepted. If eq. :eq:`cond2` is not true, :math:`\nu` is decreased by factors of 2 until it is true.

.. _theory_inversion_fixed:

Algorithm 1: fixed trade-off parameter
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The trade-off parameter, :math:`\beta`, remains fixed at its user-supplied value throughout the inversion. The least-
squares solution of eq. :eq:`SystemdmLSQ` is used. This is computed using the subroutine “LSQR” of Paige & Saunders
(:cite:`Paige1982`). If the desired value of :math:`\beta` is known, this is the fastest of the four inversion algorithms as it does not
involve a line search over trial values of :math:`\beta` at each iteration. If the appropriate value of :math:`\beta` is not known, it
can be found using this algorithm by trail-and-error. This may or may not be time-consuming.

.. _theory_inversion_disc:

Algorithm 2: discrepancy principle
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If a complete description of the noise in a set of observations is available - that is, both :math:`s_0` and :math:`\hat{s}_i \: (i=1,...,N)` are known - the expectation of the misfit,
:math:`E (\phi_d)`, is equal to the number of observations :math:`N`. Algorithm 2 therefore attempts to choose the trade-off parameter so that the misfit for the final model is equal to a target
value of :math:`chifac \times N`. If the noise in the observations is well known, :math:`chifac` should equal 1. However, :math:`chifac` can be adjusted by the user to give a target misfit appropriate for a particular data-set. If a misfit as small as the target value cannot be achieved, the algorithm searches for the smallest possible misfit.

Experience has shown that choosing the trade-off parameter at early iterations in this way can lead to
excessive structure in the model, and that removing this structure once the target (or minimum) misfit has
been attained can require a significant number of additional iterations. A restriction is therefore placed on
the greatest-allowed decrease in the misfit at any iteration, thus allowing structure to be slowly but steadily
introduced into the model. In program EM1DFM, the target misfit at the :math:`n^{th}` iteration is given by:

.. math::
    \phi_d^{n, tar} = \textrm{max} \big ( mfac \times \phi_d^{n-1}, chifac \times N \big )
    :name: mfac

where the user-supplied factor :math:`mfac` is such that :math:`0.1 \leq mfac \leq 0.5`.

The step :math:`\delta \mathbf{m}` is found from the solution of eq. :eq:`SystemdmLSQ` using subroutine
LSQR of Paige & Saunders (:cite:`Paige1982`). The line search at each iteration moves along the :math:`\phi_d` versus log :math:`\! \beta` curve until either the target misfit, :math:`\phi_d^{n, tar}`,
is bracketed, in which case a bisection search is used to converge to the target, or the minimum misfit
(:math:`> \phi_d^{n-1}`) is bracketed, in which case a golden section search (for example, Press et al., 1986) is used to
converge to the minimum. The starting value of :math:`\beta` for each line search is :math:`\beta^{n-1}`. For the first iteration, the :math:`\beta \, (=\beta_0)` for the line search is given by
:math:`N/\phi_m (\mathbf{m}^\dagger)`, where :math:`\mathbf{m}^\dagger` contains typical values of conductivity and/or susceptibility. Specifically, :math:`\mathbf{m}^\dagger` is a model whose top
:math:`M/5` layers have a conductivity of 0.02 S/m and susceptibility of 0.02 SI units, and whose remaining layers have a conductivity of 0.01 S/m and
susceptibility of 0 SI units. Also, the reference models used in the computation of :math:`\phi_m (\mathbf{m}^\dagger )` are homogeneous
halfspaces of 0.01 S/m and 0 SI units. The line search is efficient, but does involve the full forward modelling to compute the misfit for each trial value of :math:`\beta`.

.. _theory_inversion_gcv:

Algorithm 3: GCV criterion
^^^^^^^^^^^^^^^^^^^^^^^^^^

If only the relative amount of noise in the observations is known - that is, :math:`\hat{s}_i (i=1,...,N)` is known but not :math:`s_0` -
the appropriate target value for the misfit cannot be determined, and hence Algorithm 2 is not the most
suitable. The generalized cross-validation (GCV) method provides a means of estimating, during the course
of an inversion, a value of the trade-off parameter that results in an appropriate fit to the observations, and
in so doing, effectively estimating the level of noise, :math:`s_0`, in the observations (see, for example, :cite:`Wahba1990`;
:cite:`Hansen1998`).

The GCV method is based on the following argument (:cite:`Wahba1990`; :cite:`Haber1997`; :cite:`Haber2000`). Consider inverting all but the first observation using a trial value of :math:`\beta`, and then computing the
individual misfit between the first observation and the first forward-modelled datum for the model produced
by the inversion. This can be repeated leaving out all the other observations in turn, inverting the retained
observations using the same value of :math:`\beta`, and computing the misfit between the observation left out and the
corresponding forward-modelled datum. The best value of :math:`\beta` can then be defined as the one which gives the
smallest sum of all the individual misfits. For a linear problem, this corresponds to minimizing the GCV
function. For a nonlinear problem, the GCV method can be applied to the linearized problem being solved
at each iteration (:cite:`Haber1997`; :cite:`Haber2000`; :cite:`Li2003`; :cite:`Farquharson2000`). From eq. :eq:`Systemdm`, the GCV function for the :math:`n^{th}` iteration is given by:

.. math::
    GCV (\beta ) = \dfrac{\big \| \mathbf{W_d \hat{d} - W_d J}^{n-1} \mathbf{M}^{-1} \big ( \mathbf{J}^{n-1 \, T} \mathbf{W_d}T \mathbf{W_d \hat{d} + r} \big ) \big \|^2 }{\big [ \textrm{trace} \big ( \mathbf{I - W_d J}^{n-1} \mathbf{M}^{-1} \mathbf{J}^{n-1 \, T} \mathbf{W_d}^T \big )  \big ]^2}
    :name: GCV

where

.. math::
    \begin{split}
    \mathbf{M} (\beta) &= \bigg [ \mathbf{J}^{n-1 \, T} \mathbf{W_d}^T \mathbf{W_d} \mathbf{J}^{n-1} + \beta^n \sum_{i=1}^2 \mathbf{W_i}^T \mathbf{W_i} + \frac{\gamma^n}{2} \mathbf{\hat{X}}^{n-1 \, T} \mathbf{\hat{X}}^{n-1} \bigg ] \\
    \mathbf{r} &= \beta^n \sum_{i=1}^2 \mathbf{W_i}^T \mathbf{W_i} \big ( \mathbf{m_i^{ref} - \mathbf{m}^{n-1}} \big ) + \frac{\gamma^n}{2} \mathbf{\hat{X}}^{n-1 \, T} \mathbf{\hat{X}}^{n-1} \mathbf{m}^{n-1}
    \end{split}
    :name:

and :math:`\mathbf{\hat{d} - d^{obs} - d}^{n-1}`. If :math:`\beta^*` is the value of the trade-off parameter that minimizes eq. :eq:`GCV` at the :math:`n^{th}` iteration, the actual value of
:math:`\beta` used to compute the new model is given by:

.. math::
    \beta_n = \textrm{max} (\beta^*, bfac \times \beta^{n-1} )
    :name: betachoice

where the user-supplied factor :math:`bfac` is such that :math:`0.01<bfac<0.5`. As for Algorithm 2, this limit on the
allowed decrease in the trade-off parameter prevents unnecessary structure being introduced into the model
at early iterations. The inverse of the matrix :math:`\mathbf{M}` required in eq. :eq:`GCV`, and the solution to eq. :eq:`Systemdm` given this inverse, is
computed using the Cholesky factorization routines from LAPACK (:cite:`Anderson1999`). The line search at each iteration moves along the curve of the GCV function versus the logarithm of the trade-off parameter
until the minimum is bracketed (or :math:`bfac \times \beta^{n-1}` reached), and then a golden section search (e.g., Press et al.,
1986) is used to converge to the minimum. The starting value of :math:`\beta` in the line search is :math:`\beta^{n-1}` ( :math:`\beta^0` is estimated
in the same way as for Algorithm 2). This is an efficient search, even with the inversion of the matrix :math:`\mathbf{M}`.

.. _theory_inversion_lcurve:

Algorithm 4: L-curve criterion
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As for the :ref:`GCV-based method <theory_inversion_gcv>`, the L-curve method provides a means of estimating
an appropriate value of the trade-off parameter if only :math:`\hat{s}_i, \, i=1,...,N` are known and not :math:`s_0`. For a linear
inverse problem, if the data misfit :math:`\phi_d` is plotted against the model norm :math:`\phi_m` for all reasonable values of the
trade-off parameter :math:`\beta`, the resulting curve tends to have a characteristic "L"-shape, especially when plotted
on logarithmic axes (see, for example, :cite:`Hansen1998`). The corner of this L-curve corresponds to roughly
equal emphasis on the misfit and model norm during the inversion. Moving along the L-curve away from the
corner is associated with a progressively smaller decrease in the misfit for large increases in the model norm,
or a progressively smaller decrease in the model norm for large increases in the misfit. The value of :math:`\beta` at the
point of maximum curvature on the L-curve is therefore the most appropriate, according to this criterion.

For a nonlinear problem, the L-curve criterion can be applied to the linearized inverse problem at each
iteration (Li & Oldenburg, 1999; Farquharson & Oldenburg, 2000). In this situation, the L-curve is defined
using the linearized misfit, which uses the approximation given in eq. :eq:`DataPerturb` for the forward-modelled data.
The curvature of the L-curve is computed using the formula (Hansen, 1998):

.. math::
    C(\beta) = \frac{\zeta^\prime \eta^{\prime \prime } - \zeta^{\prime\prime} \eta^\prime}{\big ( (\zeta^\prime)^2 + (\eta^\prime)^2 \big )^{3/2}}
    :name: zetaeq

where :math:`\zeta = \textrm{log} \, \phi_d^{lin}` and :math:`\eta = \textrm{log}\, \phi_m`. The prime denotes differentiation with respect to log :math:`\beta`. As for both
Algorithms :ref:`2 <theory_inversion_disc>` & :ref:`3 <theory_inversion_gcv>`, a restriction is imposed on how quickly the trade-off parameter can be decreased from one iteration to the next. The actual value of :math:`\beta` chosen for use at the
:math:`n^{th}` th iteration is given by eq. :eq:`betachoice`, where :math:`\beta^*` now corresponds to the value of :math:`\beta` at the point of maximum curvature on the L-curve.

Experience has shown that the L-curve for the inverse problem considered here does not always have
a sharp, distinct corner. The associated slow variation of the curvature with :math:`\beta` can make the numerical
differentiation required to evaluate eq. :eq:`zetaeq` prone to numerical noise. The line search along the L-curve used
in program EM1DFM to find the point of maximum curvature is therefore designed to be robust (rather
than efficient). The L-curve is sampled at equally-spaced values of :math:`\textrm{log} \, \beta`, and long differences are used in the
evaluation of eq. :eq:`zetaeq` to introduce some smoothing. A parabola is fit through the point from the equally-spaced sampling with the maximum value of curvature and its two nearest neighbours. The value of :math:`\beta` at the
maximum of this parabola is taken as :math:`\beta^*`. In addition, it is sometimes found that, for the range of values of
:math:`\beta` that are tried, the maximum value of the curvature of the L-curve on logarithmic axes is negative. In this
case, the curvature of the L-curve on linear axes is investigated to find a maximum. As for Algorithms 1 &
2, the least-squares solution to eq. :eq:`SystemdmLSQ` is used, and is computed using subroutine LSQR of Paige & Saunders (:cite:`Paige1982`).

.. _theory_alphas:

Relative weighting within the model norm
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The four coefficients in the model norm (see eq. :eq:`MOF`) are ultimately the responsibility of the user. Larger
values of :math:`\alpha_s^\sigma` relative to :math:`\alpha_z^\sigma` result in constructed conductivity models that are closer to the supplied reference
model. Smaller values of :math:`\alpha_s^\sigma` and :math:`\alpha_z^\sigma` result in flatter conductivity models. Likewise for the coefficients
related to susceptibilities.

If both conductivity and susceptibility are active in the inversion, the relative size of
:math:`\alpha_s^\sigma` & :math:`\alpha_z^\sigma` to :math:`\alpha_s^\kappa` & :math:`\alpha_z^\kappa`
is also required. Program EM1DFM includes a simple means of calculating a default value for this
relative balance. Using the layer thicknesses, weighting matrices :math:`\mathbf{W_s^\sigma}`, :math:`\mathbf{W_z^\sigma}`, :math:`\mathbf{W_s^\kappa}` & :math:`\mathbf{W_z^\kappa}`, and user-supplied
weighting of the smallest and flattest parts of the conductivity and susceptibility components of the model norm (see acs, acz, ass & asz in the :ref:`input file description<inputEM1DFM>`, line 5), the following two quantities
are computed for a test model :math:`\mathbf{m}^*`:

.. math::
    \begin{split}
    \phi_m^\sigma &= acs \big \| \mathbf{W_s^\sigma} \big ( \mathbf{m}^* - \mathbf{m}_s^{\sigma, ref} \big ) \big \|^2 + acz \big \| \mathbf{W_z^\sigma} \big ( \mathbf{m}^* - \mathbf{m}_z^{\sigma, ref} \big ) \big \|^2 \\
    \phi_m^\kappa &= ass \big \| \mathbf{W_s^\kappa} \big ( \mathbf{m}^* - \mathbf{m}_s^{\kappa, ref} \big ) \big \|^2 + asz \big \| \mathbf{W_z^\kappa} \big ( \mathbf{m}^* - \mathbf{m}_z^{\kappa, ref} \big ) \big \|^2
    \end{split}
    :name:

The conductivity and susceptibility of the top :math:`N/5` layers in the test model are 0.02 S/m and 0.02 SI units
respectively, and the conductivity and susceptibility of the remaining layers are 0.01 S/m and 0 SI units.
The coefficients of the model norm used in the inversion are then :math:`\alpha_s^\sigma = acs`, :math:`\alpha_z^\sigma = acz`, :math:`\alpha_s^\kappa = A^s \times ass` & :math:`\alpha_z^\kappa = A^d \times asz` where
:math:`A^s \phi_m^\sigma / \phi_m^\kappa`. It has been found that a balance between the conductivity and
susceptibility portions of the model norm computed in this way is adequate as an initial guess. However, the
balance usually requires modification by the user to obtain the best susceptibility model. (The conductivity
model tends to be insensitive to this balance.) If anything, the default balance will suppress the constructed
susceptibility model.

.. _theory_pos_sus:

Positive susceptibility
^^^^^^^^^^^^^^^^^^^^^^^

ProgramEM1DFM can perform an unconstrained inversion for susceptibilities (along with the conductivities)
as well as invert for values of susceptibility that are constrained to be positive. Following Li & Oldenburg
(:cite:`Li2003`), the positivity constraint is implemented by incorporating a logarithmic barrier term in the objective
function (see eqs. :eq:`ObjectiveFun` & :eq:`barrier_cond`). For the initial iteration, the coefficient of the logarithmic barrier term is chosen
so that this term is of equal important to the rest of the objective function:

.. math::
    \gamma^0 = \frac{\phi_d^0 + \beta^0 \phi_m^0}{- \phi^0_{LB}}
    :name:

At subsequent iterations, the coefficient is reduced according to the formula:

.. math::
    \gamma^n = \big ( 1 - \textrm{min}(\nu^{n-1}, 0.925) \big ) \gamma^{n-1}

where :math:`\nu^{n-1}` is the step length used at the previous iteration. As mentioned at the end of the :ref:`general formulation <theory_inversion_gen>`, when
positivity is being enforced, the step length at any particular iteration must satisfy eq. :eq:`cond1`.


Convergence criteria
^^^^^^^^^^^^^^^^^^^^

To determine when an inversion algorithm has converged, the following criteria are used (:cite:`Gill1981`):

.. math::
    \begin{split}
    \Phi^{n-1} - \Phi^n &< \tau (1 + \Phi^n )\\
    \| \mathbf{m}^{n-1} - \mathbf{m} \| &< \sqrt{\tau} (1 + \| \mathbf{m}^n \| )
    \end{split}
    :name: conv_criteria

where :math:`\tau` is a user-specified parameter. The algorithm is considered to have converged when both of the above
equations are satisfied. The default value of :math:`\tau` is 0.01.

In case the algorithm happens directly upon the minimum, an additional condition is tested:

.. math::
    \| \mathbf{g}^n \| \leq \epsilon
    :name: conv_criteria2

where :math:`\epsilon` is a small number close to zero, and where the gradient, :math:`\mathbf{g}^n`, at the :math:`n^{th}` iteration is given by:

.. math::
    \mathbf{g}^n = -2 \mathbf{J}^{n \, T} \mathbf{W_d}^T \mathbf{W_d} ( \mathbf{d^{obs}} - \mathbf{d}^n ) 
    - 2 \beta^n \sum_{i=1}^2 \mathbf{W_i}^T \mathbf{W_i} \big ( \mathbf{m_i^{ref} - \mathbf{m}^{n-1}} \big )
    - \gamma^n \mathbf{\hat{X}}^{n2T} \mathbf{m}^{n}
    :name:


